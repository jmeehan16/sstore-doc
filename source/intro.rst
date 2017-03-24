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

Environmental Parameters
------------------------

Compiling a Benchmark
---------------------