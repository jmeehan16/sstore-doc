.. _deploy:

*******************************
Deploying and Executing S-Store
*******************************

Quick Start (Dockerized)
------------------------

The easiest way to build an S-Store instance is using a Docker image.  Docker is a software container platform designed to allow code to run in a virtual container, with all requirements for installation already included.  The biggest advantage to this method is that you do not need to worry about the requirements for S-Store, but instead can run an instance on any system that can run Docker.  To learn more about how to install Docker on your specific system, visit https://www.docker.com/

1. Clone S-Store from github using the following command:

.. code-block:: bash

	git clone http://github.com/s-store/s-store.git

2. Once you have Docker installed, you can then build the S-Store image. In a terminal, change to the directory that you just cloned S-Store into, and run the following command. This will install all necessary packages, compile S-Store, and prepare the benchmark votersstoreexample by default. 

.. code-block:: bash

	docker build -t s-store ./

3. Once S-Store has been built, use the following command to run s-store image. This command will run the benchmark votersstoreexample with default parameters and show the statistics of the votersstoreexample.

.. code-block:: bash

	docker run s-store

4. If you wish to run a different benchmark on the S-Store image in a non-interactive way, you can use the following command.

.. code-block:: bash

	docker run s-store /bin/bash -c "service ssh restart && ant sstore-prepare -Dproject={BENCHMARK} && ant sstore-benchmark -Dproject={BENCHMARK}"

.. Note:: A good example benchmark to begin is votersstoreexample, which highlights the core functionalities available in S-Store.

4. If you wish to run commands on S-Store in an interactive way, you will need to run two terminals.  In the first terminal, use:

.. code-block:: bash

	docker run -it s-store /bin/bash
	service ssh restart && ant sstore-prepare -Dproject={BENCHMARK} && ant sstore-benchmark-console -Dproject={BENCHMARK}

[insert screenshot here]

Then, in a second terminal, you will need to connect to the running container.  The container's ID can be obtained running "docker images"

.. code-block:: bash

	docker exec -it {CONTAINER-ID} ./sstore {BENCHMARK}

[insert screenshot here]

Once connected to this second terminal, you can run SQL statements in order to query the database.  There are also a variety of statistics tools available as well.

5. Some other useful docker commands that you might want to use:

List all images and detailed information:

.. code-block:: bash

	docker images

Check active and inactive containers and obtain containers'id:

.. code-block:: bash

	docker ps -a


Manual Start (Environment Setup on Native Linux)
------------------------------------------------

S-Store is easy to set up on any Linux machine, and is recommended as the easiest method of developing new benchmarks.  You will need a **64-bit version of Linux** with at least 2 cores and a recommended 6 GB of RAM available.  Native S-Store has the same requirements as its parent system, H-Store.  These are:

- gcc/g++ +4.3
- JDK 1.6/1.7
- Python +2.7
- Ant +1.7
- Valgrind +3.5

.. Note:: S-Store does **not** support JDK 1.8 at this time.  You will need to use JDK 1.6 or 1.7.  If you are running a machine with JDK 1.8 installed, you can either install 1.7 alongside it, or install S-Store within a virtual machine.

1. Install the required packages with the following commands:

.. code-block:: bash

	sudo apt-get update
	sudo apt-get --yes install subversion gcc g++ openjdk-7-jdk valgrind ant

2. In order to run S-Store, your machine needs to have OpenSSH enabled and you must be allowed to login to localhost without a password:

.. code-block:: bash

	sudo apt-get --yes install openssh-server
	ssh-keygen -t rsa # Do not enter a password
	cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

Execute this simple test to make sure everything is set up properly:

.. code-block:: bash

	ssh -o StrictHostKeyChecking=no localhost "date"

You should see the date printed without having to put in a password.  If this fails, then check your permissions in the ~/.ssh/ directory.

The S-Store source code can be downloaded from the Github repository using the following command:

.. code-block:: bash

	git clone http://github.com/jmeehan16/s-store.git

Once the code is downloaded and the desired branch selected, run the following command on the root directory of S-Store:

.. code-block:: bash

	ant build

.. Note:: This will build all of the portions of the S-Store codebase.  Depending on the development environment, this can take a good bit of time.  If your development is limited to benchmarks only, it is much quicker to simply rebuild the Java portion of the codebase using "ant build-java".

.. Note:: S-Store must be run on a 64 bit Linux machine, preferably with at least 6 GB of RAM. If you have a Mac or Windows machine, I recommend installing a virtual machine using a free service such as VirtualBox.  VirtualBox can be downloaded at `www.virtualbox.org <https://www.virtualbox.org/>`_.

Compiling and Executing a Benchmark
-----------------------------------

Executing S-Store is very similar to executing H-Store, documented here. All commands, including **hstore-prepare**, **hstore-benchmark**, **catalog-info**, and **hstore-invoke** work as expected, in addition to the **hstore terminal tool**, which can be extremely helpful to view what actually exists in each table.

