.. _streamgenerator:

****************************
Stream Generator Tool
****************************

The Stream Generator is a simple tool able to take a flat CSV file and convert it to a stream of tuples, represented by strings.  It is primarily used to simulate streams of tuples when no actual stream is available.

To install the Stream Generator, navigate to the tools/streamgenerator directory.  Simply run the command:

.. code-block:: bash

	mvn install

..Note:: You will need to install mvn if you have not already.

To use the Stream Generator, it is recommended that you use the "stream-generator-v1-jar-with-dependencies.jar."  A typical use looks like this:

.. code-block:: bash

	java -jar target/streamgenerator.jar -f <file> -t <throughput> -d <duration> -p <port> -m <maxtupels>

The parameters:

- **-f <file>:** the CSV file that contains the data to stream
- **-t <throughput>:** the rate at which tuples are being sent to the system (per second)
- **-d <duration>:** the amount of time that the streamgenerator will run
- **-p <port>:** the port that the stream data will be sent to
- **-m <maxtuples>:** the maximum number of tuples that the streamgenerator will send

..Note:: The application/benchmark will need to be configured to receive tuples from the streamgenerator, and the benchmark configuration run with the setting "-Dclient.txnrate=-1" in order to receive as many tuples as possible from the stream.

..Note:: Batch-ids are assigned in the client and should not be pre-assigned by the streamgenerator.