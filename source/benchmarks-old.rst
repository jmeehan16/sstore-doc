.. _benchmarks-old:

******************************************
Writing Applications/Benchmarks in S-Store
******************************************

The most common use of S-Store is the creation of applications, or benchmarks.  S-Store supports benchmarks with both streaming workloads and/or OLTP workloads.  

The benchmark creation process is very similar to that of H-Store.  To begin creating a benchmark, please follow the instructions in the H-Store documentation, linked here.  While the core benchmark pieces, such as the schema, stored procedures, project builder, and client are fundamentally the same, there are some important differences between an S-Store workload that features dataflow graphs and an H-Store OLTP workload.  These differences are listed below.

Creating Batches and a Client
-----------------------------

In S-Store (and most streaming system), order of execution is extremely important.  Data must be processed in the order in which it arrives.  S-Store executes and orders its dataflow processing in terms of batches.  A batch can be considered a group of one or more tuples that should be executed as a single, atomic unit.  Each batch includes a batch_id, which is associated with the time at which the batch arrived.  These batch_ids are what ensure that incoming tuples are processed in order.  Batches are currently created on the client side, but the batch_ids are automatically assigned by the engine.

Programming a client is very similar to the process described in the H-Store documentation, but with some key differences.  There are two primary methods of ingesting data from the client to the engine.  The first method, similar to H-Store, is to generate new tuples directly within the client.  This method is best when data is fabricated, as the input rate can easily be controlled at runtime.  The second method is to use the StreamGenerator tool (documented here) to simulate an incoming stream.

The client consists of two major methods for repeatedly inserting new tuples into : the runLoop() method and the runOnce() method.   Within the runOnce() method, the user can define a segment of code that runs x times per second, where x is defined by the -Dclient.txnrate parameter at runtime for each client.  An example can be found below:

.. code-block:: java

	protected boolean runOnce() throws IOException {

		Client client = this.getClientHandle();
    	
		String tuple = tuple_id + "," + tuple_val; //create tuple, DO NOT include a batch_id
		String[] curTuples = new String[1];
		curTuples[0] = tuple;
		boolean response = client.initStream(callback, "SP1", (Object[]) curTuples);
		
		return response;
	}

The runLoop() method runs as one would expect a loop to: as many times as possible, with no hesitation.  runLoop() is best used with the streamgenerator, as it automatically ingests tuples at whatever rate the streamgenerator is producing them.  The easiest way to code in such a way that both the runOnce() and runLoop() method can be used is to place all of the inner loop code within runOnce(), and then call runOnce() repeatedly from within runLoop(), like so:

.. code-block:: java

	//to use "runLoop" instead of "runOnce," set the client.txnrate param to -1 at runtime
	public void runLoop() {
		try {
			while (true) {
				try {
				runOnce();
				} catch (Exception e) {
					failedTuples.incrementAndGet();
				}
			} // WHILE
		} catch (Exception e) { // Client has no clean mechanism for terminating with the DB.
			e.printStackTrace();
		}
	}

As new tuples arrive, it is up to the client to group them into batches.  This is done by creating a String array with a size equal to the maximum number of tuples that you intend to send per batch.  In each iteration of the loop, the runOnce method takes in a new tuple and adds it to a batch.  When an entire batch is ready, that batch is submitted to the system by calling the client.initStream(Callback, ProcName, Tuples) method.  An example of this can be found below.

.. code-block:: java

	protected boolean runOnce() throws IOException {
		String tuple = tuple_id + "," + tuple_value; //create tuple, DO NOT include a batch_id
		curTuples[i++] = tuple;
		if (BenchmarkConstants.NUM_PER_BATCH == i) { // We have a complete batch now.
			Client client = this.getClientHandle();
			boolean response = client.initStream(callback, "SP1", (Object[]) curTuples);
			i = 0;
			curTuples = new String[BenchmarkConstants.NUM_PER_BATCH];
		}
	}

runOnce()/runLoop() can easily be connected to the StreamGenerator using a clientSocket and BufferedInputStream, as shown below:

.. code-block:: java

	public void runLoop() {
		Socket clientSocket = null;

		try {

			clientSocket = new Socket(BenchmarkConstants.STREAMINGESTOR_HOST, BenchmarkConstants.STREAMINGESTOR_PORT);
			clientSocket.setSoTimeout(5000);

			BufferedInputStream in = new BufferedInputStream(clientSocket.getInputStream());

			int i = 0;
			while (true) {
				int length = in.read();
				if (length == -1 || length == 0) {
					if (i > 0) {
						Client client = this.getClientHandle();
						boolean response = client.initStream(callback, "SP1", (Object[]) curTuples);
						i = 0;
					}
					break;
				}
				byte[] messageByte = new byte[length];
				in.read(messageByte);
				String tuple = new String(messageByte);
				curTuples[i++] = tuple;
				if (BenchmarkConstants.NUM_PER_BATCH == i) {
					// We have a complete batch now.
					Client client = this.getClientHandle();
					boolean response = client.initStream(callback, "SP1", (Object[]) curTuples);
					i = 0;
					curTuples = new String[BenchmarkConstants.NUM_PER_BATCH];
				}
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}


Creating Tables, Windows, and Streams
-------------------------------------

As is the case in H-Store, application schemas are defined in a DDL file.  The DDL file must be named the same as your benchmark, followed by "-ddl.sql".

There are three primary types of state in S-Store applications: Tables, Streams, and Windows.  All three types of state are defined as tables, and all three are fully recoverable.

**Tables** constitute the primary "shared mutable state" of S-Store.  Any publicly writeable data (accessible to all OLTP or ad-hoc queries) should be defined in a table.  Creating tables is identical to both VoltDB and H-Store.  The table schema and any indexes are defined as in the example below:

.. code-block:: sql
	
	CREATE TABLE T1
	(
    	tuple_id	bigint    NOT NULL,
    	tuple_val	integer   NOT NULL,
    	CONSTRAINT PK_T1 PRIMARY KEY (tuple_id)
	);

.. Note:: Partition keys for tables are defined in the ProjectBuilder class.

**Streams** are the primary method of moving information from one stored procedure to another within a dataflow graph.  While the data is primarily passed through stored procedure arguments, it is important to also store the data in persistent streams as well for recovery purposes.  Streams are logically append and remove only.  For now, it is left to the application developer to prevent any updates to data items in a stream.  Stream creation is very similar to table creation. An example of a stream is shown below.  

.. code-block:: sql

	CREATE STREAM S1
	(
    	tuple_id	bigint    	NOT NULL,
    	tuple_val	integer   	NOT NULL,
    	batch_id 	bigint		NOT NULL
	);

.. Note:: Automatic garbage collection on Streams is left to future functionality.  The application developer should ensure that expired data items within Streams are garbage collected once the tuples are no longer needed (i.e. once the downstream SP has committed).

**Windows** hold a fixed quantity of data that updates as new data arrives.  Windows can be either **tuple-based**, meaning that they always hold a fixed number of tuples, or **batch-based**, meaning that they hold a fixed number of batches at any given time.  Windows update periodically as a specific quantity of tuples or batches arrive.  This is known as the window's **slide** value.

In order to create a window, the user must first create a stream that features the same schema as the window.  This stream must feature two columns to be used by the system, but not by the user: *WSTART* and *WEND*.  Both columns are to be left nullable, and should be of the INTEGER data type.  Aside from defining these columns, the user does not need to be concerned with them.  In the case of batch-based windows, the user must define a third column, *ts*, of the bigint data type.  This column corresponds with the batch-id, and determines when the window slides.  Unlike *WSTART* and *WEND*, the *ts* column must be managed by the user. An example of this base stream is defined below:

.. code-block:: sql

	CREATE STREAM stream_for_win
	(
    	tuple_id 	bigint    	NOT NULL,
    	tuple_val 	integer    	NOT NULL,
    	ts 			bigint		NOT NULL,
    	WSTART		integer,
    	WEND		integer
	);

Once the template stream has been defined, the window can be defined based on that.  An example of a tuple-based window is below:

.. code-block:: sql

	CREATE WINDOW tuple_win ON stream_for_win ROWS [*number of rows*] SLIDE [*slide size*];

An example of a batch-based window is below:

.. code-block:: sql

	CREATE WINDOW batch_win ON stream_for_win RANGE [*number of batches*] SLIDE [*slide size*];


It is important to keep in mind that the window is its own separate data structure.  When inserting tuples into a window, they should be directly inserted into the window rather than the base stream.  Additionally, both the *WSTART* and *WEND* columns should be ignored during insert.  An example insert statement is shown below:

.. code-block:: java

	//insert into window
	public final SQLStmt insertProcTwoWinStmt = new SQLStmt(
		"INSERT INTO tuple_win (tuple_id, tuple_val, ts) VALUES (?,?,?);"
	);

Windows slides are handled automatically by the system, as the user would expect.  As new tuples/batches arrive, they are staged behind the scenes until enough tuples/batches arrive to slide the window by the appropriate amount.  Garbage collection is handled automatically, meaning that the user does ever need to manually delete tuples from a window.

.. Note:: In tuple-based window, no ordering is maintained within tuples in a batch.  This means that if a stored procedure is replayed upon recovery, the result may differ from the original value.  The results will remain consistent with our guarantees, however.

It is possible to attach an Execution Engine trigger to a window, as described below.  EE triggers execute on each window slide, not necessarily on each tuple insertion.

Creating OLTP Stored Procedures
-------------------------------

The primary unit of execution in S-Store are **stored procedures**.  Each execution of an S-Store stored procedure on an input batch results in a **transaction** with full ACID properties.  The definition of a stored procedure is very similar to that of H-Store Procedures_.  Constant SQL statements are defined and then submitted to the engine with parameters to be executed in batches.  An example of an OLTP stored procedure can be seen below.

.. _Procedures: http://hstore.cs.brown.edu/documentation/development/new-benchmark/#storedprocedures

.. code-block:: java

	@ProcInfo(
		partitionNum = 0; //states which partition this SP runs on
		singlePartition = true;
	)
	public class SP2 extends VoltProcedure {
		protected void toSetDataflowGraph() { //where dataflow graph details are set
			addPrevProc("SP1");
			addNextProc("SP3");
			setDataflowGraphName("D1");
		}

		public final SQLStmt temp = "SELECT * FROM S1;" //define SQL statements here

		public long run(int part_id, VoltStream sp1Data, long[] extraArgs) {
			//procedure work happens here

			return BenchmarkConstants.SUCCESS;
		}
	}


Creating Dataflow Graph Stored Procedures
------------------------------------------

Like most streaming systems, the main method of programming a workload in S-Store is via **dataflow graphs**.  A dataflow graph in S-Store is a series of stored procedures which are connected via streams in a directed acyclic graph.  

[image of dataflow graph]
[dataflow graph caption]

By default, each stored procedure in a dataflow graph executes on each batch that arrives from the input.  When a stored procedure commits on an input batch, the S-Store scheduler automatically triggers a transaction execution of the downstream stored procedure.  For each stored procedure, batch *b* is guaranteed to commit before batch *b+1*, and for each batch, stored procedure *t* is guaranteed to commit before transaction *t+1*.  Each transaction *t* is guaranteed to execute once and only once.  See the scheduler_ section for more details on how this occurs and in what order the transactions will execute.

.. Note:: Downstream stored procedures are triggered for each batch, even if no batch is passed downstream.  In this case, it is important that stored procedures be able to handle these empty, or **NULL** tuples, in order to avoid unexpected results.

Dataflow graphs are defined within the **dataflow stored procedure** definitions.  At the beginning of each dataflow SP, the user should define several traits within the *toSetDataflowGraph()* function.  The user must define 1) the name of the dataflow graph, 2) the SP immediately preceding this one in the dataflow graph (if any), and 3) the SP immediately following this one (if any).  An example of this for *SP2* as listed below:

