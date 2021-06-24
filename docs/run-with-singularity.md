<h1>Run RiboViz with Singularity</h1>

This documentation shows how the user can run RiboViz with a containerize tool, singularity.
By using containerize tools, the user will not have to spend too many time on setting up or restoring the software and runtime environment of RiboViz.

- [Basic Information of Singularity](#basic-information-of-singularity)
  - [Singularity definition file, image and sandbox](#singularity-definition-file-image-and-sandbox)
  - [Useful commands to run singularity](#useful-commands-to-run-singularity)
    - [Build image from definition file](#build-image-from-definition-file)
    - [Run commands with image](#run-commands-with-image)
    - [Run image interactively](#run-image-interactively)
- [0. Install/Load Singularity](#0-installload-singularity)
  - [0.1 Use Singularity on Eddie](#01-use-singularity-on-eddie)
  - [0.2 Install Singularity on Your PC](#02-install-singularity-on-your-pc)
- [1. Build Image from Definition Files](#1-build-image-from-definition-files)
    - [step 1. Clone the definition file](#step-1-clone-the-definition-file)
    - [step 2. Build the script](#step-2-build-the-script)
  - [1.3 Structure of image](#13-structure-of-image)
- [2. Run the vignette dataset](#2-run-the-vignette-dataset)
  - [step 2.1: Setup Singularity on Eddie (omit if you are running on local machine)](#step-21-setup-singularity-on-eddie-omit-if-you-are-running-on-local-machine)
  - [step 2.2: Prepare the RiboViz](#step-22-prepare-the-riboviz)
    - [step 2.3: Run vignette dataset](#step-23-run-vignette-dataset)
- [3. Run a full dataset](#3-run-a-full-dataset)
  - [step 3.1: Setup Singularity on Eddie (omit if you are running on local machine)](#step-31-setup-singularity-on-eddie-omit-if-you-are-running-on-local-machine)
  - [step 3.2: Prepare the RiboViz](#step-32-prepare-the-riboviz)
    - [step 3.3: Run full dataset](#step-33-run-full-dataset)
- [4. Sample submit script on Eddie](#4-sample-submit-script-on-eddie)
- [5. Run interactively](#5-run-interactively)
- [6. Troubleshooting](#6-troubleshooting)
  - [1. Missing folder (Unable to mount folder)(very unusual)](#1-missing-folder-unable-to-mount-foldervery-unusual)
  - [2. Script Halt Half Way](#2-script-halt-half-way)

# Basic Information of Singularity

Singularity is a containerize tool that is widely used in HPC systems and softwares.
It is supported by Eddie and many other HPC systems, which provides a container to run programs in a separate environment.
Running RiboViz with singularity requires little knowledge about containerize tools, and we will introduce some basic background information of singularity first.

## Singularity definition file, image and sandbox

A definition file is a template to generate images. It contains the scripts and execution order to build a singularity image or sandbox.

A singularity image (.sif file) is a read-only image that contains the runtime environment. It contains every files, environment configurations and system settings that allows user restore the runtime environment easily on another machine.

A singularity sandbox (chroot folder) is a image that is read-only inside image but writable from host. The user can modify the files in the sandbox directly from host, making it easier to copy files between host and sandbox.

## Useful commands to run singularity

These are some basic command to run singularity container. You can also refer to the official guide at [https://sylabs.io/guides/3.7/user-guide/](https://sylabs.io/guides/3.7/user-guide/).

### Build image from definition file
To build singularity image from definition file, you can run `singularity build <image file> <definiation file>`.
The output is a read-only .sif image.

You can add `--sandbox` flag to build a writable sandbox container.

You can add `--fakeroot` to build the image with root user inside, in order to run commands that requires root privilege.

### Run commands with image

You can run `singularity run <image file> <command>` to run commands with the image.

You can add flag `-vvv` to show a verbose running log.

You can add flag `-B <path on host>:<path in image>` to bind folders between host and image. You can add multiple `-B <>:<>` flags to bind more than 1 folders.

You can add flag `--writable` to allow the container writes the image file while running. However, specifying both `-B` and `--writable` flag is considered to be an undefined behavior, that may cause unpredictable results. 

### Run image interactively

You can run `singularity shell <image file>` to run the image interactively.
This will start a shell inside the image, and you can run commands with that shell as if you are using a VM.

The `-vvv`, `-B` and `--writable` flags in `singularity run` are also available in `singularity shell`.

# 0. Install/Load Singularity

To run RiboViz with singularity, you will need to install singularity first.
## 0.1 Use Singularity on Eddie
If you would like to use singularity on Eddie or any other system managed by *modules environment* like Cirrus, you can run 
```console
module load singularity
```

## 0.2 Install Singularity on Your PC
If you would like to install the singularity program on you own PC or VM, you can refer to the official installation guide of singularity at [https://sylabs.io/guides/3.7/user-guide/quick_start.html#quick-installation-steps](https://sylabs.io/guides/3.7/user-guide/quick_start.html#quick-installation-steps).

# 1. Build Image from Definition Files
Singularity uses definition files to control the building stages of a image.
The building of the image is like installing RiboViz on a brand new Linux system, but all installation steps, system configurations and console commands are configured and executed automatically by the install script (the definition files), so the user will not need to interact with the system durning installation. The build time of the image may be longer than **1-2 hour**.

There are two steps to build the image from definition files.
### step 1. Clone the definition file
A example definition file can be found at `container/singularity/riboviz_singularity.def`.
You can either [download the file](../container/singularity/riboviz_singularity.def) directly, or clone the whole repository and then `cd` to the target folder.

If you are familiar with singularity and RiboViz, and you want to modify the image to satisfy your special need (for example, install some extra software), you may modify the `%post` script of the definition file.

### step 2. Build the script
To build the script, you can run
```console
$ singularity build --fakeroot riboviz_singularity.sif riboviz_singularity.def
```
The output will be an image file in the current folder.
```console
$ ls
riboviz_singularity.sif riboviz_singularity.def
```
You can also change the name of output folder by modifying the parameters in the command, type `singularity --help` for more information.

It will likely take more than **1-2 hour** to build the image.

## 1.3 Structure of image
The structure of the image is a standard linux system like structure. The folders that contain dataset or the RiboViz program are stored in the host, and mapped into the container when execution. The runtime environment is installed inside the container, which can be activated by `source /env/setenv.sh` inside the container.


# 2. Run the vignette dataset

To run the RiboViz with singularity with the vignette dataset, you can follow the following 3 steps.

## step 2.1: Setup Singularity on Eddie (omit if you are running on local machine)

To run singularity on Eddie, you need load the module and set the cache dir first. You can run:
```console
$ module load singularity
$ export SINGULARITY_TMPDIR=$TMPDIR
$ export SINGULARITY_CACHEDIR=/exports/eddie/scratch/$USER/cache
```

## step 2.2: Prepare the RiboViz
The RiboViz is stored on the host machine, so you may reuse the existing files.
You need to setup the following path:
* `/path/to/riboviz` : The path to your RiboViz folder, which is the `https://github.com/riboviz/riboviz.git` repository in main branch

An example to prepare RiboViz on Eddie and in `$HOME` dir is:
```console
$ cd ~
$ git clone https://github.com/riboviz/riboviz.git
$ cd riboviz
```
In this situation, the `/path/to/riboviz` should be `$HOME/riboviz`.

### step 2.3: Run vignette dataset
To run the vignette dataset, you can run the following command:
```console
singularity -vvv run -B /local:/local -B /path/to/riboviz:/riboviz <image file> /bin/bash -c "cd /env;source setenv.sh;cd /riboviz; nextflow run prep_riboviz.nf -params-file vignette/vignette_config.yaml -ansi-log false"
```
**If you are running on a local machine, you can omit `-B /local:/local`.**
The result can be visited in the `/path/to/riboviz/vignette` folder on host.

Continue with the example in step 2.2, you can run
```console
$ cd ~
$ ls
riboviz riboviz_singularity.sif
$ singularity -vvv run -B /local:/local -B $HOME/riboviz:/riboviz riboviz_singularity.sif /bin/bash -c "cd /env;source setenv.sh;cd /riboviz; nextflow run prep_riboviz.nf -params-file vignette/vignette_config.yaml -ansi-log false"
```
And you should be able to check the result by running:
```console
$ ls $HOME/riboviz/vignette/output
read_counts.tsv  TPMs_collated.tsv  WT3AT  WTnone
```

# 3. Run a full dataset
Running a full size dataset is similar to the vignette dataset, but you need to do some extra step in preparation. You can follow the following 3 steps.

## step 3.1: Setup Singularity on Eddie (omit if you are running on local machine)

To run singularity on Eddie, you need load the module and set the cache dir first. You can run:
```console
$ module load singularity
$ export SINGULARITY_TMPDIR=$TMPDIR
$ export SINGULARITY_CACHEDIR=/exports/eddie/scratch/<your id>/cache
```

## step 3.2: Prepare the RiboViz
The RiboViz is stored on the host machine, so you may reuse the existing files.
You need to setup the following path:
* `/path/to/riboviz` : The path to your RiboViz (and example-datasets) folder. Inside the folder, there should be two folders, one is `riboviz`, which is the RiboViz in main branch. The other is `example-datasets`, which is example-datasets in master branch. In the `riboviz` folder, there should be a folder containing the dataset (for example, Wallace_2020_JEC21).

To prepare the dataset, you need to download and install `sratoolkit` to generate fastq files. The documentation on that software can be found at [https://hpc.nih.gov/apps/sratoolkit.html](https://hpc.nih.gov/apps/sratoolkit.html). 
**If you don't have sratoolkit installed on you PC, you can generate the files and folders on Eddie, then download them to your PC**

``` text
/path/to/riboviz
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

An example to prepare RiboViz on Eddie and in scratch dir is:
```console
$ cd /exports/eddie/scratch/$USER
$ mkdir riboviz
$ cd riboviz
$ git clone https://github.com/riboviz/riboviz.git
$ cd riboviz
$ ls
$ cd ..
$ git clone https://github.com/riboviz/example-datasets.git
$ cd riboviz
$ mkdir Wallace_2020_JEC21
$ mkdir Wallace_2020_JEC21/input

$ module load igmm/apps/sratoolkit/2.10.8
$ cd Wallace_2020_JEC21/input
$ prefetch SRR9620588 SRR9620586
$ fasterq-dump SRR9620588
$ fasterq-dump SRR9620586

$ module load igmm/apps/pigz
$ pigz *.fastq
$ cd ..
$ cp ../../example-datasets/fungi/cryptococcus/Wallace_2020_JEC21_2-samples_10p_up12dwn9_CDS_120bpL_120bpR_config.yaml Wallace_2020_JEC21_2-samples_10p_up12dwn9_CDS_120bpL_120bpR_config.yaml
```
In this situation, the `/path/to/riboviz` should be `/exports/eddie/scratch/$USER/riboviz`.


### step 3.3: Run full dataset
To run the full dataset, you can run the following command:
```console
singularity -vvv run -B /local:/local -B /path/to/riboviz:/riboviz <image file> /bin/bash -c "cd /env;source setenv.sh;cd /riboviz/riboviz; nextflow run prep_riboviz.nf -params-file Wallace_2020_JEC21/Wallace_2020_JEC21_2-samples_10p_up12dwn9_CDS_120bpL_120bpR_config.yaml -work-dir Wallace_2020_JEC21/work -ansi-log false"
```
**If you are running on a local machine, you should omit `-B /local:/local`.**
The result can be visited in the `/path/to/riboviz` folder on host.

Continue with the example in step 3.2, you can run
```console
$ cd ~
$ ls
riboviz riboviz_singularity.sif
$ singularity -vvv run -B /local:/local -B /exports/eddie/scratch/$USER/riboviz:/riboviz riboviz_singularity.sif /bin/bash -c "cd /env;source setenv.sh;cd /riboviz/riboviz; nextflow run prep_riboviz.nf -params-file Wallace_2020_JEC21/Wallace_2020_JEC21_2-samples_10p_up12dwn9_CDS_120bpL_120bpR_config.yaml -work-dir Wallace_2020_JEC21/work -ansi-log false"
```
And you should be able to check the result by running:
```console
$ ls exports/eddie/scratch/$USER/riboviz/riboviz/Wallace_2020_JEC21
input index output tmp work Wallace_2020_JEC21_2-samples_10p_up12dwn9_CDS_120bpL_120bpR_config.yaml
```


# 4. Sample submit script on Eddie
The example submit job file of vignette and full-size dataset can be found at `examples/vignette.job` and `examples/Wallace_2020_JEC21.job`. Please replace `<your id>` with your login id in the script before submit.

# 5. Run interactively
The singularity allows interactive sessions connected to the image, and you can use the image as a new OS with the session.
This is helpful for debugging and customizing the container.
To start the interactive session, you can run
```console
$ singularity shell <container name>
```

# 6. Troubleshooting
## 1. Missing folder (Unable to mount folder)(very unusual)
* **What happened**: In some occasion, we may see that the singularity gives warning that some folders are not mounted. This will cause the workflow halt in `trim5pMismatches`, `generateStatFlag` or other procedure.

* **When**: This is likely to happen when you reuse a chroot folder for more than once.
* **Why**: The reason of this may be that Eddie and Singularity will flush every thing in `/tmp`, `/var`, `/proc`, `/sys` and other folders after a job is finished to allow next job run on this node. However, these folders are bind to the folders inside the container, which will cause these folders inside the image be flushed.
* **Solution**: 
  Try with `pid` or `containall` flag when running.
  
## 2. Script Halt Half Way
* **What happened**: The workflow halts half way, and there are no error reported in the .e or .o output file. Run `qacct -j <job id>` and reported the failure reason is `execd enforced h_vmem limit`.
* **When**: Running a big dataset with not enough memory requested.
* **Why**: The SGE will stop the job if it reaches the memory limit requested. Though job with a small memory request is likely to be scheduled faster, it is too risky. 
* **Solution** Increase the memory requested in job submission file. It is recommended that you shoule request at least 8GB memory per process. However, this may cause the job scheduled slower because many node have only 32GB, 64GB or 128GB memory.(which means that if you request more than 4,8 or 16 cores, the job will never be scheduled to these nodes.)
