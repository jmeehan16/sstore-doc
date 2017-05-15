.. _bigdawg:

*****************************
Connecting S-Store to BigDAWG
*****************************

We demonstrate the connection of S-Store and BigDAWG with the S-Store benchmark mimic2bigdawg. In this configuration, S-Store is responsible for data ingestion. Benchmark mimic2bigdawg injects data into table medevents in S-Store, and S-Store periodically pushes data in medevents to table mimic2v26.medevents in the analytical engine of Postgres. Analytical queries can be posted to BigDAWG. If mimic2v26.medevents is included in the query, BigDAWG will pull the data from S-Store first before executing the query in the Postgres engine. This guarantees that the user always obtains the most fresh data injected into BigDAWG. We demonstrate this functionality by the dockerized BigDAWG and S-Store.

Manual Setup
------------
We will open three terminals for the setup.

1. In the first termianl, prepare the S-Store benchmark mimic2bigdawg.

.. code-block:: bash

    docker run s-store /bin/bash -c "ant sstore-prepare -Dproject=mimic2bigdawg -Dhosts=localhost:0:0"

2. In the same terminal, compile the stream generator.

.. code-block:: bash

    docker run -it s-store /bin/bash
    cd /root/s-store/tools/streamgenerator
    mvn install

3. In the same terminal, download the csv file for the ingestion of medevents.

.. code-block:: bash

    cd /root/s-store/tools/streamgenerator/target
    wget http://www.cs.toronto.edu/~jdu/data/medevents.csv

4. In the same terminal, start the stream generator. The following command will start to generate data stream for table medevents with the throughput of 100 tuples per second, for 10 minutes, and send to port 18000.

.. code-block:: bash

    cd /root/s-store/tools/streamgenerator/target
    java -jar stream-ingestor-0.0.1-SNAPSHOT-jar-with-dependencies.jar -f medevents.csv -t 100 -d 600000 -p 18000

5. Start a second terminal. In the second terminal, start S-Store benchmark mimic2bigdawg.

.. code-block:: bash

    docker run -it s-store /bin/bash
    cd /root/s-store
    ./tools/sstore/testsstore.sh mimic2bigdawg -1 "-Dnoshutdown=true"

6. Start a third terminal. In the third terminal, check out and start BigDAWG.

.. code-block:: bash

    git clone https://github.com/bigdawg-istc/bigdawg.git
    cd bigdawg
    git checkout sstore-injection
    cd provisions
    ./setup_bigdawg_docker.sh

Querying through BigDAWG/JDBC
-----------------------------

Start a terminal window. Execute the following query:

.. code-block:: bash

    curl -X POST -d "bdrel(select count(*) from mimic2v26.medevents;)" http://192.168.99.100:8080/bigdawg/query/


Pushing data from S-Store to Postgres
-------------------------------------

S-Store starts to push data to Postgres once both S-Store and BigDAWG are started and alive. Currently data is pushed from S-Store to Postgres on a time-based fashion only. The time between two pushes is defined in bigdawg/profiles/dev/dev-config.properties. The name of the entry is "sstore.injection.migrationGap", with the unit of millisecond, and is set to one minute (60000 milliseconds) by default, i.e., S-Store pushes data to Postgres once every one minute.


Pulling data from S-Store
-------------------------

Data in a table is pulled from S-Store to Postgres for each query that requires the table. Currently we support queries that require only one table from S-Store.


Pushing/Pulling data via Binanry Format
---------------------------------------

Data are migrated from S-Store to Postgres in CSV format by default.


Known limitations
-----------------



..
	Quick Start (Dockerized)
	------------------------

	Manual Setup
	------------

	Querying through BigDAWG/JDBC
	-----------------------------

	Migrating data from S-Store to Postgres
	---------------------------------------

	Migrating data to S-Store from Postgres
	---------------------------------------

	Migrating via CSV
	-----------------