.. code-block:: java

	protected void toSetDataflowGraph() {
		setDataflowGraphName("D1"); //defines which dataflow graph this proc is a part of
		addPrevProc("SP1"); //defines the previous SP in the dataflow graph
		addNextProc("SP3"); //defines the next SP in the dataflow graph
	}

.. Note: At the moment, S-Store only supports linear dataflow graphs.  Functionality to fork a dataflow graph will be provided in the near future, and support for merging branches of a dataflow graph will also be included in a later release.

Dataflow stored procedures are required to take in three primary parameters:  

1. *int* part_id - This parameter will automatically be filled in with the partitionNum ProcInfo parameter set at the beginning of the SP.  It is irrelevant for single-partition S-Store, but will be used in the distributed version.
2. *VoltStream* spData - This parameter is how stream data is passed from procedure to procedure.
3. *long[]* extraArgs - Provides a method of adding additional information into the stored procedure.

Passing Data Along Streams using VoltStreams
--------------------------------------------

Stream data is passed from procedure to procedure using VoltStreams as arguments.  VoltStreams are attached to Stream tables that are defined in the DDL.  The stream tables used must include a *batch_id* column of long data type.

As mentioned in the previous section, downstream stored procedures are activated with every transaction invocation.  This ensures that every SP executes for every batch_id, regardless of whether that batch contains new data that must be processed.

