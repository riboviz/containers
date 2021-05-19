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
  - [1.2 Download a prebuilt image.](#12-download-a-prebuilt-image)
  - [1.3 Structure of image](#13-structure-of-image)
- [2. Run the Image](#2-run-the-image)
  - [2.1 load Singularity](#21-load-singularity)
    - [2.2 Running interactively (optional)](#22-running-interactively-optional)
    - [2.3 Prepare the dataset and RiboViz](#23-prepare-the-dataset-and-riboviz)
    - [2.4 Set cache dir](#24-set-cache-dir)
    - [2.5 Run vignette dataset](#25-run-vignette-dataset)
    - [2.6 Run a full size dataset](#26-run-a-full-size-dataset)
      - [2.7 Run through job](#27-run-through-job)
- [3. Troubleshooting](#3-troubleshooting)
  - [3.1 Missing folder (Unable to mount folder)(very unusual)](#31-missing-folder-unable-to-mount-foldervery-unusual)
  - [3.2 Script Halt Half Way](#32-script-halt-half-way)
  - [3.3 Python is not found](#33-python-is-not-found)

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
The building of the image is like installing RiboViz on a brand new Linux system, but all installation steps, system configurations and console commands are configured and executed automatically by the install script (the definition files), so the user will not need to interact with the system durning installation. The build time of the image may be longer than **1-2 hour**.

There are three steps to build the image from definition files.
### step 1. Clone the definition file
A example definition file can be found at `container/singularity/riboviz_singularity.def`.
You can either download the file directly, or clone the whole repository and then `cd` to the target folder.

You can also modify the build script to configure your own environment.

### step 2. Build the script
You will only need to run `singularity build --fakeroot riboviz_singularity.sif riboviz_singularity.def` and the script will do all building steps for you.
The output will be an image file, `riboviz_singularity.sif`, in the current folder. You can also change the name of output folder by modifying the parameters in the command, type `singularity --help` for more information.

It will likely take more than **1-2 hour** to build the image.

## 1.2 Download a prebuilt image.
You can also download a prebuilt image if the building stage is too time consuming for you.
You can download a prebuilt `riboviz_singularity.sif` at [https://img.canuse.xyz/riboviz_singularity.sif](https://img.canuse.xyz/riboviz_singularity.sif), which is built by the steps above.
It should be able to execute on Eddie and any other computers or VMs with x86-64 structure, but it may not be able to execute on ARM or other platforms.

## 1.3 Structure of image
The structure of the image is a standard linux system like structure. The folders that contain dataset or the RiboViz program are stored in the host, and mapped into the container when execution. The runtime environment is installed inside the container, which can be activated by `source /env/setenv.sh` inside the container.


# 2. Run the Image

Running the image on Eddie or your VM is similar.

## 2.1 load Singularity

If you want to run on Eddie, you should load singularity first by `module load singularity`. If you wang to run on you own VM, you need to make sure that singularity is properly installed.

### 2.2 Running interactively (optional)
The image can be executed on eddie both interactively or through command line.
To run the image interactively, you need to first request a node with `qlogin`.
For example, command `qlogin -pe interactivemem 4 -l h_vmem=6G` will request for a parallel environment with 4 cores and 4*6=24G memory.
Then, you can simply run `singularity shell -B<path/in/host>:<path/in/container> riboviz_singularity.sif`. A shell session will be provided and you can run commands as if you are using a VM that have every thing installed. The path bind `-B<>:<>` will be introduced later.

### 2.3 Prepare the dataset and RiboViz
The dataset and RiboViz is stored on the host machine, so you may reuse the existing files.
If you didn't have the dataset and RiboViz software, you can refer to `riboviz/docs/user/run-on-eddie.md`.
You can follow the same instruction in that documentation to prepare the dataset.

After this step, you should remember two paths.

* `/path/to/dataset` : The path to your dataset, for example, `/exports/eddie/scratch/xxxxxxx/Wallace`. If you are running a vignette dataset, this can be omitted.
* `/path/to/riboviz` : The path to your RiboViz (and example-datasets) folder. Inside the folder, there should be two folders, one is `riboviz`, which is the RiboViz in develop branch. The other is `example-datasets`, which is example-datasets in master branch. It has the same structure with the `riboviz/docs/user/run-on-eddie.md`.

Alternatively, you may want to put the example-datasets folder into dataset folder. This is OK, but you need to manually check the path in the yaml file to make sure the riboviz can find the files.

### 2.4 Set cache dir
You should set the cache dir of singularity before running singularity container, otherwise, it may fill you home dir or cause other problems.
You should run 
```bash
export SINGULARITY_TMPDIR=$TMPDIR
export SINGULARITY_CACHEDIR=/exports/eddie/scratch/<your id>/cache
```

### 2.5 Run vignette dataset
To run the vignette dataset, you can simply run
```bash
singularity -vvv run -B /local:/local -B /path/to/riboviz:/riboviz riboviz_singularity.sif /bin/bash -c "cd /env;source setenv.sh;cd /riboviz/riboviz/; nextflow run prep_riboviz.nf -params-file vignette/vignette_config.yaml -ansi-log false"
```
The result can be visited in the `/path/to/riboviz` folder on host.

The command can be break into three parts. The first part, `singularity -vvv run` will start the image with a verbose log. The second part, `/bin/bash -c` will let singularity run `bash` after it starts the image. The third part, `"cd /env ;source setenv.sh ;cd riboviz/riboviz; nextflow run prep_riboviz.nf -params-file vignette/vignette_config.yaml -ansi-log false"` contains the command to start nextflow. It will first active the environment, then run the nextflow workflow.

### 2.6 Run a full size dataset
To run a full size dataset, you need to first create symbol link inside the riboviz folder.
```bash
cd /path/to/riboviz/riboviz/
ln -s /Wallace_2020_JEC21
```
Please notice that the folder `/Wallace_2020_JEC21` may not exist on the host machine, but you can always be able to create the symbol link on linux, and the container will map this folder automatically.

Then you should run the following command. The `Wallace_2020_JEC21` can change to any dataset, but you need to manually check the file path.
```bash
singularity -vvv run -B /local:/local -B /path/to/riboviz:/riboviz -B /path/to/dataset:/Wallace_2020_JEC21 riboviz_singularity.sif /bin/bash -c "cd /env;source setenv.sh;cd /riboviz/riboviz/; nextflow run prep_riboviz.nf -params-file Wallace_2020_JEC21/Wallace_2020_JEC21_2-samples_10p_up12dwn9_CDS_120bpL_120bpR_config.yaml -work-dir Wallace_2020_JEC21/work -ansi-log false"
```
The result can be visited in the `/path/to/dataset` folder on host.

#### 2.7 Run through job
The example submit job file of vignette and full-size dataset can be found at `examples/vignette.job` and `examples/Wallace_2020_JEC21.job`. Please replace `<your id>` with your login id in the script before submit.


# 3. Troubleshooting
## 3.1 Missing folder (Unable to mount folder)(very unusual)
* **What happened**: In some occasion, we may see that the singularity gives warning that some folders are not mounted. This will cause the workflow halt in `trim5pMismatches`, `generateStatFlag` or other procedure.

* **When**: This is likely to happen when you reuse a chroot folder for more than once.
* **Why**: The reason of this may be that Eddie and Singularity will flush every thing in `/tmp`, `/var`, `/proc`, `/sys` and other folders after a job is finished to allow next job run on this node. However, these folders are bind to the folders inside the container, which will cause these folders inside the image be flushed.
* **Solution**: 
  Try with `pid` or `containall` flag when running.
  
## 3.2 Script Halt Half Way
* **What happened**: The workflow halts half way, and there are no error reported in the .e or .o output file. Run `qacct -j <job id>` and reported the failure reason is `execd enforced h_vmem limit`.
* **When**: Running a big dataset with not enough memory requested.
* **Why**: The SGE will stop the job if it reaches the memory limit requested. Though job with a small memory request is likely to be scheduled faster, it is too risky. 
* **Solution** Increase the memory requested in job submission file. It is recommended that you shoule request at least 8GB memory per process. However, this may cause the job scheduled slower because many node have only 32GB, 64GB or 128GB memory.(which means that if you request more than 4,8 or 16 cores, the job will never be scheduled to these nodes.)

## 3.3 Python is not found
In some occasion, the environment inside the container will likely to use python2 instead of python3. When this happen, you will need to modify the workflow file `prep_riboviz.nf`, replace all `python` to `python3` in that file.
It is recommended to modify this even this not happened.