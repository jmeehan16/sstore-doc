.. _bigdawg:

*****************************
Connecting S-Store to BigDAWG
*****************************

We demonstrate the connection of S-Store and BigDawg with the S-Store benchmark mimic2bigdawg. The benchmark injects data into table medevents in S-Store, and S-Store periodically pushes data in medevents to table mimic2v26.medevents in the analytical engine of Postgres. Analytical queries can be posted to BigDawg. If mimic2v26.medevents is included in the query, BigDawg will pull the data from S-Store first before executing the query in the Postgres engine. This guarantees that the user always obtains the most fresh data injected into BigDawg. We demonstrate this functionality by the dockerized BigDawg and S-Store.

Manual Setup
------------
1. Prepare the S-Store benchmark mimic2bigdawg.
2. Compile the stream ingestor.
3. Download the csv file for the ingestion of medevents.
4. Start the stream ingestor.
5. Start S-Store with benchmark mimic2bigdawg.

Querying through BigDAWG/JDBC
-----------------------------


Pushing data from S-Store to Postgres
-------------------------------------


Pulling data from S-Store
-------------------------


Pushing/Pulling data via Binanry Format
---------------------------------------


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

