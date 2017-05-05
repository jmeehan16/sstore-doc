.. _statistics:

******************
S-Store Statistics
******************

Client-side Statistics
----------------------

By default, when running S-Store benchmarks, transaction and latency statistics are provided in the terminal at a specified interval of time (by default, every 10 seconds).  It is important to note that these statistics are recorded and managed on the client side, within the callback of the procedures.  An example of how to manage these statistics is provided below:

.. code-block:: java

	private class Callback implements ProcedureCallback {
		private final int idx; //which SP id is associated with this callback

		public Callback(int idx) {
			this.idx = idx;
		}

		public void clientCallback(ClientResponse clientResponse) {
			incrementTransactionCounter(clientResponse, idx);
		}
	}

.. Note:: These statistics are only valid for transactions that originate at the client.  Any transactions that occur later in the dataflow graph are not recorded on the client side, and must instead be viewed on the server side.

At the end of the benchmark, the S-Store client will output the transaction counter statistics to the terminal for all procedures run during the benchmark.  While these statistics do not include latency or throughput, they provide an indication of how many downstream stored procedures executed during the benchmark period.

Server-side Statistics
----------------------

In addition to the client-side statistics, the user can also view server-side statistics via the S-Store terminal.  The easiest way to view these statistics is to open a separate S-Store terminal.  To do this, use the "-Dnoshutdown=true" argument when running the benchmark.  Once the benchmark has completed but the S-Store instance is still running, open another terminal and run the interactive S-Store terminal.

If running Dockerized S-Store, use the following command.  You can find the container's ID by running "docker images":

.. code-block bash::
	docker exec -it {CONTAINER-ID} ./sstore {BENCHMARK}

If running native S-Store, you can simply run:

.. code-block:: bash

	./sstore {BENCHMARK}

Once the S-Store terminal is running, you can query a variety of statistics using H-Store's @Statistics system transaction, which is fully documented here_.

.. _here: http://hstore.cs.brown.edu/documentation/system-procedures/statistics/

The most useful of these commands are:

.. code-block:: bash

	exec @Statistics txncounter 1 #gives the transaction counter statistics
	exec @Statistics table 1 #gives statistics about the tuples in each table