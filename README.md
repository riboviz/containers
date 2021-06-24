# Containerized RiboViz pipeline
This repository contains the build scripts of the Singularity and Docker containers of RiboViz pipeline and their documentation.

To get the latest official release (release 2.1), you can run `git checkout 2.1`.

To get the latest development version, you can run `git checkout develop`.

The sturcture of this repository is:

* The `container` folder contains the Dockerfile to build a docker image and a Definition file to build a singularity image.

* The `doc` folder contains the documentation of how to build the image with the definition files, how to run it on you local PC, how to run it on Eddie and other essential information.

* The `examples` folder contains the example job scripts of running the singularity image on eddie.

It is recommended to run with the Singularity container rather than the Docker container, for Singularity is more widely used on HPC systems.