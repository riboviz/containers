FROM ubuntu:18.04 as build
MAINTAINER Junchen Liu
WORKDIR /root/
SHELL ["/bin/bash", "-c"]
ENV DEBIAN_FRONTEND="noninteractive"
ENV TZ="Europe/London"
RUN apt-get update && \
    apt-get install -y git curl bedtools hdf5-tools pigz nano unzip wget gnupg && \
    echo 'deb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran35/'>>/etc/apt/sources.list && \
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9 && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
    apt-get update && apt-get install -y r-base r-base-dev libxml2-dev libssl-dev libcurl4-openssl-dev && \
    R -e "install.packages('testthat'); install.packages('RcppRoll'); install.packages('optparse'); install.packages('tidyr'); install.packages('ggplot2'); install.packages('shiny'); install.packages('plotly'); install.packages('readr'); install.packages('git2r'); install.packages('git2r'); install.packages('here'); install.packages('BiocManager'); BiocManager::install('Rsamtools'); BiocManager::install('rtracklayer');  BiocManager::install('rhdf5'); BiocManager::install('Biostrings'); BiocManager::install('ShortRead')" 

FROM ubuntu:18.04
MAINTAINER Junchen Liu
WORKDIR /root/
SHELL ["/bin/bash", "-c"]
ENV DEBIAN_FRONTEND="noninteractive"
ENV TZ="Europe/London"
RUN cd / && mkdir env && cd env && apt-get update && \
    apt-get install -y git curl bedtools hdf5-tools pigz nano unzip wget gnupg && \
    apt-get autoclean -y && apt-get autoremove -y && \
    wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /env/miniconda3.sh && \
    bash /env/miniconda3.sh -b -p /env/miniconda3 && \
    source /env/miniconda3/bin/activate && conda create -y --name riboviz python=3.7 && conda activate riboviz && \
    conda install -y pyyaml gitpython pytest pandas && \
    conda install -y -c bioconda cutadapt && \
    conda install -y -c bioconda pysam && \
    conda install -y -c bioconda samtools && \
    conda install -y -c anaconda biopython && \
    pip install gffutils && \
    conda install -y -c anaconda h5py && \
    conda install -y -c bioconda umi_tools && \
    wget ftp://ftp.ccb.jhu.edu/pub/infphilo/hisat2/downloads/hisat2-2.1.0-Linux_x86_64.zip && \
    wget https://sourceforge.net/projects/bowtie-bio/files/bowtie/1.2.2/bowtie-1.2.2-linux-x86_64.zip/download -O bowtie.zip && \
    unzip hisat2-2.1.0-Linux_x86_64.zip && unzip bowtie.zip && \
    export PATH=/env/hisat2-2.1.0:$PATH && export PATH=/env/bowtie-1.2.2-linux-x86_64/:$PATH && \
    conda install -y -c bioconda nextflow && \
    echo 'deb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran35/'>>/etc/apt/sources.list && \
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9 && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
    apt-get update && apt-get install -y r-base r-base-dev libxml2-dev libssl-dev libcurl4-openssl-dev && \
    conda clean -t -y && apt-get clean && apt-get autoremove && apt-get autoclean && rm -rf *.zip && rm -rf miniconda3.sh && \
    cd /env && mkdir riboviz && cd riboviz && git clone https://github.com/riboviz/riboviz && cd riboviz && git checkout develop && cd .. && git clone https://github.com/riboviz/example-datasets.git && cd ~ && touch /env/setenv.sh && echo '#!/bin/bash'>>/env/setenv.sh && echo 'export PATH=/env/hisat2-2.1.0:$PATH'>>/env/setenv.sh && echo 'export PATH=/env/bowtie-1.2.2-linux-x86_64/:$PATH'>>/env/setenv.sh && echo 'source /env/miniconda3/bin/activate'>>/env/setenv.sh && echo 'conda activate riboviz'>>/env/setenv.sh
COPY --from=build /usr/local/lib/R/site-library  /usr/local/lib/R/site-library
COPY --from=build  /usr/lib/R/library /usr/lib/R/library