When running S-Store on a single node, these are the commands you will want to run. Note that you will need to recompile each time you make changes to your code.

.. code-block:: bash

	ant clean-java build-java
	ant sstore-prepare -Dproject=$benchmarkname
	ant sstore-benchmark -Dproject=$benchmarkname $parameters

Or simply use the included shell script, which will run each command for you:

.. code-block:: bash

	./runsstorev1.sh $benchmarkname $txnspersecond "other parameters here"

The runsstorev1.sh shell script uses a number of parameters that are desired by most S-Store runs, including the use of a single non-blocking client and disabling logging. If you want to run the script without those parameters, you can easily override them by re-adding the parameters with your desired values.

Interacting with a Live Database
--------------------------------

Like most databases, it is possible to interact directly with a live S-Store database.  Because S-Store is a main-memory database, it will need to reload data into its table objects every time it restarts.  To interact with an S-Store database, you can run an existing benchmark in a way that does not shut down the system once the data has been loaded.  The easiest way to do this is to use the following command:

.. code-block:: bash

	ant sstore-benchmark-console -Dproject=$benchmarkname $parameters

This will automatically set the "noshutdown" parameter to true.  Once S-Store is running, open another terminal window in the same root directory as S-Store.  From there, you can open an interactive S-Store terminal by running (in a new terminal!):

.. code-block:: bash

	./sstore $benchmarkname

From this interactive terminal, you can run adhoc SQL statements, as well as `statistics_ <http://hstore.cs.brown.edu/documentation/system-procedures/statistics/>`_ transactions.  This terminal window can remain open even once S-Store is stopped, and will automatically reconnect to a new S-Store instance run from the same root directory.  However, clearly you will be unable to query the database when it is not running.


Environmental Parameters
------------------------

S-Store adds a number of enviroment parameters to H-Store's base parameters.  To use these parameters at runtime, use "-D" and then the parameter name (for instance, "-Dclient.txnrate=[txnrate]").  A full list of H-Store's parameters can be found here:

- `Global Parameters`_
- `Site Parameters`_
- `Client Parameters`_

.. _Global Parameters: http://hstore.cs.brown.edu/documentation/configuration/properties-file/global/
.. _Site Parameters: http://hstore.cs.brown.edu/documentation/configuration/properties-file/site/
.. _Client Parameters: http://hstore.cs.brown.edu/documentation/configuration/properties-file/client/

Some of the most helpful S-Store parameters are listed below:

**client.txnrate**:

- Default: 1000
- Permitted Type: integer
- Indicates the number of transactions per second that are being submitted to the engine (per client).  If using the streamgenerator, it is recommended that you set this parameter to "-1", as this will cause the client to send as many transaction requests per second as are provided by the streamgenerator.

**client.threads_per_host**:

- Default: 1
- Permitted Type: integer
- Indicates the number of client threads that will be submitting transaction requests to the engine.

**client.duration**:

- Default: 60000
- Permitted Type: integer
- Indicates the period of time the benchmark will run, in milliseconds.

**client.benchmark_param_0**:

- Default: 0
- Permitted Type: integer
- Generic input parameter that can be used within a benchmark.

**client.benchmark_param_str**:

- Default: NULL
- Permitted Type: String
- Generic input parameter that can be used within a benchmark.

**site.commandlog_enable**:

- Default: false
- Permitted Type: boolean
- Indicates whether commands are being logged to disk.

**noshutdown**:

- Default: false
- Permitted Type: boolean
- Keeps S-Store running, even after the benchmark has completed.

**noexecute**:

- Default: false
- Permitted Type: boolean
- Causes the benchmark to run, but no requests to be sent from the client.

There are several S-Store-specific parameters as well. They are:

**global.sstore**:

- Default: true
- Permitted Type: boolean
- Enables S-Store and its related functionality.  When set to false, the system should operate as pure H-Store.

**global.sstore_scheduler**:

- Default: true
- Permitted Type: boolean
- Enables the serial scheduler, which ensures that when a procedure triggers another procedure, that transaction is scheduled before any other. 

**global.weak_recovery**:

- Default: true
- Permitted Type: boolean
- Enables the weak recovery mechanism, which only logs the "border" stored transactions that exist at the beginning of a workflow.  If not enabled, then strong recovery is used instead.

**global.sstore_frontend_trigger**:

- Default: true
- Permitted Type: boolean
- Enables frontend (PE) triggers.

**client.input_port**:

- Default: 21001
- Permitted Type: integer
- Specifies which port the streamgenerator should connect to

**client.input_host**:

- Default: "localhost"
- Permitted Type: String
- Specifies which hostname the streamgenerator should connect to

**client.bigdawg_port**:

- Default: 21002
- Permitted Type: integer
- Specifies the port to be used to connect to BigDAWG
