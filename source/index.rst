.. S-Store documentation master file, created by
   sphinx-quickstart on Thu Mar 23 18:14:22 2017.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

#######
S-Store
#######

********************************
S-Store Documentation
********************************


Introduction
------------

Welcome to S-Store! S-Store is the world's first streaming OLTP engine, which seeks to seemlessly combine online transactional processing with push-based stream processing for real-time applications.  We accomplish this by designing our workloads as dataflow graphs of transactions, pushing the output of one transaction to the input of the next.

S-Store provides three fundamental guarantees, which together are found in no other system:

1) **ACID** - All updates to state are accomplished within ACID transactions

2) **Ordering** - S-Store executes on batches of data items, and ensures that batches are processed in an order consistent with their arrival.

3) **Exactly-once** - All operations are performed on data items once and only once, even in the event of failure

To learn more about the transaction model and design of S-Store, please read our publications at `sstore.cs.brown.edu <https://sstore.cs.brown.edu/about.html>`_.

S-Store is built on top of H-Store, and owes much of our architecture and programming model to H-Store's design.  You can read more about H-Store `here <https://hstore.cs.brown.edu>`_.


A simple example
-----------------

TPC-DI example here? Voter S-Store Example here?

Get the code
-------------

Git repository link here


Table of Contents
-----------------

.. toctree::
   :maxdepth: 2
   :numbered: 

   intro
   benchmarks
   engine
   bigdawg
   statistics