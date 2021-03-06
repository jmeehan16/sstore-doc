.. _engine:

**************
S-Store Engine
**************

Batching
--------

S-Store dataflow graphs process all tuples in atomic batches, which have been defined by the client.  These batches can have zero, one, or several tuples.  Batch-ids are assigned by the client, attached to the input tuples, and sent to the engine with the transaction request.

Scheduler
---------

The scheduler is responsible for ensuring that streaming transactions execute in the proper order.  For each stored procedure, earlier batches are guaranteed to execute before later ones.  For each batch, earlier stored procedures (in the dataflow graph) are guaranteed to execute before the later ones.  The scheduler is also responsible for ensuring that each transaction executes once and only once.

In single-node S-Store, there is no opportunity for parallelism in transactions, and thus all dataflow graphs are executed serially.  This means that when one SP executes on batch B, the next SP in the dataflow graph will then immediately execute on the same batch B before any other streaming transaction occurs.  This has the added benefit of serving a similar purpose to nested transactions, as it prevents other transactions from accessing the state until a batch has completely executed within the dataflow graph.

.. Note:: In the distributed case, stored procedures are assigned to a specific "home" node for distributed scheduling.  This is future functionality.

Windows and EE Triggers
-----------------------

Under the hood, streams and windows are largely defined as extensions on SQL tables.  This ensures that all state in an application is transactional and fully recoverable.  Tuple management in windows in particular are handled in the execution engine.  New tuples are staged in the window upon insert, marked as invisible, only to be marked as visible when enough tuples/batches arrive for a window slide.

Execution engine triggers that are defined on a stream or table activate every time a new tuple is inserted.  On a window, on the other hand, the EE triggers only execute when the window slides.


Logging
--------

S-Store provides two types of logging, which can be toggled using "-Dglobal.weak_recovery" at runtime.

**Strong recovery** logging is very similar to that of H-Store/OLTP logging.  Every transaction within the dataflow graph is logged, and every OLTP transaction is logged.  This ensures that, on recovery, the transactions are replayed in the exact order that they were originally executed.  When recovering using strong recovery, the scheduler's triggering of downstream transactions must be disabled in order to ensure that transactions are not replayed twice.  Once the log has been fully replayed, then the scheduler should be re-enabled and allowed to generate transactions from any unprocessed tuples remaining in the streams.

**Weak recovery** logging, on the other hand, only records OLTP transactions and **border transactions**, or transactions at the beginning on the dataflow graph (on this particular node).  When recovering using weak recovery, the scheduler remains on, so that every transaction then triggers a downstream transaction in the dataflow graph, as would happen in normal runtime.  This recovery method guarantees that all transactions will execute exactly-once and in compliance with S-Store's ordering guarantees, but it does not necessarily ensure that the transactions will be replayed in the *exact* order as the original runtime. 

By default, S-Store uses group commit for transaction logging in order to batch write transactions to disk.  It is possible to disable group commit at runtime using the "-Dsite.commandlog_timeout=-1" option, though this will have a very large impact on performance.  If this option is chosen, weak recovery is highly recommended.

