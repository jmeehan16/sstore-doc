.. _benchmarks:

****************************
Writing Applications/Benchmarks in S-Store
****************************

The most common use of S-Store is the creation of applications, or benchmarks.  S-Store supports benchmarks with both streaming workloads and/or OLTP workloads.  

The benchmark creation process is very similar to that of H-Store.  To begin creating a benchmark, please follow the instructions in the H-Store documentation, linked here.  While the core benchmark pieces, such as the schema, stored procedures, project builder, and client are fundamentally the same, there are some important differences between an S-Store workload that features dataflow graphs and an H-Store OLTP workload.  These differences are listed below.

Creating Batches and a Client
-----------------------------

In S-Store (and most streaming system), order of execution is extremely important.  Data must be processed in the order in which it arrives.  S-Store executes and orders its dataflow processing in terms of batches.  A batch can be considered a group of one or more tuples that should be executed as a single, atomic unit.  Each batch includes a batch_id, which is associated with the time at which the batch arrived.  These batch_ids are what ensure that incoming tuples are processed in order.  Batches are currently created on the client side, but the batch_ids are automatically assigned by the engine.

Programming a client is very similar to the process described in the H-Store documentation, but with some key differences.  There are two primary methods of ingesting data from the client to the engine.  The first method, similar to H-Store, is to generate new tuples directly within the client.  This method is best when data is fabricated, as the input rate can easily be controlled at runtime.  The second method is to use the StreamGenerator tool (documented here) to simulate an incoming stream.

The client consists of two major methods for repeatedly inserting new tuples into : the runLoop() method and the runOnce() method.   Within the runOnce() method, the user can define a segment of code that runs x times per second, where x is defined by the -Dclient.txnrate parameter at runtime for each client.  An example can be found below:

.. code-block:: java

	[example code for runOnce()]

The runLoop() method runs as one would expect a loop to: as many times as possible, with no hesitation.  runLoop() is best used with the streamgenerator, as it automatically ingests tuples at whatever rate the streamgenerator is producing them.  The easiest way to code in such a way that both the runOnce() and runLoop() method can be used is to place all of the inner loop code within runOnce(), and then call runOnce() repeatedly from within runLoop(), like so:

.. code-block:: java

	[example code for runLoop()]

As new tuples arrive, it is up to the client to group them into batches.  This is done by creating a String array with a size equal to the maximum number of tuples that you intend to send per batch.  In each iteration of the loop, the runOnce method takes in a new tuple and adds it to a batch.  When an entire batch is ready, that batch is submitted to the system by calling the client.initStream(Callback, ProcName, Tuples) method.  An example of this can be found below.

.. code-block:: java

	[example code for batching]

runOnce()/runLoop() can easily be connected to the StreamGenerator using a clientSocket and BufferedInputStream, as shown below:

.. code-block:: java

	[example code for taking in an ingestion stream]


Creating Tables, Windows, and Streams
-------------------------------------

As is the case in H-Store, application schemas are defined in a DDL file.  The DDL file must be named the same as your benchmark, followed by "-ddl.sql".

There are three primary types of state in S-Store applications: Tables, Streams, and Windows.  All three types of state are defined as tables, and all three are fully recoverable.

**Tables** constitute the primary "shared mutable state" of S-Store.  Any publicly writeable data (accessible to all OLTP or ad-hoc queries) should be defined in a table.  Creating tables is identical to both VoltDB and H-Store.  The table schema and any indexes are defined as in the example below:

.. code-block:: sql
	
	[example Table code]

.. Note:: Partition keys for tables are defined in the ProjectBuilder class.

**Streams** are the primary method of moving information from one stored procedure to another within a dataflow graph.  While the data is primarily passed through stored procedure arguments, it is important to also store the data in persistent streams as well for recovery purposes.  Streams are logically append and remove only.  For now, it is left to the application developer to prevent any updates to data items in a stream.  An example of a stream is shown below.  

.. code-block:: sql

	[example Stream code]

.. Note:: Automatic garbage collection on Streams is left to future functionality.  The application developer should ensure that expired data items within Streams are garbage collected once the tuples are no longer needed (i.e. once the downstream SP has committed).

**Windows** hold a fixed quantity of 


Creating windows
2 types - tuple-based and batch-based
Create window on stream
NOTE - windows are separate data structure
Special columns - nullable
Tuple-based - nothing special
Batch-based - required ts column

Creating SPs
------------

Explanation of what an SP is and how they relate to transactions

Mandatory arguments
Part_id - links to partitionNum
VoltStream - all upstream data passes through VoltStream
Extra_args - all other arguments that the procedure needs, passed as longs (?)

Writing SQL statements

Executing batches of SQL statements

Return success code

Border SPs vs downstream SPs - statistics on the client side

Creating a Dataflow Graph
-------------------------

Like most streaming systems, the main method of programming a workload in S-Store is via dataflow graphs.  A dataflow graph in S-Store is a series of Stored Procedures which are connected via streams in a directed acyclic graph.  

[image of dataflow graph]
[dataflow graph caption]

Explain how a dataflow works by default
Each SP triggers downstream SPs by default, even if no data is passed
Batch ordering
Exactly-once

Describe how to program link stored procedures in a benchmark
Name the dataflow graph
Indicate next SP
Indicate previous SP

Missing functionalities
Unable to fork a dataflow graph
Unable to merge dataflow sources

Passing Data Along Streams
--------------------------

Downstream SPs are triggered whether new data is passed or not

Stream data is passed via arguments to SP, along with batch_id

Two options:
Pass data
How to explicitly pass data
Keep same batch ID
Data needs to be inserted into a stream table
Manual deletion of stream data (missing functionality: garbage collection)

Don’t pass data
Still use same batchID
“NULL” tuple - define way of recognizing
Developer’s job to manage null tuples
Future functionality - option to skip null tuple SPs, management of NULL tuples
