.. _bigdawg:

*****************************
Connecting S-Store to BigDAWG
*****************************

What is BigDAWG?
----------------

BigDAWG is a research polystore system developed by Intel.  It supports heterogeneous database engines, multiple programming languages and complex analysis on a variety of workloads.  BigDAWG provides a single user interface for querying several systems, allowing a user to potentially request data from multiple systems within a single query.  It also contains the ability to easily and safely migrate data from one system to another.  More information on BigDAWG is available on the `BigDAWG website <http://bigdawg.mit.edu>`_

As a transactional streaming system, S-Store is able to serve several roles within BigDAWG.  It can be used as a main-memory relational engine, much like its parent system, H-Store.  It can be used as a pure streaming system.  Or, if used as a hybrid of the two, S-Store is able to serve as a streaming data ingestion engine, able to transform incoming data items as they arrive and then migrate them to the appropriate engine.

Benchmark
---------

BigDAWG features a sample workload operating on the MIMIC II dataset.  We demonstrate the connection of S-Store and BigDAWG using the same dataset, with the S-Store benchmark mimic2bigdawg. In this configuration, S-Store is responsible for data ingestion into the polystore, specifically into Postgres. Benchmark mimic2bigdawg injects data into table medevents in S-Store, and S-Store periodically pushes data in medevents to table mimic2v26.medevents in Postgres. Analytical queries can be posted to BigDAWG. If mimic2v26.medevents is included in a query, BigDAWG will pull the data from S-Store first before executing the query in Postgres. This guarantees that the user always obtains the most fresh data injected into BigDAWG. We demonstrate this functionality by the dockerized BigDAWG and S-Store.

Setting up BigDAWG via Docker
-----------------------------

Connecting S-Store to BigDAWG is easiest using Docker containers.  Starting a BigDAWG cluster is easy, and only requires access to the BigDAWG repository.  

Start a terminal. In the terminal, check out BigDAWG, switch to the sstore-injection branch, compile and execute using the following commands:

.. code-block:: bash

    git clone https://github.com/bigdawg-istc/bigdawg.git
    cd bigdawg
    git checkout sstore-injection
    ./compile
    cd provisions
    ./setup_bigdawg_docker.sh

S-Store ingests data in a rate of 100 tuples per second, for 10 minutes by default. After the ingestion is finished, S-Store stays online.

Notes:

1. BigDAWG only compiles in JDK 8.
2. If BigDAWG is installed on Ubuntu, setup_bigdawg_docker.sh may reports errors during the setup of the BigDAWG catalog in Postgres. This is likely caused by Docker's default storage driver aufs. This can be fixed by `switching the storage driver to devicemapper <https://muehe.org/posts/switching-docker-from-aufs-to-devicemapper/>`_.
3. If BigDAWG is installed on Mac, please compile and run the setup script in Docker Quickstart Terminal as described in `the BigDAWG documentation <http://bigdawg-documentation.readthedocs.io/en/latest/getting-started.html#bigdawg-cluster-setup-steps>`_.

Querying through BigDAWG/JDBC
-----------------------------

Once BigDAWG is started, it may still take S-Store a few minutes to be ready for data ingestion. Start a second terminal window. After one to two minutes, if BigDAWG is running on Ubuntu, execute the following query in the terminal:

.. code-block:: bash

    curl -X POST -d "bdrel(select count(*) from mimic2v26.medevents;)" http://localhost:8080/bigdawg/query/

If BigDAWG is running on Mac, execute the following query in the terminal:

.. code-block:: bash

    curl -X POST -d "bdrel(select count(*) from mimic2v26.medevents;)" http://192.168.99.100:8080/bigdawg/query/

The above query shows the amount of tuples in table mimic2v26.medevents in Postgres that have been migrated from S-Store.


Pushing data from S-Store to Postgres
-------------------------------------

When BigDawg is started, it deletes the historical data in mimic2v26.medevents in Postgres by dropping the table. Once S-Store comes alive, BigDawg recreates table mimic2v26.medevents in Postgres by the table definition in S-Store, and it starts to push data from S-Store to Postgres. Currently data is pushed from S-Store to Postgres on a time-based fashion only. The time between two pushes is defined in bigdawg/profiles/dev/dev-config.properties. The name of the entry is "sstore.injection.migrationGap", with the unit of millisecond, and is set to one minute (60000 milliseconds) by default, i.e., S-Store pushes data to Postgres once per minute.


Pulling data from S-Store
-------------------------

Data in a table is pulled from S-Store to Postgres for each SQL query that requires the table. Currently we support queries that require only one table a time from S-Store for transactional safety. The support for pulling multiple tables for one query is not yet provided, but is in progress.


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

