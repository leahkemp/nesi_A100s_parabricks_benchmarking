# Calculate mapping metrics

Created: 2021/04/06 17:17:47
Last modified: 2021/04/08 14:43:20

- **Aim:** Benchmarking the performance (speed) of the A100's on NeSi analysing public whole genome sequence (WSG) data
- **Prerequisite software:**
- **OS:** ESR cluster (CentOS Linux 7)

## Table of contents

- [Calculate mapping metrics](#calculate-mapping-metrics)
  - [Table of contents](#table-of-contents)
  - [Overview](#overview)
  - [Getting onto NeSi](#getting-onto-nesi)
  - [Input data](#input-data)
  - [Setup](#setup)
    - [Setup an interactive slurm session](#setup-an-interactive-slurm-session)
    - [Create a screen to not loose my interactive slurm session if I get cut off](#create-a-screen-to-not-loose-my-interactive-slurm-session-if-i-get-cut-off)
    - [Load modules](#load-modules)
    - [Export paths](#export-paths)
    - [Test parabricks is running](#test-parabricks-is-running)
  - [Run genomes through parabricks](#run-genomes-through-parabricks)
    - [Sample data from NVIDIA](#sample-data-from-nvidia)
    - [A single whole genome](#a-single-whole-genome)
    - [Watch the gpus running](#watch-the-gpus-running)
    - [Cancel interactive slurm session](#cancel-interactive-slurm-session)

## Overview

Up till now, we have run our 

NeSi has 8x [NVIDIA A100 GPU's](https://www.nvidia.com/en-us/data-center/a100/)

## Getting onto NeSi

I've currently got it set up so that I can connect to NeSi via Wintermute. I ended up doing this because I had some difficulties installing a linux subsystem on my ESR laptop

My current ssh config file:

```bash
cat ~/.ssh/config
```

My output:

```bash
Host remote
    HostName oak
    ProxyCommand ssh -i ~/.ssh/id_rsa -W %h:%p lkemp@oak

Host tfs.esr.cri.nz
    ProxyCommand ssh -i ~/.ssh/id_rsa -W %h:%p remote

Host orac
    ProxyCommand ssh -i ~/.ssh/id_rsa -W %h:%p remote

Host mahuika
  User lkemp
  Hostname login.mahuika.nesi.org.nz
  ProxyCommand ssh -W %h:%p lander
  ForwardX11 yes
  ForwardX11Trusted yes
  ServerAliveInterval 300
  ServerAliveCountMax 2
  Host lander
  User lkemp
  HostName lander.nesi.org.nz
  ForwardX11 yes
  ForwardX11Trusted yes
  ServerAliveInterval 300
  ServerAliveCountMax 2
```

That means the below command will connect me to lander, then mahuika on NeSi

```bash
ssh mahuika
```

Parabricks etc is installed on mahuika (a cluster) so that's were we want to be

## Input data

Our project directory has been set up at `/nesi/project/nesi03181`. Let's have a look see

```bash
cd /nesi/project/nesi03181
ls -lh
```

My output:

```bash
total 2.5K
drwxrws---+ 3 adejorod nesi03181 4.0K Nov 16 22:25 MGSS_Intro
drwxrws---+ 3 dsen018  nesi03181 4.0K Nov 16 07:00 MGSS_resources_2020
drwxrws---+ 3 dsen018  nesi03181 4.0K Mar 31 07:51 Parabricks
drwxrws---+ 5 dsen018  nesi03181 4.0K Apr  6 01:54 PBDATA
drwxrws---+ 3 dsen018  nesi03181 4.0K Mar 31 19:16 SW
```

Miles has already transferred the human reference data (b37) to `/nesi/project/nesi03181/PBDATA/REFERENCE/` and the public genomes to run through parabricks to `/nesi/project/nesi03181/PBDATA/`

Have a look:

```bash
ls -lh /nesi/project/nesi03181/PBDATA/REFERENCE/
```

```bash
total 30G
-rw-rw----+ 1 mbenton nesi03181 227M Apr  6 02:06 1000G_phase1.indels.b37.vcf
-rw-rw----+ 1 mbenton nesi03181  43M Apr  6 02:06 1000G_phase1.indels.b37.vcf.gz
-rw-rw----+ 1 mbenton nesi03181  101 Apr  6 02:06 1000G_phase1.indels.b37.vcf.gz.md5
-rw-rw----+ 1 mbenton nesi03181 1.2M Apr  6 02:06 1000G_phase1.indels.b37.vcf.idx
-rw-rw----+ 1 mbenton nesi03181 326K Apr  6 02:06 1000G_phase1.indels.b37.vcf.idx.gz
-rw-rw----+ 1 mbenton nesi03181  105 Apr  6 02:06 1000G_phase1.indels.b37.vcf.idx.gz.md5
-rw-rw----+ 1 mbenton nesi03181 2.3G Apr  6 01:56 dbsnp_138.b37.excluding_sites_after_129.vcf
-rw-rw----+ 1 mbenton nesi03181 334M Apr  6 01:56 dbsnp_138.b37.excluding_sites_after_129.vcf.gz
-rw-rw----+ 1 mbenton nesi03181  117 Apr  6 01:56 dbsnp_138.b37.excluding_sites_after_129.vcf.gz.md5
-rw-rw----+ 1 mbenton nesi03181  12M Apr  6 01:56 dbsnp_138.b37.excluding_sites_after_129.vcf.idx
-rw-rw----+ 1 mbenton nesi03181 3.7M Apr  6 01:56 dbsnp_138.b37.excluding_sites_after_129.vcf.idx.gz
-rw-rw----+ 1 mbenton nesi03181  121 Apr  6 01:56 dbsnp_138.b37.excluding_sites_after_129.vcf.idx.gz.md5
-rw-rw----+ 1 mbenton nesi03181  10G Apr  6 01:58 dbsnp_138.b37.vcf
-rw-rw----+ 1 mbenton nesi03181 1.4G Apr  6 01:58 dbsnp_138.b37.vcf.gz
-rw-rw----+ 1 mbenton nesi03181   91 Apr  6 01:58 dbsnp_138.b37.vcf.gz.md5
-rw-rw----+ 1 mbenton nesi03181  12M Apr  6 01:58 dbsnp_138.b37.vcf.idx
-rw-rw----+ 1 mbenton nesi03181 3.9M Apr  6 01:58 dbsnp_138.b37.vcf.idx.gz
-rw-rw----+ 1 mbenton nesi03181   95 Apr  6 01:58 dbsnp_138.b37.vcf.idx.gz.md5
-rw-rw----+ 1 mbenton nesi03181  22M Apr  6 01:58 dbsnp_138.b37.vcf.sidx
-rw-rw----+ 1 mbenton nesi03181  10K Apr  6 02:02 human_g1k_v37_decoy.dict
-rw-rw----+ 1 mbenton nesi03181 2.6K Apr  6 02:02 human_g1k_v37_decoy.dict.gz
-rw-rw----+ 1 mbenton nesi03181   98 Apr  6 02:02 human_g1k_v37_decoy.dict.gz.md5
-rw-rw----+ 1 mbenton nesi03181 3.0G Apr  6 02:02 human_g1k_v37_decoy.fasta
-rw-rw----+ 1 mbenton nesi03181 105K Apr  6 02:02 human_g1k_v37_decoy.fasta.amb
-rw-rw----+ 1 mbenton nesi03181 6.9K Apr  6 02:02 human_g1k_v37_decoy.fasta.ann
-rw-rw----+ 1 mbenton nesi03181 3.0G Apr  6 02:03 human_g1k_v37_decoy.fasta.bwt
-rw-rw----+ 1 mbenton nesi03181 2.8K Apr  6 02:03 human_g1k_v37_decoy.fasta.fai
-rw-rw----+ 1 mbenton nesi03181 1.8K Apr  6 02:03 human_g1k_v37_decoy.fasta.fai.gz
-rw-rw----+ 1 mbenton nesi03181  103 Apr  6 02:03 human_g1k_v37_decoy.fasta.fai.gz.md5
-rw-rw----+ 1 mbenton nesi03181 3.0G Apr  6 02:03 human_g1k_v37_decoy.fasta.flat
-rw-rw----+ 1 mbenton nesi03181 6.9K Apr  6 02:03 human_g1k_v37_decoy.fasta.gdx
-rw-rw----+ 1 mbenton nesi03181 839M Apr  6 02:03 human_g1k_v37_decoy.fasta.gz
-rw-rw----+ 1 mbenton nesi03181   99 Apr  6 02:03 human_g1k_v37_decoy.fasta.gz.md5
-rw-rw----+ 1 mbenton nesi03181 749M Apr  6 02:04 human_g1k_v37_decoy.fasta.pac
-rw-rw----+ 1 mbenton nesi03181 1.5G Apr  6 02:04 human_g1k_v37_decoy.fasta.sa
-rw-rw----+ 1 mbenton nesi03181 2.6K Apr  6 02:01 human_g1k_v37.dict.gz
-rw-rw----+ 1 mbenton nesi03181   92 Apr  6 02:01 human_g1k_v37.dict.gz.md5
-rw-rw----+ 1 mbenton nesi03181 3.0G Apr  6 02:00 human_g1k_v37.fasta
-rw-rw----+ 1 mbenton nesi03181 2.7K Apr  6 02:00 human_g1k_v37.fasta.fai
-rw-rw----+ 1 mbenton nesi03181 1.1K Apr  6 02:00 human_g1k_v37.fasta.fai.gz
-rw-rw----+ 1 mbenton nesi03181   97 Apr  6 02:00 human_g1k_v37.fasta.fai.gz.md5
-rw-rw----+ 1 mbenton nesi03181 830M Apr  6 02:00 human_g1k_v37.fasta.gz
-rw-rw----+ 1 mbenton nesi03181   93 Apr  6 02:00 human_g1k_v37.fasta.gz.md5
-rw-rw----+ 1 mbenton nesi03181  83M Apr  6 02:05 Mills_and_1000G_gold_standard.indels.b37.vcf
-rw-rw----+ 1 mbenton nesi03181  19M Apr  6 02:05 Mills_and_1000G_gold_standard.indels.b37.vcf.gz
-rw-rw----+ 1 mbenton nesi03181  118 Apr  6 02:05 Mills_and_1000G_gold_standard.indels.b37.vcf.gz.md5
-rw-rw----+ 1 mbenton nesi03181 2.3M Apr  6 02:05 Mills_and_1000G_gold_standard.indels.b37.vcf.idx
-rw-rw----+ 1 mbenton nesi03181 536K Apr  6 02:05 Mills_and_1000G_gold_standard.indels.b37.vcf.idx.gz
-rw-rw----+ 1 mbenton nesi03181  122 Apr  6 02:05 Mills_and_1000G_gold_standard.indels.b37.vcf.idx.gz.md5
```

```bash
ls -lh /nesi/project/nesi03181/PBDATA/MPHG007-23100082/
```

```bash
total 36G
-rw-rw----+ 1 dsen018 nesi03181 18G Jul 31  2015 HGHG007_S6_L006_R1_001.fastq.gz
-rw-rw----+ 1 dsen018 nesi03181 18G Jul 31  2015 HGHG007_S6_L006_R2_001.fastq.gz 
```

```bash
ls -lh /nesi/project/nesi03181/PBDATA/MPHG007-23110112/
```

```bash
total 38G
-rw-rw----+ 1 dsen018 nesi03181 19G Jul 31  2015 HGHG007_S6_L006_R1_001.fastq.gz
-rw-rw----+ 1 dsen018 nesi03181 20G Jul 31  2015 HGHG007_S6_L006_R2_001.fastq.gz
```

So these are the fastq's for the mother of the publicly available "ChineseTrio" data

ftp: https://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/ChineseTrio/HG007_NA24695-hu38168_mother/NIST_Stanford_Illumina_6kb_matepair/fastqs/MPHG007-23100082/

Let's get the data (fastq) for the other individuals in the trio

ftp for son: https://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/ChineseTrio/HG005_NA24631_son/NIST_Stanford_Illumina_6kb_matepair/fastqs/

ftp for father: https://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/ChineseTrio/HG006_NA24694-huCA017E_father/NIST_Stanford_Illumina_6kb_matepair/fastqs/

```bash
cd /nesi/project/nesi03181/PBDATA/

mkdir MPHG005-23100080
mkdir MPHG005-23110110
mkdir MPHG006-23100081
mkdir MPHG006-23110111

cd MPHG005-23100080

wget https://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/ChineseTrio/HG005_NA24631_son/NIST_Stanford_Illumina_6kb_matepair/fastqs/MPHG005-23100080/MPHG005_S4_L004_R1_001.fastq.gz
wget https://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/ChineseTrio/HG005_NA24631_son/NIST_Stanford_Illumina_6kb_matepair/fastqs/MPHG005-23100080/MPHG005_S4_L004_R2_001.fastq.gz

cd ../MPHG005-23110110/

wget https://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/ChineseTrio/HG005_NA24631_son/NIST_Stanford_Illumina_6kb_matepair/fastqs/MPHG005-23110110/MPHG005_S4_L004_R1_001.fastq.gz
wget https://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/ChineseTrio/HG005_NA24631_son/NIST_Stanford_Illumina_6kb_matepair/fastqs/MPHG005-23110110/MPHG005_S4_L004_R2_001.fastq.gz

cd ../MPHG006-23100081

wget https://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/ChineseTrio/HG006_NA24694-huCA017E_father/NIST_Stanford_Illumina_6kb_matepair/fastqs/MPHG006-23100081/MPHG006_S5_L005_R1_001.fastq.gz
wget https://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/ChineseTrio/HG006_NA24694-huCA017E_father/NIST_Stanford_Illumina_6kb_matepair/fastqs/MPHG006-23100081/MPHG006_S5_L005_R2_001.fastq.gz

cd ../MPHG006-23110111

wget https://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/ChineseTrio/HG006_NA24694-huCA017E_father/NIST_Stanford_Illumina_6kb_matepair/fastqs/MPHG006-23110111/MPHG006_S5_L005_R1_001.fastq.gz
wget https://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/ChineseTrio/HG006_NA24694-huCA017E_father/NIST_Stanford_Illumina_6kb_matepair/fastqs/MPHG006-23110111/MPHG006_S5_L005_R2_001.fastq.gz
```

A summary of the individuals in the trio:

- MPHG007-23100082: mother
- MPHG007-23110112: mother
- MPHG005-23100080: son
- MPHG005-23110110: son
- MPHG006-23100081: father
- MPHG006-23110111: father

## Setup

### Setup an interactive slurm session

We want to get to the wbl009 node (a big memory node on the mahuika cluster) because that is where the GPU's are

We want to set up an interactive slurm session so we can troubleshoot on the fly

- The "nesi03181" account is a GA account this work gets charged to
- If the slurm memory we've asked for with the `--mem` flag is too small for the 

```bash
srun --account nesi03181 --job-name pbtest --mem 200G --cpus-per-task 8 --gres=gpu:A100 --qos=testing --time 01:00:00 --pty bash
```

### Create a screen to not loose my interactive slurm session if I get cut off

```bash
screen -S pbricks
```

### Load modules

My currently loaded modules (shouldn't be much because I haven't loaded any yet)

```bash
module list
```

My output:

```bash

Currently Loaded Modules:
  1) slurm   2) NeSI (S)   3) craype-broadwell   4) craype-network-infiniband   5) gcc/4.9.1

```

Parabricks is containerised (in a singularity container) so we're gonna need Singularity to interact with it

```bash
module load Singularity/3.7.1 CUDA/11.2.0 Python
```

Check it loaded

```bash
module list
```

My output:

```bash

Currently Loaded Modules:
  1) slurm   2) NeSI (S)   3) craype-broadwell   4) craype-network-infiniband   5) gcc/4.9.1   6) Singularity/3.7.1

```

Great

### Export paths

This is required to bind the paths to the singularity container, Dini could set this up in the build for the non-trial version of parabricks, but it's too much unnecesary work for now

```bash
export SINGULARITYENV_LD_LIBRARY_PATH="\${LD_LIBRARY_PATH}:${LD_LIBRARY_PATH}"
export SINGULARITY_BIND="/cm/local/apps/cuda,${EBROOTCUDA}"
```

### Test parabricks is running

The Parabricks binary can be found at `/nesi/project/nesi03181/SW/parabricks/pbrun` and can be called directly

```bash
/nesi/project/nesi03181/SW/parabricks/pbrun --help
```

## Run genomes through parabricks

### Sample data from NVIDIA

```bash
cd /nesi/project/nesi03181/SW/parabricks

./pbrun fq2bam --ref /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/Ref/Homo_sapiens_assembly38.fasta --in-fq /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/Data/sample_1.fq.gz /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/Data/sample_2.fq.gz --out-bam /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/out/output.bam
```

My output

```bash
[lkemp@wbl009 parabricks]$ ./pbrun fq2bam --ref /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/Ref/Homo_sapiens_assembly38.fasta --in
-fq /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/Data/sample_1.fq.gz /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/Data
/sample_2.fq.gz --out-bam /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/out/output.bam
Please visit https://docs.nvidia.com/clara/#parabricks for detailed documentation


Please visit https://docs.nvidia.com/clara/#parabricks for detailed documentation


[Parabricks Options Mesg]: Checking argument compatibility
[Parabricks Options Mesg]: Automatically generating ID prefix
[Parabricks Options Mesg]: Read group created for
/nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/Data/sample_1.fq.gz and
/nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/Data/sample_2.fq.gz
[Parabricks Options Mesg]: @RG\tID:HK3TJBCX2.1\tLB:lib1\tPL:bar\tSM:sample\tPU:HK3TJBCX2.1
------------------------------------------------------------------------------
||                 Parabricks accelerated Genomics Pipeline                 ||
||                           Version v3.5.0_ampere                          ||
||                       GPU-BWA mem, Sorting Phase-I                       ||
||                  Contact: Parabricks-Support@nvidia.com                  ||
------------------------------------------------------------------------------
[M::bwa_idx_load_from_disk] read 0 ALT contigs

GPU-BWA mem
ProgressMeter   Reads           Base Pairs Aligned
[01:57:58]      5043564         560000000
[01:58:37]      10087128        1160000000
[01:59:16]      15130692        1750000000
[01:59:54]      20174256        2330000000
[02:00:32]      25217820        2910000000
[02:01:08]      30261384        3470000000
[02:01:47]      35304948        4050000000
[02:02:25]      40348512        4650000000
[02:03:02]      45392076        5220000000
[02:03:40]      50435640        5810000000

GPU-BWA Mem time: 444.235937 seconds
GPU-BWA Mem is finished.

GPU Sorting, Marking Dups, BQSR
ProgressMeter   SAM Entries Completed

Total GPU-BWA Mem + Sorting + MarkingDups + BQSR Generation + BAM writing
Processing time: 444.238573 seconds

[main] CMD: PARABRICKS mem -Z ./pbOpts.txt /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/Ref/Homo_sapiens_assembly38.fasta /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/Data/sample_1.fq.gz /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/Data/sample_2.fq.gz @RG\tID:HK3TJBCX2.1\tLB:lib1\tPL:bar\tSM:sample\tPU:HK3TJBCX2.1
[main] Real time: 447.882 sec; CPU: 3432.603 sec
------------------------------------------------------------------------------
||        Program:                      GPU-BWA mem, Sorting Phase-I        ||
||        Version:                                     v3.5.0_ampere        ||
||        Start Time:                       Thu Apr  8 01:56:53 2021        ||
||        End Time:                         Thu Apr  8 02:04:22 2021        ||
||        Total Time:                           7 minutes 29 seconds        ||
------------------------------------------------------------------------------
------------------------------------------------------------------------------
||                 Parabricks accelerated Genomics Pipeline                 ||
||                           Version v3.5.0_ampere                          ||
||                             Sorting Phase-II                             ||
||                  Contact: Parabricks-Support@nvidia.com                  ||
------------------------------------------------------------------------------
progressMeter - Percentage
[02:04:23]      0.0      0.00 GB
[02:04:33]      52.6     0.22 GB
Sorting and Marking: 20.003 seconds
------------------------------------------------------------------------------
||        Program:                                  Sorting Phase-II        ||
||        Version:                                     v3.5.0_ampere        ||
||        Start Time:                       Thu Apr  8 02:04:23 2021        ||
||        End Time:                         Thu Apr  8 02:04:43 2021        ||
||        Total Time:                                     20 seconds        ||
------------------------------------------------------------------------------
------------------------------------------------------------------------------
||                 Parabricks accelerated Genomics Pipeline                 ||
||                           Version v3.5.0_ampere                          ||
||                         Marking Duplicates, BQSR                         ||
||                  Contact: Parabricks-Support@nvidia.com                  ||
------------------------------------------------------------------------------
progressMeter - Percentage
[02:04:53]      19.2     1.76 GB
[02:05:03]      41.8     2.32 GB
[02:05:13]      61.6     3.34 GB
[02:05:23]      82.2     2.40 GB
[02:05:33]      100.0    0.00 GB
BQSR and writing final BAM:  50.042 seconds
------------------------------------------------------------------------------
||        Program:                          Marking Duplicates, BQSR        ||
||        Version:                                     v3.5.0_ampere        ||
||        Start Time:                       Thu Apr  8 02:04:43 2021        ||
||        End Time:                         Thu Apr  8 02:05:33 2021        ||
||        Total Time:                                     50 seconds        ||
------------------------------------------------------------------------------

```

### A single whole genome

```bash
cd /nesi/project/nesi03181/SW/parabricks

./pbrun germline --ref /nesi/project/nesi03181/PBDATA/REFERENCE/human_g1k_v37_decoy.fasta \
--in-fq /nesi/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R1_001.fastq.gz /nesi/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R2_001.fastq.gz \
--knownSites /nesi/project/nesi03181/PBDATA/REFERENCE/1000G_phase1.indels.b37.vcf.gz \
--knownSites /nesi/project/nesi03181/PBDATA/REFERENCE/Mills_and_1000G_gold_standard.indels.b37.vcf.gz \
--out-bam /nesi/project/nesi03181/PBDATA/output.bam \
--out-variants /nesi/project/nesi03181/PBDATA/output.vcf \
--out-recal-file /nesi/project/nesi03181/PBDATA/report.txt
```

My output

```bash

```

### Watch the gpus running

In a new session I can ssh into mahuika cluster again, ssh directly to the wbl009 node and run nvidia-smi (first I'll need to load the CUDA module)

```bash
ssh mahuika

shh wbl009

module load CUDA
watch nvidia-smi
```

### Cancel interactive slurm session

Once finished, cancel the interactive slurm session and free up the resources

```bash
exit
```


