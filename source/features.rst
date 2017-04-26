.. _features:

******************************************
Supported Features
******************************************

Below is a list of supported features in S-Store.  All features are directly implemented into the engine unless otherwise specified.  Missing features are not directly supported in this release, but are in development for future releases

**GENERAL MODEL:**

- Write stored procedures: **SUPPORTED**
- Create dataflow graphs of stored procedures: **SUPPORTED**
- Pass data from one stored procedure to another: **SUPPORTED**
- Batch incoming stream data: **MANUALLY WRITTEN IN APPLICATIONS**
- OLTP transactions: **SUPPORTED**
- Nested transactions: **LIMITED SUPPORT - SCHEDULER SERIALIZES TXNS**
- Stream garbage collection: **MANUALLY WRITTEN IN APPLICATIONS**

**WINDOWS:**

- Window incoming data: **SUPPORTED**
- Pass data from windows in-engine: **SUPPORTED**
- Group windowed data: **FUTURE RELEASE**
- Trigger SPs with window: **MANUALLY WRITTEN IN APPLICATIONS**
- Limiting window access from other SPs: **MANUALLY WRITTEN IN APPLICATIONS**
- Window garbage collection: **SUPPORTED**

**ENGINE:**

- Stored procedures execute ACID transactionally: **SUPPORTED**
- Ordered execution for dataflow graphs of SPs: **SUPPORTED**
- Proper triggering from one SP to next in dataflow graph: **SUPPORTED**
- Exactly-once - No repeat transactions: **SUPPORTED**
- Exactly-once - Retry all aborted txns immediately: **FUTURE RELEASE**

**RECOVERY:**

- Logging for weak recovery: **SUPPORTED**
- Logging for strong recovery: **SUPPORTED**
- Recovery for weak logging: **MANUALLY WRITTEN IN APPLICATIONS**
- Recovery for strong logging: **SUPPORTED**
- Snapshotting: **FUTURE RELEASE**
- Group commit: **SUPPORTED**
- Non-group commit: **SUPPORTED**

**BIG DAWG:**

- Query through JDBC: **SUPPORTED**
- Connect to BigDAWG: **FUTURE RELEASE (COMING SOON)**
- Query from BigDAWG: **FUTURE RELEASE (COMING SOON)**
- Migrate data from S-Store to Postgres: **FUTURE RELEASE (COMING SOON)**
- Migrate data from Postgres to S-Store: **FUTURE RELEASE (COMING SOON)**

**STATISTICS:**

- Transaction counter (engine): **SUPPORTED**
- Transaction counter (client): **BORDER TXNS ONLY**
- Transaction Latency: **BORDER TXNS ONLY**
- Dataflow statistics: **FUTURE RELEASE**
- Table statistics: **SUPPORTED**