When data is being passed downstream, it must be inserted into a stream database object.  The stream is primarily there for recovery purposes, to ensure that transactions that have not been queued are able to be recovered in the case of failure.  In addition to being stored in the database object, the data must also be explicitly put into a VoltStream using the voltQueueSQLDownStream(SQLStmt, Object...) and voltExecuteSQLDownStream(String) commands.  These operate similarly to the voltQueueSQL() and voltExecuteSQL() commands, but with some important additions.  Below is an example:

.. code-block:: java

	voltQueueSQLDownStream(insertStmt, batch_id, tuple_value);//inserts the current tuple into the output stream
	voltExecuteSQLDownStream("out_stream");//selects the appropriate output stream for the current procedure

.. Note:: Garbage collection is not currently implemented for stream tables.  Tuples will need to be manually deleted from these tables once the downstream stored procedure has executed on the corresponding batch.

In some cases, downstream stored procedures will need to be executed on a batch_id even when there is no new data to be processed.  In this case, the developer will need to declare a type of "NULL" tuple, and manage a way to recognize these within the system.  One common way of doing this is to define some value that would otherwise never appear, and declare that value to represent a NULL tuple.  It is important that even in the NULL tuple case, the *batch_id* remain the same as the batch that is executing.  An example is shown below:

.. code-block:: java

	voltQueueSQLDownStream(insertStmt, batch_id, null_tuple_value);//inserts a recognized NULL tuple to the output stream, but with the same batch_id
	voltExecuteSQLDownStream("out_stream");//selects the appropriate output stream for the current procedure

.. Note:: Currently, NULL tuples do require downstream stored procedures to execute, even if no real work is being accomplished.  Future releases of S-Store will include the option to circumvent these additional SP executions as a way of optimizing transaction processing.

Execution Engine Triggers
-------------------------

**Execution Engine triggers** (also known as **EE triggers** or **backend triggers**) are SQL statements that are attached to tables, windows, or streams. These triggers execute the attached SQL code immediately upon the insertion of a tuple. Note that if a batch of many tuples is inserted with one command, the trigger will fire once for each insertion.

EE triggers are defined in a way that is similar to stored procedures. They are placed in the "procedures" package of the benchmark, and similarly declared within the ProjectBuilder class. Any EE trigger object extends the VoltTrigger class. The stream/window/table to which the trigger is attached must be defined by overriding the "toSetStreamName()" method, which will return the target object name.

.. code-block:: java

	protected String toSetStreamName() {
		return "s1";
	}

Each SQL statement that should be run upon tuple insert is then defined. These statements will run sequentially. Usually an "INSERT INTO...SELECT" statement will be used in order to somehow manipulate the data and push it downstream. Here is an example:

.. code-block:: java

	public final SQLStmt thisStmtName = new SQLStmt(
		"INSERT INTO sometable SELECT * FROM thisstream;"
	);

EE triggers have different semantics depending on what type of object they are attached to. For streams and tables, the triggers execute the attached SQL code immediately upon the insertion of a tuple. Note that if a batch of many tuples is inserted with one command, the trigger will fire once for each insertion. Tuples are automatically garbage collected once the attached SQL has finished running.

EE triggers attached to windows, however, operate differently. Rather than firing on the insertion of new tuples, the triggers instead fire on the sliding of the window. This is particularly useful for aggregating the contents of a window upon slide and pushing it into a downstream table or stream.

There are some limitations. EE triggers are unable to accept parameterized SQL statements, but both joins and aggregates can be used. Additionally, EE triggers are unable to activate a PE trigger. This means that if a tuple is inserted into a PE trigger stream directly from an EE trigger, the downstream stored procedure will not be activated.


