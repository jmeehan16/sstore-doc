.. _bigdawg:

*****************************
Connecting S-Store to BigDAWG
*****************************

We demonstrate the connection of S-Store and BigDAWG with the S-Store benchmark mimic2bigdawg. In this configuration, S-Store is responsible for data ingestion. Benchmark mimic2bigdawg injects data into table medevents in S-Store, and S-Store periodically pushes data in medevents to table mimic2v26.medevents in Postgres. Analytical queries can be posted to BigDAWG. If mimic2v26.medevents is included in a query, BigDAWG will pull the data from S-Store first before executing the query in Postgres. This guarantees that the user always obtains the most fresh data injected into BigDAWG. We demonstrate this functionality by the dockerized BigDAWG and S-Store.

Manual Setup
------------

Start a terminal. In the terminal, check out BigDAWG, switch to the sstore-injection branch, compile and execute.

.. code-block:: bash

    git clone https://github.com/bigdawg-istc/bigdawg.git
    cd bigdawg
    git checkout sstore-injection
    ./compile
    cd provisions
    ./setup_bigdawg_docker.sh

S-Store ingests data in a rate of 100 tuples per second, for 10 minutes by default. After the ingestion is finished, S-Store stays online.

Querying through BigDAWG/JDBC
-----------------------------

Start a second terminal window. Execute the following query:

.. code-block:: bash

    curl -X POST -d "bdrel(select count(*) from mimic2v26.medevents;)" http://localhost:8080/bigdawg/query/

The above query shows the amount of tuples in table mimic2v26.medevents in Postgres that have been migrated from S-Store.


Pushing data from S-Store to Postgres
-------------------------------------

When BigDawg is started, it deletes the historical data in mimic2v26.medevents in Postgres by dropping the table. Once S-Store comes alive, BigDawg recreates table mimic2v26.medevents in Postgres by the table definition in S-Store, and it starts to push data from S-Store to Postgres. Currently data is pushed from S-Store to Postgres on a time-based fashion only. The time between two pushes is defined in bigdawg/profiles/dev/dev-config.properties. The name of the entry is "sstore.injection.migrationGap", with the unit of millisecond, and is set to one minute (60000 milliseconds) by default, i.e., S-Store pushes data to Postgres once per minute.


Pulling data from S-Store
-------------------------

Data in a table is pulled from S-Store to Postgres for each SQL query that requires the table. Currently we support queries that require only one table a time from S-Store for transactional safety. The support for pulling multiple tables for one query is in progress.


Pushing/Pulling data via Binanry Format
---------------------------------------

Data are migrated from S-Store to Postgres in CSV format by default. The support for binary format is in progress.


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

