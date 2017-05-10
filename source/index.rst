.. S-Store documentation master file, created by
   sphinx-quickstart on Thu Mar 23 18:14:22 2017.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

#######
S-Store
#######

.. toctree::
   :maxdepth: 2
   :numbered: 
   :hidden:

   intro
   deploy
   benchmarks
   engine
   bigdawg
   streamgenerator
   statistics
   features

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

S-Store is designed for a variety of streaming use cases that involve shared mutable state, including real-time data ingestion, heartrate waveform analysis, and bicycle sharing applications, to name a few.  To learn more about applications, the transaction model, and design of S-Store, please read our publications at `sstore.cs.brown.edu <https://sstore.cs.brown.edu/about.html>`_.

S-Store is built on top of H-Store, a distributed main-memory OLTP database.  You can read more about H-Store `here <https://hstore.cs.brown.edu>`_.


A simple example
-----------------

S-Store comes with a number of benchmarks, including a simple streaming example meant to showcase the functionalities of S-Store.  This benchmark, votersstoreexample, mimics an online voting competition in which the audience votes for their favorite contestant, a sliding window is generated of the current leaderboard, and periodically, based on who has the least votes in that moment, a contestant is removed from the running.

.. image:: images/voter3sp.png
   :height: 300px
   :width: 600px
   :align: center

This workload can be broken down into three stored procedures: Vote (collect the audience's votes), Generate Leaderboard (update the sliding window), and Delete Contestant (remove the lowest contestant every X votes).  

Get the code
-------------

S-Store is licensed under the terms of the GNU Affero General Public License Version 3 as published by the Free Software Foundation. See the `GNU Affero General Public License <http://www.gnu.org/licenses/>`_ for more details.  All software is provided as-is.

S-Store can be downloaded on `GitHub <https://github.com/s-store/sstore-soft>`_


