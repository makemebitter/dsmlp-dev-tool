### (WIP) DSMLP example

#### Overview

This is an example toolkit to spawn a Spark cluster on UCSD DSMLP. If you are using other softwares such as Dask or Ray, you will need to start from here. Build your own docker image, and write your own script to utilize DSMLP's Kubernetes interface fro your project.

#### Directory structure 

```bash
cluster-manager.sh	# The cluster manager script to spwan and kill a Spark cluster
spark-cluster.yaml.template # Kubernetes template required by cluster-manager.sh
DSMLP_spark.pdf # Step-to-step guide of spwaning a Spark cluster on DSMLP
tex # Src tex file for DSMLP_spark.pdf
docker # Directory for the docker files
|--	Dockerfile # The docker file, bootstrap script
|-- common.sh # Startup steps that are common for both master and workers
|-- spark-master # Startup script for master
|-- spark-worker # startup script for workers
|-- requirements.txt # Python dependencies
|-- spark-defaults.conf # Spark-specific configuration file
```

#### Get started

Here we go through some general steps you need to take to configure your own cluster that runs Dask, Ray, or any other distributed framework. Everything below is standard Kubernetes, if you are already familiar with it, feel free to skip. 

##### 1. Try the Spark example
First of all, try to access DSMLP and spwan a cluster with Spark pre-configured. Read `DSMLP_spark.pdf` for a step-to-step guide.

##### 2. Build your own image
First step you will need to build your own docker container image that has all the required softwares installed. If you are not familiar with [docker](https://docs.docker.com/), now is a good time to learn more about it. We have an example for Spark in the `docker` folder. Modify these to install and configure your required softwares. 

To be more specific, in `Dockerfile`,  install your dependencies and configure the softwares. In `spark-master, spark-worker` put the node startup code (and modify the filenames if you want). 

Build and test your image, then publish it to [Docker Hub](https://hub.docker.com/).

Note: do this step on your local machine instead of the DSMLP login node, which is not supposed to execute any complex program and does not have docker installed.
##### 3. Modify the Kubernetes template

Next you need to modify the `spark-cluster.yaml.template` for your project. More specifically, you need to: 

-   modify the `image` to point to your custom image. Also you need to modify all the port-related configs to allow your nodes to communicate. 
-   Do not forget to change the `Service` as well. All these port settings will be project-dependent, you need to figure out what ports your frameworks use and expose them accordingly.
-   By default, the master node requests 2 CPU cores and 4 GB RAM, while the worker nodes are 2 CPU cores with 20 GB memory. You can modify these for your project. 
    -   In most cases, you will not need to change the master's setting as it typically is not involved in any computation.
    -    No GPU is enabled here, for GPU support, check the dedicated section below.

##### 4. Modify the cluster manager

Finally, you can modify the cluster manager `cluster-manager.sh`. 

-   Mostly you need to change the port-forwarding settings for your project-specific frameworks. These are usually the debug/monitoring/dashboard utilities that the framework comes with. Check their docs for details.
-   **WARNING:** The script spans only one master and one worker by default; stick to it when you are still developing/trying things out. Spawn more workers only when you are benchmarking the performance.

##### 5. Optional GPU support

You do not need to worry about this if your project does not involve GPUs.

To request GPUs to your workers, add the following to the worker's resource request in the Kubernetes template file:

```yaml

```

**WARNING:** proceed with caution when you use GPUs, as these resources are usually at high demand. Delete your cluster whenever you are not using it. Never start developing directly with GPUs on. First 





