<h1>Run RiboViz with Docker</h1>

This documentation shows how the user can run RiboViz with Docker.
It is recommended to run containerized RiboViz with Singularity, another containerize tool targets on HPC systems.
However, we still provide a docker version of RiboViz, for the Docker has a very large community and detailed documentations.
The guide to run RiboViz with Singularity can be found at [here](./run-with-singularity.md).

- [Basic information of Docker and running Docker images](#basic-information-of-docker-and-running-docker-images)
  - [Docker, Docker Image and Docker Container](#docker-docker-image-and-docker-container)
  - [Useful Commands to Use Docker](#useful-commands-to-use-docker)
    - [List all Images/Containers](#list-all-imagescontainers)
    - [Export Image to File](#export-image-to-file)
    - [Restore an Image from File](#restore-an-image-from-file)
    - [Build an Image from File](#build-an-image-from-file)
    - [Run commands in an Image (by creating a container)](#run-commands-in-an-image-by-creating-a-container)
    - [Run commands in a Container](#run-commands-in-a-container)
    - [Start or Stop a Container](#start-or-stop-a-container)
- [0. Install Docker on your PC](#0-install-docker-on-your-pc)
    - [IMPORTANT](#important)
- [1. Build the Docker image and container](#1-build-the-docker-image-and-container)
  - [1.1 Build from Definition Files](#11-build-from-definition-files)
    - [step 1.1 Clone the definition file](#step-11-clone-the-definition-file)
    - [step 1.2 Build the image](#step-12-build-the-image)
    - [step 1.3 Package the image [omit this step if you do not want to share the image]](#step-13-package-the-image-omit-this-step-if-you-do-not-want-to-share-the-image)
  - [1.2 Load image from packaged file](#12-load-image-from-packaged-file)
- [2. Run the vignette dataset](#2-run-the-vignette-dataset)
  - [step 2.1: Download RiboViz](#step-21-download-riboviz)
  - [step 2.2: Create a Container](#step-22-create-a-container)
    - [step 2.2.a: Reuse a container](#step-22a-reuse-a-container)
    - [step 2.2.b: Create a new container](#step-22b-create-a-new-container)
  - [step 2.3: Run the vignette dataset](#step-23-run-the-vignette-dataset)
  - [step 2.4: View results](#step-24-view-results)
- [3. Run a full dataset](#3-run-a-full-dataset)
  - [step 3.1: Download RiboViz and Example-dataset](#step-31-download-riboviz-and-example-dataset)
  - [step 3.2: Create a Container](#step-32-create-a-container)
    - [step 3.2.a: Reuse a container](#step-32a-reuse-a-container)
    - [step 3.2.b: Create a new container](#step-32b-create-a-new-container)
  - [step 3.3: Run the Wallace dataset](#step-33-run-the-wallace-dataset)
  - [step 3.4: Check results](#step-34-check-results)
- [Run interactively](#run-interactively)

# Basic information of Docker and running Docker images

Running the RiboViz with Docker requires some basic knowledge of Docker. In this section, we will first introduce the Docker Image and Docker Container, then we will introduce some instruction to start, run or stop a docker image or container.

## Docker, Docker Image and Docker Container

Docker is a containerize tool that allow us run programs inside a container. It allows you to install softwares, config environments or run and develop programs in a separated environment (container), isolated from the host machine. It also allows you package the program as well as the runtime environment into a image and restore it on another machine easily. With the help of Docker, we can distribute the RiboViz and the runtime environment at the same time, which eliminated the time consuming and complex setting-up-environment procedure.

A Docker Image is a read-only template for deploying containers. It is like the snapshot of container, which stores everything inside that container, like files, settings and environment configurations. It can either be generated from a container by taking a snapshot, or be built by a Dockerfile, which is like a installation script.

A Docker Container is a runtime environment that we run program (like RiboViz) in. It is a instance of a Docker Image, which creates a layer on the read-only image that allows we modify the content. One Docker Image may have more than one containers as its instances, and each container is isolated from other containers.

## Useful Commands to Use Docker

These are some useful commands to run Docker Image and Container.
If you are unsure of some commands or option flags, you can always add `--help` flag to see the manual, or refer to the [documentation](https://docs.docker.com/reference/).

### List all Images/Containers
To list all Images on you computer, you can run `docker image ls`.
To list all live Containers on you computer, you can run `docker container ls` or `docker ps`.
To list all live and stopped Containers on you computer, you can run `docker container ls -a` or `docker ps -a`.

### Export Image to File
To export a Image to a file on your disk, you can run `docker save <image name> | gzip > <file name>.tgz`.
This will create a file `<file name>.tgz` that stores the `<image name>` image, and then you can distribute the file through internet and restore it on other computers.

### Restore an Image from File
To restore an Image from file, you can run `gunzip -c riboviz_docker.tgz | docker load`.
This will load the file and turn it back to an image.

### Build an Image from File
Besides restoring an Image from file, you can also build an Image from Dockerfile.
You can run `docker build -t <image name> <path to Dockerfile>`, and the Docker will build the image automatically.

### Run commands in an Image (by creating a container)
To create a container from a image and run some commands, you can run `docker run <image name> <commands>`.
You can also specify the container name by `--name` flag, or run interactively by `-it`.

A `run` command will create a container from the image, run the container, and stop the container after the execution. As a result, you may need to start the container again before executing another command. To keep alive a container, started using `run`, you can run `docker run <image name> -itd /bin/bash`.

### Run commands in a Container
To run commands in a container, you can run `docker exec <container name> <command>`.
The `<container name>` can be replaced by `<container hash>`, if you didn't specify the container name when you start the container.

An `exec` command will run a command in a running container, but will not stop the container after execution. So you will need to start a stopped container before executing the command.

### Start or Stop a Container
To start or stop a container, you can run `docker start|stop <container name>|<container hash>`.
More information can be found at:
* [docker exec](https://docs.docker.com/engine/reference/commandline/exec/).
* [docker start](https://docs.docker.com/engine/reference/commandline/start/).
* [docker stop](https://docs.docker.com/engine/reference/commandline/stop/).


# 0. Install Docker on your PC
Docker is not available on Eddie and most other HPC systems, it is also very difficult to install docker under a non-privilege user.
Installing Docker on your PC is well documented by the official documentation.
Please refer to the official installing guide. [https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)


### IMPORTANT
* The docker image ideally should be working properly on Windows, Linux and MacOS, but in this RiboViz runtime image, only Linux (Host OS: Ubuntu 18.04) had been tested yet.

* Please refer to the official documentation on how to run Docker with non-privilege user and the workaround of problems.

# 1. Build the Docker image and container
The first step of using the RiboViz docker image will be building the image.
## 1.1 Build from Definition Files
Docker uses Dockerfile to control the building stages of a image.
The building of the image is like installing RiboViz on a brand new Linux system, but all installation steps, system configurations and console commands are configured and executed automatically by the install script (the definition files), so the user will not need to interact with the system durning installation. The build time of the image may be longer than **1 hour**.

There are three steps to build the image from definition files.
### step 1.1 Clone the definition file
A example definition file can be found at `container/docker/Dockerfile`.
You can either [download the file](../container/docker/Dockerfile) directly, or clone the whole repository and then `cd` to the target folder.

If you are familiar with docker and RiboViz, and you want to modify the image to satisfy your special need (for example, install some extra software), you may modify the `RUN` script of the Dockerfile.

### step 1.2 Build the image
To build the image, you need to run
```console
$ docker build -t riboviz/riboviz_docker:v1 .
```
The script will do all building steps for you, and generate an image `riboviz/riboviz_docker` with version tag `v1`.
The `-t` flag allows you name the docker image and the version number. If you do not specify it, a random string (the image's hash) will be assigned to the image, which is difficult to remember.

It will likely take more than **1 hour** to build the image.

After the build script finished, you will be able to see your image (`riboviz/riboviz_docker:v1`) and its properties by running
```console
$ docker images
```

### step 1.3 Package the image [omit this step if you do not want to share the image]
The built image is stored inside the docker, it need to be exported before sharing.
Run 
```console
$ docker save riboviz/riboviz_docker:v1 | gzip > riboviz_docker.tgz
```
will generate a compressed file, `riboviz_docker.tgz`, which is the exported image.

## 1.2 Load image from packaged file
If you had already built the image on another machine, and had exported it to a compressed file, you can load it back by running
```console
$ gunzip -c riboviz_docker.tgz | docker load
```
You can check if the image is loaded successfully by running
```console
$ docker images
```

# 2. Run the vignette dataset

To run the vignette dataset, you can follow the following 4 steps.

## step 2.1: Download RiboViz

Though the docker image contains the runtime environment of RiboViz, it doesn't contain the RiboViz itself.
The reason is that the RiboViz is under development and updates are made frequently, especially the develop branch. However, the runtime environment will not change for a long time.
We think that it is unnecessary to rebuild the image every time when the RiboViz updates, we only want to rebuild the image when the runtime environment need to be updated.

To download RiboViz, you need to clone the `https://github.com/riboviz/riboviz.git` repository.
You can run the following command, and replace `/your/path` with the path that you wish RiboViz repository to be stored.
```console
$ cd /your/path/
$ git clone https://github.com/riboviz/riboviz.git
$ cd riboviz
$ git checkout develop
```
## step 2.2: Create a Container

To run RiboViz, you will need a live container. There are two possible situation.

### step 2.2.a: Reuse a container
If you had already created a container before, you can reuse that container.
You can see all containers (include stopped) by running:
```console
$ docker container ls -a
```
If the container is stopped, you can run:
```console
$ docker start <container name>
```

### step 2.2.b: Create a new container
If you didn't created a container before, or you want to create a new container, you can run:
```console
$ docker run -v /your/path/riboviz:/riboviz --name riboviz1 -dit riboviz/riboviz_docker:v1 /bin/bash
```
This will create a new container with name `riboviz1`, and bind the riboviz folder into the container.
The path `/your/path` is the path where RiboViz is stored in step 2.1.

Please notice that Docker does **NOT** allow user to bind new folder to a existing container, so you need to create a new container if you want to bind new folders.

## step 2.3: Run the vignette dataset
To run the RiboViz with the vignette dataset, you can run:
```console
$ docker exec riboviz1 /bin/bash -c "cd /env ;source setenv.sh ;cd /riboviz; nextflow run prep_riboviz.nf -params-file vignette/vignette_config.yaml -ansi-log false
```
This command will first activate the environment, then run the workflow.

## step 2.4: View results
Due to the privilege reason of Docker, the output files will be owned by `root`.
To solve this, you will need to first get your `UID` and `GID`.
You can run
```console
$ id
```
And the output will be like
```console
uid=<uid>(<your username>) gid=<gid>(<your username>) groups= ....
```
Then you can execute:
```console
$ docker exec riboviz1 /bin/bash -c "chown -R <uid>:<gid> /riboviz"
```
The result will be available in `/your/path/riboviz/` on the host.
For example, in the `/your/path/riboviz/vignette/output` folder, there will be the following two files and two folders:
```console
$ ls /your/path/riboviz/vignette/output
read_counts.tsv  TPMs_collated.tsv  WT3AT  WTnone
```

# 3. Run a full dataset
To run the full dataset, you can follow the following 4 steps. Please notice that the container created for running vignette dataset may not be reused, because there are some difference in preparing the dataset.

## step 3.1: Download RiboViz and Example-dataset

Like the step in running a vignette dataset, we need to first download the RiboViz repository. However, to run the full dataset, you will also need to download the example-dataset and the fastq files of the dataset.

To download RiboViz, you need to clone the `https://github.com/riboviz/riboviz.git` repository.
You can run the following command, and replace `/your/path` with the path that you wish RiboViz repository to be stored.
```console
$ cd /your/path/
$ mkdir riboviz
$ cd riboviz
$ git clone https://github.com/riboviz/riboviz.git
$ cd riboviz
$ git checkout develop
```

To download the example-dataset, you need to clone the `https://github.com/riboviz/example-datasets.git` repository.
You can run
```console
$ cd /your/path/riboviz
$ git clone https://github.com/riboviz/example-datasets.git
```

To prepare the dataset, you need to download and install `sratoolkit` to generate fastq files. The documentation on that software can be found at [https://hpc.nih.gov/apps/sratoolkit.html](https://hpc.nih.gov/apps/sratoolkit.html). 
**If you don't have sratoolkit installed on you PC, it is recommended to generate the fastq files on Eddie, then download them to your PC**
Take the Wallace dataset as an example, you should run the following command **on Eddie** to generate the fastq files:
```console
$ cd /exports/eddie/scratch/$USER/
$ mkdir Wallace_2020_JEC21
$ mkdir Wallace_2020_JEC21/input

$ module load igmm/apps/sratoolkit/2.10.8
$ cd Wallace_2020_JEC21/input
$ prefetch SRR9620588 SRR9620586
$ fasterq-dump SRR9620588
$ fasterq-dump SRR9620586

$ module load igmm/apps/pigz
$ pigz *.fastq
```
Then you can download the `/exports/eddie/scratch/$USER/Wallace_2020_JEC21` dir from Eddie to your computer, and put it into the `/your/path/riboviz/riboviz` folder. This can be done by using the drag-drop function of MobaXTerm, or the `scp` command.

Finally, you should prepare the configuration files for RiboViz.
You can run 
```console
$ cd /your/path/riboviz/riboviz/Wallace_2020_JEC21/
$ cp ../../example-datasets/fungi/cryptococcus/Wallace_2020_JEC21_2-samples_10p_up12dwn9_CDS_120bpL_120bpR_config.yaml Wallace_2020_JEC21_2-samples_10p_up12dwn9_CDS_120bpL_120bpR_config.yaml
```

After the preparation, the folder should have the following structure
``` text
/your/path/riboviz
              |------example-dataset/
              | 
              ----riboviz/
                     |----- every thing in riboviz repository
                     ------Wallace_2020_JEC21/
                             |---- Wallace_2020_JEC21_2-samples_10p_up12dwn9_CDS_120bpL_120bpR_config.yaml
                             -----input/
                                    |-----SRR9620588.fastq.gz
                                    ------SRR9620586.fastq.gz
```
## step 3.2: Create a Container

### step 3.2.a: Reuse a container
If you had already created a container before, you can reuse that container. Pleach check if the folder is binded to the container correctly.
You can see all containers (include stopped) by running:
```console
$ docker container ls -a
```
If the container is stopped, you can run:
```console
$ docker start <container name>
```

### step 3.2.b: Create a new container
If you didn't created a container before, or you want to create a new container, you can run:
```console
$ docker run -v /your/path/riboviz:/riboviz --name riboviz2 -dit riboviz/riboviz_docker:v1 /bin/bash
```
This will create a new container with name `riboviz2`.
The path `/your/path` is the path where RiboViz is stored in step 3.1.

Please notice that Docker does **NOT** allow user to bind new folder to a existing container, so you need to create a new container if you want to bind new folders.

## step 3.3: Run the Wallace dataset
To run the RiboViz with the vignette dataset, you can run:
```console
$ docker exec riboviz2 /bin/bash -c "cd /env ;source setenv.sh ;cd /riboviz/riboviz; nextflow run prep_riboviz.nf -params-file Wallace_2020_JEC21/Wallace_2020_JEC21_2-samples_10p_up12dwn9_CDS_120bpL_120bpR_config.yaml -work-dir Wallace_2020_JEC21/work -ansi-log false
```
This command will first activate the environment, then run the workflow.

## step 3.4: Check results
Due to the privilege reason of Docker, the output files will be owned by `root`.
To solve this, you will need to first get your `UID` and `GID`.
You can run
```console
$ id
```
And the output will be like
```console
$ uid=<uid>(<your username>) gid=<gid>(<your username>) groups= ....
```
Then you can execute:
```console
$ docker exec riboviz2 /bin/bash -c "chown -R <uid>:<gid> /riboviz"
```
The result will be available in `/your/path/riboviz/riboviz/Wallace_2020_JEC21/` on the host.

# Run interactively
The docker allows interactive sessions  connected to the container, and you can use the container as a new OS with the session.
This is helpful for debugging and customizing the container.
To start the interactive session, you can run
```console
$ docker exec -it <container name> /bin/bash
```