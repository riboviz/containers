<h1>Run RiboViz with Docker</h1>

This documentation shows how the user can run RiboViz with Docker.
It is recommended to run containerized RiboViz with Singularity, another containerize tool targets on HPC systems.
However, we still provide a docker version of RiboViz, for the Docker has a very large community and detailed documentations.

- [0. Install Docker on your PC](#0-install-docker-on-your-pc)
      - [IMPORTANT](#important)
- [1. Build the Docker image and container](#1-build-the-docker-image-and-container)
  - [1.1 Build from Definition Files](#11-build-from-definition-files)
    - [step 1. Clone the definition file](#step-1-clone-the-definition-file)
    - [step 2. Build the script](#step-2-build-the-script)
    - [step 3. Package the image [omit this step if you do not want to share the image]](#step-3-package-the-image-omit-this-step-if-you-do-not-want-to-share-the-image)
      - [Important](#important-1)
  - [1.2 Load image from packaged file](#12-load-image-from-packaged-file)
  - [1.3 Create a container from image](#13-create-a-container-from-image)
      - [IMPORTANT](#important-2)
- [2. Run the vignette dataset](#2-run-the-vignette-dataset)
      - [TROUBLESHOOTING](#troubleshooting)
- [3. Run a full dataset](#3-run-a-full-dataset)
      - [IMPORTANT](#important-3)

# 0. Install Docker on your PC
Docker is not available on Eddie and most other HPC systems, it is also very difficult to install docker under a non-privilege user.
Installing Docker on your PC is simple and well documented by the official documentation.
Please refer to the official installing guide.
[https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)


#### IMPORTANT
* The docker ideally should be installed easily and work properly on Windows, Linux and MacOS, but in this RiboViz container, only Linux (Host OS: Ubuntu 18.04) had been tested yet.

* Please refer to the official documentation on how to run Docker with non-privilege user and the workaround of problems.

# 1. Build the Docker image and container
Similar to Singularity, the Docker image can be build from dockerfile or downloaded.
## 1.1 Build from Definition Files
Docker uses dockerfile to control the building stages of a image.
The building of the image is like installing RiboViz on a brand new Linux system, but all installation steps, system configurations and console commands are configured and executed automatically by the install script (the definition files), so the user will not need to interact with the system durning installation. The build time of the image may be longer than **1 hour**.

There are three steps to build the image from definition files.
### step 1. Clone the definition file
A example definition file can be found at `container/docker/Dockerfile`.
You can either download the file directly, or clone the whole repository and then `cd` to the target folder.

You can also modify the build script to configure your own environment.

### step 2. Build the script
You will only need to run `docker build -t riboviz/riboviz_docker:v1 --rm .` and the script will do all building steps for you.
The `-t` flag allows you name the docker image and the version number. If you do not specify it, a random string will be assigned to the image, which is difficult to read.

It will likely take more than **1 hour** to build the image.

### step 3. Package the image [omit this step if you do not want to share the image]
The built image is stored inside the docker, it need to be exported before sharing.
Run `docker save riboviz/riboviz_docker | gzip > riboviz_docker.tgz` will generate a compressed file, `riboviz_singularity.tgz`, which is the exported image.

#### Important
If you do not want to compress the image, run `docker save riboviz/riboviz_docker > riboviz_docker.tar`. Do not omit the file path and `>` because the command will write everything to stdout.

## 1.2 Load image from packaged file
You can get a pre-built image from [https://img.canuse.xyz/riboviz_docker.tgz](https://img.canuse.xyz/riboviz_docker.tgz). Of course, you can also get the image exported by other's.
To load the image back to docker, you need to run `gunzip -c riboviz_docker.tgz | docker load`. If the docker image is not compressed, you can simply run `docker load < riboviz_docker.tar`.

## 1.3 Create a container from image
The docker has two concepts, image and container. Image is like a snapshot and is readonly. Container is an instance of image. One image can have many independent containers, each container may modify some files inside itself but not affect the files in the image. The image can be updated by committing from a container.

To run the code, we need to first create a container from the image by running `docker run --name riboviz1 -itd riboviz/riboviz_docker:v1 /bin/bash`. This command will start the container from image `riboviz_docker` at background and give the container a name `riboviz1`.

Alternatively, if you want to reuse a existing container, you can simply run `docker run -itd <container_name> /bin/bash`.

#### IMPORTANT
The `run` and `exec` are different in docker. A `run` command will run the image, generate a container and stop the container after the execution. A `exec` command will run a started container, but will not stop the container after execution. You can `start` or `stop` a container, when you `start` a container, it will run the last command in `run`. Run the container with `-it` will allow you interact with the container, like `docker run -it <container_name> /bin/bash`. To keep alive a container, you can run `docker run -itd <container_name> /bin/bash`.

# 2. Run the vignette dataset
Running the vignette dataset is very simple.
You can simply run `docker exec <container_name> /bin/bash -c "cd /env ;source setenv.sh ;cd riboviz/riboviz; nextflow run prep_riboviz.nf -params-file vignette/vignette_config.yaml -ansi-log false"`

The output file is stored inside the container, if you want to use the file , you can run `docker cp container_created:path <path>`.

#### TROUBLESHOOTING
In some situation, the conda will likely to use python2 instead of python3 in the docker. When this happen, you will need to modify the workflow file inside the container.
You need to run `docker exec -it <container_name> /bin/bash` to run a interactive shell, then you nee to replace all `python` to `python3` in `/env/riboviz/riboviz/prep_riboviz.nf`.
It is recommended to modify this even this not happened.


# 3. Run a full dataset
To run the full dataset, you can follow the same steps of preparing dataset in singularity, then you need to config the container to bind the dataset.

#### IMPORTANT
**Docker does NOT allow bind new folder to a exist container, so you need to create a new container if you want to bind new folders.**

For example, if the dataset is located at `/path/to/dataset/Wallace_2020_JEC21`, and the container name is `riboviz1`.
Then you can run the following command to bind the dataset.
```bash
docker run -v /path/to/dataset/Wallace_2020_JEC21/:/Wallace_2020_JEC21 --name riboviz2 -it riboviz/riboviz_docker:v1 /bin/bash
cd /env/riboviz/riboviz/
ln -s /Wallace_2020_JEC21
exit
```
and run the following command to execute.
```bash
docker start riboviz2
docker exec riboviz2 /bin/bash -c "cd /env ;source setenv.sh ;cd riboviz/riboviz ; nextflow run prep_riboviz.nf -params-file Wallace_2020_JEC21/Wallace_2020_JEC21_2-samples_10p_up12dwn9_CDS_120bpL_120bpR_config.yaml -work-dir Wallace_2020_JEC21/work -ansi-log false --validate_only"
```

The output will be located in the `/path/to/dataset/Wallace_2020_JEC21` folder, so you can see the result from host machine.
**root privilege may be required to visit the result.**