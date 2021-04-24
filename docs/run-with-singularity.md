<h1>Run RiboViz with Singularity</h1>

This documentation shows how the user can run RiboViz with a containerize tool, Singularity.
By using containerize tools, the user will not have to spend too many time on setting up or restoring the software and runtime environment of RiboViz.

- [0. Install/Load Singularity](#0-installload-singularity)
  - [0.1 Use Singularity on Eddie](#01-use-singularity-on-eddie)
  - [0.2 Install Singularity on Your PC](#02-install-singularity-on-your-pc)
- [1. Build the Image](#1-build-the-image)
  - [1.1 Build from Definition Files](#11-build-from-definition-files)
    - [step 1. Clone the definition file](#step-1-clone-the-definition-file)
    - [step 2. Build the script](#step-2-build-the-script)
    - [step 3. Package the image](#step-3-package-the-image)
  - [1.2 Download a prebuilt image.](#12-download-a-prebuilt-image)
  - [1.3 Structure of image](#13-structure-of-image)
- [2. Run the Image and vignette dataset on Eddie](#2-run-the-image-and-vignette-dataset-on-eddie)
  - [step 2.1 Unpack the image](#step-21-unpack-the-image)
    - [IMPORTANT](#important)
  - [step 2.2 Modify image to fit Eddie](#step-22-modify-image-to-fit-eddie)
  - [step 2.3 Run the Image](#step-23-run-the-image)
    - [2.3.1 Run in Interactive Sessions](#231-run-in-interactive-sessions)
    - [2.3.2 Run with submit script](#232-run-with-submit-script)
- [3. Run the Image and full dataset on Eddie](#3-run-the-image-and-full-dataset-on-eddie)
  - [step 3.1 Unpack the image](#step-31-unpack-the-image)
  - [step 3.2 Prepare the full dataset](#step-32-prepare-the-full-dataset)
  - [step 3.3 Modify image to fit Eddie](#step-33-modify-image-to-fit-eddie)
  - [step 3.4 Create mount folder](#step-34-create-mount-folder)
      - [IMPORTANT](#important-1)
  - [step 3.5 Run the Image](#step-35-run-the-image)
    - [3.5.1 Run in Interactive Sessions](#351-run-in-interactive-sessions)
    - [3.5.2 Run with submit script](#352-run-with-submit-script)
- [4. Run on local PC](#4-run-on-local-pc)
- [5. Debug in Singularity](#5-debug-in-singularity)
  - [5.1 [eddie only] request a login node](#51-eddie-only-request-a-login-node)
  - [5.2 run shell](#52-run-shell)
  - [5.3 load environment](#53-load-environment)
- [6. Troubleshooting](#6-troubleshooting)
  - [6.1 Missing folder (Unable to mount folder)](#61-missing-folder-unable-to-mount-folder)
  - [6.2 Script Halt Half Way](#62-script-halt-half-way)

# 0. Install/Load Singularity

Singularity is a containerize tool that is widely used in HPC systems and softwares.
We thus provide a  RiboViz runtime container based on Singularity.

## 0.1 Use Singularity on Eddie
If you would like to use Singularity on Eddie or any other system managed by *modules environment* like Cirrus, you would only need to run `module load singularity`.

## 0.2 Install Singularity on Your PC
If you would like to install the Singularity program on you own PC or VM, you can refer to the official installation guide of Singularity at [https://sylabs.io/guides/3.7/user-guide/quick_start.html#quick-installation-steps](https://sylabs.io/guides/3.7/user-guide/quick_start.html#quick-installation-steps).

# 1. Build the Image
The image is like a snapshot of the runtime environment, which contains all essential programs and environment settings to run the software.
We can either build the image through definition files or download a prebuilt image.

## 1.1 Build from Definition Files
Singularity uses definition files to control the building stages of a image.
The building of the image is like installing RiboViz on a brand new Linux system, but all installation steps, system configurations and console commands are configured and executed automatically by the install script (the definition files), so the user will not need to interact with the system durning installation. The build time of the image may be longer than **1 hour**.

There are three steps to build the image from definition files.
### step 1. Clone the definition file
A example definition file can be found at `container/singularity/riboviz_singularity.def`.
You can either download the file directly, or clone the whole repository and then `cd` to the target folder.

You can also modify the build script to configure your own environment.

### step 2. Build the script
You will only need to run `singularity build --fakeroot --sandbox riboviz_singularity riboviz_singularity.def` and the script will do all building steps for you.
The output will be a chroot folder (controlled by --sandbox), `riboviz_singularity/`, in the current folder. The reason we choose a chroot folder instead of a image file is that the singularity image file may have write problems. You can also change the name of output folder by modifying the parameters in the command, type `singularity --help` for more information.

It will likely take more than **1 hour** to build the image.

### step 3. Package the image
The image we get now is a chroot folder, which is not easy to share among different machines.
You can package the image by `tar` command, like `tar -czf riboviz_singularity.tgz riboviz_singularity/`. You can also specify other compression parameters or use another compression tool like `7z` or `rar`.
Durning the compression, you may see two warnings that says two files can not be compressed. You can simply neglect that two warnings.

The compressed file, `riboviz_singularity.tgz` is the snapshot of running environment we need.

## 1.2 Download a prebuilt image.
You can also download a prebuilt image if the building stage is too time consuming for you.
You can download a prebuilt `riboviz_singularity.tgz` at [https://img.canuse.xyz/riboviz_singularity.tgz](https://img.canuse.xyz/riboviz_singularity.tgz), which is built by the steps above.
It should be able to execute on Eddie and any other computers or VMs with x86-64 structure, but it may not able to execute on ARM or other platforms.

## 1.3 Structure of image
Inside the chroot folder `riboviz_singularity/`, you will see a standard linux system like structure. The only folder we are interested in is the `env` folder, which contains the runtime environment and the RiboViz software.

The `env/setenv.sh` contains script to set PATH and activate conda environment.
The `env/riboviz` folder contains the RiboViz repository (develop branch) and example dataset repository (master branch).

# 2. Run the Image and vignette dataset on Eddie

The image can be executed on eddie both interactively or through command line.

## step 2.1 Unpack the image
You need to unpack the tar file back to chroot folder first.
If you upload the chroot folder directly or you want to reuse a previous chroot folder, you can skip this step. However, reuse a existing chroot folder may cause some problem on Eddie, please see troubleshooting 1 for more information.

### IMPORTANT
On Eddie, do not unpack the image in your home dir, because it is likely to use all your 10 GB disk quota. You can store the image in you home dir, but copy it into scratch space and unpack it there.

For example, if you are using Eddie, you may run the following commands. Remember to replace `<your id>` with your login id.
```bash
cp riboviz_singularity.tgz /exports/eddie/scratch/<your id>/
cd /exports/eddie/scratch/<your id>/
tar -xzf riboviz_singularity.tgz
```

## step 2.2 Modify image to fit Eddie

We notice that some times the conda environment will use python2 inside the Singularity image, so you need to manually configure the `prep_riboviz.nf` file. 
You can edit the file with `nano`, `vim` or any other editor, search for `python ` and replace it with `python3 `.
The `prep_riboviz.nf` is located at `/exports/eddie/scratch/<your id>/riboviz_singularity/env/riboviz/riboviz/prep_riboviz.nf`

## step 2.3 Run the Image
You can run the image in interactive sessions or through a submit script.
### 2.3.1 Run in Interactive Sessions
To run image on a interactive session of eddie compute node, you need to first request a node with `qlogin`.
For example, command `qlogin -pe interactivemem 4 -l h_vmem=6G` will request for a parallel environment with 4 cores and 4*6=24G memory.

Then you need to go to `/exports/eddie/scratch/<your id>/` by using `cd`, and run `singularity run --writable riboviz_singularity /bin/bash -c "cd /env ;source setenv.sh ;cd riboviz/riboviz; nextflow run prep_riboviz.nf -params-file vignette/vignette_config.yaml -ansi-log false"`. You result will be stored in `riboviz_singularity/env/riboviz/riboviz/vignette`.

The command can be break into three parts. The first part, `singularity run --writable riboviz_singularity` will start the image with a writeable environment. The second part, `/bin/bash -c` will let singularity run `bash` after it starts the image. The third part, `"cd /env ;source setenv.sh ;cd riboviz/riboviz; nextflow run prep_riboviz.nf -params-file vignette/vignette_config.yaml -ansi-log false"` contains the command to start nextflow. It will first active the environment, then run the nextflow workflow.

You may also want to debug with interactive session, this will be introduced later.

### 2.3.2 Run with submit script
You can also run directly with the submit script, an example script can be found at `examples/vignette.job`. Please replace `<your id>` with your login id in the script before submit.

# 3. Run the Image and full dataset on Eddie

## step 3.1 Unpack the image
You should follow the same instruction above (step 2.1) to unpack the image.


## step 3.2 Prepare the full dataset
We will use the same example dataset in `riboviz/docs/user/run-on-eddie.md`.
You can follow the same instruction in that documentation to prepare the dataset.

In short, you can run:
```bash
cd /exports/eddie/scratch/<your id>/
mkdir Wallace_2020_JEC21
mkdir Wallace_2020_JEC21/input
cd Wallace_2020_JEC21
mkdir annotation
cp /exports/eddie/scratch/<your id>/riboviz_singularity/env/riboviz/example-datasets/fungi/cryptococcus/annotation/JEC21_10p_up12dwn9_CDS_with_120bputrs.fa annotation/
cp /exports/eddie/scratch/<your id>/riboviz_singularity/env/riboviz/example-datasets/fungi/cryptococcus/Wallace_2020_JEC21_2-samples_10p_up12dwn9_CDS_120bpL_120bpR_config.yaml Wallace_2020_JEC21/Wallace_2020_JEC21_2-samples_10p_up12dwn9_CDS_120bpL_120bpR_config.yaml

module load igmm/apps/sratoolkit/2.10.8
cd input
prefetch SRR9620588 SRR9620586
fasterq-dump SRR9620588
fasterq-dump SRR9620586
module load igmm/apps/pigz
pigz *.fastq
```
## step 3.3 Modify image to fit Eddie

We notice that some times the conda environment will use python2 inside the Singularity image, so you need to manually configure the `prep_riboviz.nf` file. 
You need to edit the file with `nano`, `vim` or any other editor, search for `python ` and replace it with `python3 `.
The `prep_riboviz.nf` is located at `/exports/eddie/scratch/<your id>/riboviz_singularity/env/riboviz/riboviz/prep_riboviz.nf`

## step 3.4 Create mount folder
The singularity will automatically map the folders in host machine to the folders in the container.
However, it can not auto mount folders that does not exist in the container, so we need to manually create these folder.

To run on eddie, you will need to run the following command to mount the scratch folder.
```bash
cd /exports/eddie/scratch/<your id>/riboviz_singularity/
mkdir exports
cd exports
mkdir eddie
cd eddie
mkdir scratch
cd scratch
mkdir <your id>
```

Then you can create a symbol link in the container to link the full dataset with the container.
```bash
cd /exports/eddie/scratch/<your id>/riboviz_singularity/env/riboviz/riboviz/
ln -s /exports/eddie/scratch/<your id>/Wallace_2020_JEC21
```
#### IMPORTANT
Remember to check troubleshooting 1 when you reuse a container.

## step 3.5 Run the Image
You can run the image in interactive sessions or through a submit script.
### 3.5.1 Run in Interactive Sessions
To run image on a interactive session of eddie compute node, you need to first request a node with `qlogin`.
For example, command `qlogin -pe interactivemem 4 -l h_vmem=8G` will request for a parallel environment with 4 cores and 4*8=32G memory.

Then you need to go to `/exports/eddie/scratch/<your id>/` by using `cd`, and run `singularity run --writable  riboviz_singularity /bin/bash -c "cd /env ;source setenv.sh ;cd riboviz/riboviz ; nextflow run prep_riboviz.nf -params-file Wallace_2020_JEC21/Wallace_2020_JEC21_2-samples_10p_up12dwn9_CDS_120bpL_120bpR_config.yaml -work-dir Wallace_2020_JEC21/work -ansi-log false -with-report"`. You result will be stored in `Wallace_2020_JEC21/` folder.

The command can be break into three parts, which is the same as above.

You may also want to debug with interactive session, this will be introduced later.

### 3.5.2 Run with submit script
You can also run directly with the submit script, an example script can be found at `examples/Wallace_2020_JEC21.job`. Please replace `<your id>` with your login id in the script before submit.

# 4. Run on local PC
Running singularity image on local PC is similar to running on Eddie. The vignette dataset is tested, but running a full dataset on PC is only tested due to limited disk space.

To run the vignette dataset, you can simply run `singularity run --writable riboviz_singularity /bin/bash -c "cd /env ;source setenv.sh ;cd riboviz/riboviz; nextflow run prep_riboviz.nf -params-file vignette/vignette_config.yaml -ansi-log false"`.

To run the full dataset, you can follow the same steps of running it on Eddie. You need to bind the path of dataset first, then run the command.

For example, if the dataset is located at `/path/to/dataset/Wallace_2020_JEC21`, and the singularity image is located at `/path/to/image/riboviz_singularity`.
Then you can run the following command to bind the dataset.
```bash
cd /path/to/image/riboviz_singularity
mkdir Wallace_2020_JEC21
cd env/riboviz/riboviz/
ln -s /Wallace_2020_JEC21
cd /path/to/image/
```
and run the following command to execute.
```bash
cd /path/to/image/riboviz_singularity
singularity run --writable -B /path/to/dataset/Wallace_2020_JEC21:/Wallace_2020_JEC21 riboviz_singularity /bin/bash -c "cd /env ;source setenv.sh ;cd riboviz/riboviz ; nextflow run prep_riboviz.nf -params-file Wallace_2020_JEC21/Wallace_2020_JEC21_2-samples_10p_up12dwn9_CDS_120bpL_120bpR_config.yaml -work-dir Wallace_2020_JEC21/work -ansi-log false --validate_only"
```

# 5. Debug in Singularity
The Singularity supports a interactive session with the container, which might be useful in debugging. Starting a interactive session contains these steps.

## 5.1 [eddie only] request a login node
Similar to above, you may run `qlogin -pe interactivemem 4 -l h_vmem=8G` to get a interactive session.

## 5.2 run shell
To run a shell, you can simply run `singularity shell --writable riboviz_singularity`.

## 5.3 load environment
The environment is located at `/env/setenv.sh` in the container. The riboviz is also located in that folder. You can use the image as if it is a Linux VM.

# 6. Troubleshooting
## 6.1 Missing folder (Unable to mount folder)
* **What happened**: In some occasion, we may see that the singularity gives warning that some folders are not mounted. This will cause the workflow halt in `trim5pMismatches`, `generateStatFlag` or other procedure.

* **When**: This is likely to happen when you reuse a chroot folder for more than once.
* **Why**: The reason of this may be that Eddie and Singularity will flush every thing in `/tmp`, `/var`, `/proc`, `/sys` and other folders after a job is finished to allow next job run on this node. However, these folders are bind to the folders inside the container, which will cause these folders inside the image be flushed.
* **Solution**: There are two possible solutions.
  1. Never reuse a image, unpack the image every time.
  2. [recommend] Manually create these folders by 
  ```bash
  cd /exports/eddie/scratch/<your id>/riboviz_singularity
  mkdir tmp
  mkdir sys
  mkdir proc
  mkdir dev
  mkdir home
  ```
## 6.2 Script Halt Half Way
* **What happened**: The workflow halts half way, and there are no error reported in the .e or .o output file. Run `qacct -j <job id>` and reported the failure reason is `execd enforced h_vmem limit`.
* **When**: Running a big dataset with not enough memory requested.
* **Why**: The SGE will stop the job if it reaches the memory limit requested. Though job with a small memory request is likely to be scheduled faster, it is too risky. 
* **Solution** Increase the memory requested in job submission file. It is recommended that you shoule request at least 8GB memory per core. However, this may cause the job scheduled slower because many node have only 32GB, 64GB or 128GB memory.(which means that if you request more than 4,8 or 16 cores, the job will never be scheduled to these nodes.)