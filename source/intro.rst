.. _intro:

****************************
Introduction to S-Store
****************************

Quick Start (Dockerized)
------------------------

# docker
1.Command to build the s-store image. 
Change to the directory containing s-store Dockerfile, and run the fllowing command which install all necessary packages, and compile s-store and prepare the benchmark votersstoreexample by default. 

.. code-block:: bash

	docker build -t s-store ./

2.Command to run s-store image, this command will run the benchmark votersstoreexample and show statistics of the votersstoreexample

.. code-block:: bash

	docker run s-store

3.Command to run s-store image with specified benchmark in a non-interactive way

.. code-block:: bash

	docker run s-store /bin/bash -c "service ssh restart && ant sstore-prepare -Dproject={BENCHMARK} && ant sstore-benchmark -Dproject={BENCHMARK}"

4.Command to run s-store in an interactive way
Open a new terminal:

.. code-block:: bash

	docker run -it s-store /bin/bash
	service ssh restart && ant sstore-prepare -Dproject={BENCHMARK} && ant sstore-benchmark-console -Dproject={BENCHMARK}

Open another terminal. Containers' id can be obtained using command 5

.. code-block:: bash

	docker exec -it {CONTAINER-ID} ./sstore {BENCHMARK}

5. Some userful commands you might want to use:
List all images and detailed information:

.. code-block:: bash

	docker images

Check active and inactive containers and obtain containers'id:

.. code-block:: bash

	docker ps -a


Manual Start (Run on Native Linux)
----------------------------------

The S-Store source code can be downloaded from the private Github repository using the following command:

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
	ant hstore-prepare $benchmarkname -Dhosts="localhost:0:0"
	ant hstore-benchmark $benchmarkname $parameters

Or simply use the included shell script, which will run each command for you:

.. code-block:: bash

	./runsstorev1.sh $benchmarkname $txnspersecond "other parameters here"

The runsstorev1.sh shell script uses a number of parameters that are desired by most S-Store runs, including the use of a single non-blocking client and disabling logging. If you want to run the script without those parameters, you can easily override them by re-adding the parameters with your desired values.


Environmental Parameters
------------------------

S-Store adds a number of enviroment parameters to H-Store's base global, client, and site parameters. These are all enabled by default, and most likely you will not need to disable them. They are:

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