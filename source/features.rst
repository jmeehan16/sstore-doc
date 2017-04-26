.. _features:

******************************************
Supported Features
******************************************

Below is a list of supported features in S-Store.  All features are directly implemented into the engine unless otherwise specified.  Missing features are not directly supported in this release, but are in development for future releases

GENERAL MODEL
-------------

Ability to write stored procedures							SUPPORTED
Ability to create dataflow graphs of stored procedures		SUPPORTED
Ability to pass data from one stored procedure to another	SUPPORTED
Ability to batch incoming stream data						MANUALLY WRITTEN IN APPLICATIONS
OLTP transactions											SUPPORTED
Nested transactions											LIMITED SUPPORT - SCHEDULER SERIALIZES TXNS
Stream garbage collection									MANUALLY WRITTEN IN APPLICATIONS

WINDOWS
-------
Ability to window incoming data								SUPPORTED
Ability to pass data from windows in-engine					SUPPORTED
Ability to group windowed data								FUTURE RELEASE
Ability to trigger SPs with window							MANUALLY WRITTEN IN APPLICATIONS
Protection limiting window access from other SPs			MANUALLY WRITTEN IN APPLICATIONS
Window garbage collection									SUPPORTED

ENGINE
------
Stored procedures execute ACID transactionally				SUPPORTED
Ordered execution for dataflow graphs of SPs				SUPPORTED
Proper triggering from one SP to next in dataflow graph		SUPPORTED
Exactly-once - No repeat transactions						SUPPORTED
Exactly-once - Retry all aborted txns immediately			FUTURE RELEASE

RECOVERY
--------
Logging for weak recovery									SUPPORTED
Logging for strong recovery									SUPPORTED
Recovery for weak logging									MANUALLY WRITTEN IN APPLICATIONS
Recovery for strong logging									SUPPORTED
Snapshotting												FUTURE RELEASE
Group commit												SUPPORTED
Non-group commit											SUPPORTED

BIG DAWG
--------
Ability to query through JDBC								SUPPORTED
Ability to connect to BigDAWG								FUTURE RELEASE (COMING SOON)                      
Ability to query from BigDAWG								FUTURE RELEASE (COMING SOON)
Ability to migrate data from S-Store to Postgres			FUTURE RELEASE (COMING SOON)
Ability to migrate data from Postgres to S-Store			FUTURE RELEASE (COMING SOON)

STATISTICS
----------
Transaction counter (engine)								SUPPORTED
Transaction counter (client)								BORDER TXNS ONLY
Transaction Latency											BORDER TXNS ONLY
Dataflow statistics											FUTURE RELEASE
Table statistics											SUPPORTED

UNIT TESTS
Scheduler
Do transactions execute in batch order (per stored procedure)?
Do transactions execute in dataflow order (per batch)?
If a transaction is submitted multiple times, does it only execute once?
