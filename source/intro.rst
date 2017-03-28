.. _intro:

****************************
Introduction to S-Store
****************************

Quick Start (Dockerized)
------------------------

The easiest way to build an S-Store instance is using a Docker image.  The biggest advantage to this method is that you do not need to worry about the requirements for S-Store, but instead can run an instance on any system that can run Docker.  To learn more about how to install Docker, visit https://www.docker.com/

1. Clone S-Store from github using the following command:

.. code-block:: bash

	git clone http://github.com/jmeehan16/s-store.git

2. Once you have Docker installed, you will then build the S-Store image. In a terminal, change to the directory that you just cloned S-Store into, and run the following command. This will install all necessary packages, compile S-Store, and prepare the benchmark votersstoreexample by default. 

.. code-block:: bash

	docker build -t s-store ./

3. Once S-Store has been built, use the following command to run s-store image. This command will run the benchmark votersstoreexample with default parameters and show the statistics of the votersstoreexample.

.. code-block:: bash

	docker run s-store

4. If you wish to run a different benchmark on the S-Store image in a non-interactive way, you can use the following command.

.. code-block:: bash

	docker run s-store /bin/bash -c "service ssh restart && ant sstore-prepare -Dproject={BENCHMARK} && ant sstore-benchmark -Dproject={BENCHMARK}"

4. If you wish to run commands on S-Store in an interactive way, you will need to run two terminals.  In the first terminal, use:

.. code-block:: bash

	docker run -it s-store /bin/bash
	service ssh restart && ant sstore-prepare -Dproject={BENCHMARK} && ant sstore-benchmark-console -Dproject={BENCHMARK}

Then, in a second terminal, you will need to connect to the running container.  The container's ID can be obtained running "docker images"

.. code-block:: bash

	docker exec -it {CONTAINER-ID} ./sstore {BENCHMARK}

5. Some other useful docker commands that you might want to use:

List all images and detailed information:

.. code-block:: bash

	docker images

Check active and inactive containers and obtain containers'id:

.. code-block:: bash

	docker ps -a


Manual Start (Run on Native Linux)
----------------------------------

In order to run The S-Store source code can be downloaded from the Github repository using the following command:

.. code-block:: bash

	git clone http://github.com/jmeehan16/s-store.git

Once you have downloaded the source code, you should create a new branch for your group using:

.. code-block:: bash

	git checkout -b "your branch name"

From there, follow the environmental setup instructions and the quick start instructions located at the H-Store webpage. Unless otherwise specified, the instructions are followed exactly.

.. Note:: S-Store must be run on a 64 bit Linux machine, preferably with at least 6 GB of RAM. If you have a Mac or Windows machine, I recommend installing a virtual machine using a free service such as VirtualBox.

Compiling and Executing a Benchmark
-----------------------------------

Executing S-Store is very similar to executing H-Store, documented here. All commands, including **hstore-prepare**, **hstore-benchmark**, **catalog-info**, and **hstore-invoke** work as expected, in addition to the **hstore terminal tool**, which can be extremely helpful to view what actually exists in each table.

When running S-Store on a single node, these are the commands you will want to run. Note that you will need to recompile each time you make changes to your code.

.. code-block:: bash

	ant clean-java build-java
	ant sstore-prepare $benchmarkname
	ant sstore-benchmark $benchmarkname $parameters

Or simply use the included shell script, which will run each command for you:

.. code-block:: bash

	./runsstorev1.sh $benchmarkname $txnspersecond "other parameters here"

The runsstorev1.sh shell script uses a number of parameters that are desired by most S-Store runs, including the use of a single non-blocking client and disabling logging. If you want to run the script without those parameters, you can easily override them by re-adding the parameters with your desired values.


Environmental Parameters
------------------------

S-Store adds a number of enviroment parameters to H-Store's base parameters:

GlobalParameters_: http://hstore.cs.brown.edu/documentation/configuration/properties-file/global/
SiteParameters_: http://hstore.cs.brown.edu/documentation/configuration/properties-file/site/
ClientParameters_: http://hstore.cs.brown.edu/documentation/configuration/properties-file/client/

There are a few S-Store-specific parameters as well. They are:

**global.sstore**::

	Default: true
	Permitted Type: boolean
	Enables S-Store and its related functionality.

**global.sstore_scheduler**::

	Default: true
	Permitted Type: boolean
	Enables the serial scheduler, which ensures that when a procedure triggers another procedure, that transaction is scheduled before any other. 

.. Note:: the serial scheduler is designed for workflows that run on a single node. It will be replaced by nested transactions.

**global.weak_recovery**::

	Default: true
	Permitted Type: boolean
	Enables the weak recovery mechanism, which only logs the "border" stored transactions that exist at the beginning of a workflow.

**global.sstore_frontend_trigger**::

	Default: true
	Permitted Type: boolean
	Enables frontend (PE) triggers.