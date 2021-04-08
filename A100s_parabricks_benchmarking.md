# A100's parabricks benchmarking

Created: 2021/04/06 17:17:47
Last modified: 2021/04/08 17:47:27

- **Aim:** Benchmarking the performance (speed) of the A100's on NeSi analysing public whole genome sequence (WSG) data
- **Prerequisite software:**
- **OS:** ESR cluster (CentOS Linux 7)

## Table of contents

- [A100's parabricks benchmarking](#a100s-parabricks-benchmarking)
  - [Table of contents](#table-of-contents)
  - [Overview](#overview)
  - [Getting onto NeSi](#getting-onto-nesi)
  - [Input data](#input-data)
  - [Setup](#setup)
    - [Create a screen to not loose my interactive slurm session if I get cut off](#create-a-screen-to-not-loose-my-interactive-slurm-session-if-i-get-cut-off)
    - [Setup an interactive slurm session](#setup-an-interactive-slurm-session)
    - [Load modules](#load-modules)
    - [Export paths](#export-paths)
    - [Test parabricks is running](#test-parabricks-is-running)
  - [Run genomes through parabricks](#run-genomes-through-parabricks)
    - [Sample data from NVIDIA on 2 GPUs](#sample-data-from-nvidia-on-2-gpus)
    - [A single whole genome on 2 GPUs](#a-single-whole-genome-on-2-gpus)
    - [A single whole genome on 8 GPUS](#a-single-whole-genome-on-8-gpus)
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

### Create a screen to not loose my interactive slurm session if I get cut off

```bash
screen -S pbricks
```
### Setup an interactive slurm session

We want to get to the wbl009 node (a big memory node on the mahuika cluster) because that is where the GPU's are

We want to set up an interactive slurm session so we can troubleshoot on the fly

- The "nesi03181" account is a GA account this work gets charged to
- If the slurm memory we've asked for with the `--mem` flag is too small for the 

```bash
srun --account nesi03181 --job-name pbtest --mem 100G --cpus-per-task 8 --gres=gpu:A100:2 --qos=testing --time 01:00:00 --pty bash
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

### Sample data from NVIDIA on 2 GPUs

```bash
cd /nesi/project/nesi03181/SW/parabricks

./pbrun fq2bam \
--ref /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/Ref/Homo_sapiens_assembly38.fasta \
--in-fq /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/Data/sample_1.fq.gz /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/Data/sample_2.fq.gz \
--out-bam /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/out/output.bam
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

### A single whole genome on 2 GPUs

```bash
cd /nesi/project/nesi03181/

./SW/parabricks/pbrun germline \
--ref /nesi/project/nesi03181/PBDATA/REFERENCE/human_g1k_v37_decoy.fasta \
--in-fq /nesi/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R1_001.fastq.gz /nesi/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R2_001.fastq.gz \
--knownSites /nesi/project/nesi03181/PBDATA/REFERENCE/1000G_phase1.indels.b37.vcf.gz \
--knownSites /nesi/project/nesi03181/PBDATA/REFERENCE/Mills_and_1000G_gold_standard.indels.b37.vcf.gz \
--out-bam /nesi/project/nesi03181/PBDATA/output.bam \
--out-variants /nesi/project/nesi03181/PBDATA/output.vcf \
--out-recal-file /nesi/project/nesi03181/PBDATA/report.txt
```

My output:

```bash
[lkemp@wbl009 nesi03181]$ ./SW/parabricks/pbrun germline \                                                                                      > --ref /nesi/project/nesi03181/PBDATA/REFE
RENCE/human_g1k_v37_decoy.fasta \                                                                    > --in-fq /nesi/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R1_001.fastq
.gz /nesi/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R2_001.fastq.gz \
> --knownSites /nesi/project/nesi03181/PBDATA/REFERENCE/1000G_phase1.indels.b37.vcf.gz \
> --knownSites /nesi/project/nesi03181/PBDATA/REFERENCE/Mills_and_1000G_gold_standard.indels.b37.vcf.gz \
> --out-bam /nesi/project/nesi03181/PBDATA/output.bam \
> --out-variants /nesi/project/nesi03181/PBDATA/output.vcf \
> --out-recal-file /nesi/project/nesi03181/PBDATA/report.txt
Please visit https://docs.nvidia.com/clara/#parabricks for detailed documentation


[Parabricks Options Mesg]: Automatically generating ID prefix
[Parabricks Options Mesg]: Read group created for
/scale_wlg_persistent/filesets/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R1_001.fastq.gz and
/scale_wlg_persistent/filesets/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R2_001.fastq.gz
[Parabricks Options Mesg]: @RG\tID:C6UP4ANXX.6\tLB:lib1\tPL:bar\tSM:sample\tPU:C6UP4ANXX.6

Please visit https://docs.nvidia.com/clara/#parabricks for detailed documentation


[Parabricks Options Mesg]: Checking argument compatibility
[Parabricks Options Mesg]: Read group created for
/scale_wlg_persistent/filesets/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R1_001.fastq.gz and
/scale_wlg_persistent/filesets/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R2_001.fastq.gz
[Parabricks Options Mesg]: @RG\tID:C6UP4ANXX.6\tLB:lib1\tPL:bar\tSM:sample\tPU:C6UP4ANXX.6
------------------------------------------------------------------------------
||                 Parabricks accelerated Genomics Pipeline                 ||
||                           Version v3.5.0_ampere                          ||
||                       GPU-BWA mem, Sorting Phase-I                       ||
||                  Contact: Parabricks-Support@nvidia.com                  ||
------------------------------------------------------------------------------
[M::bwa_idx_load_from_disk] read 0 ALT contigs
ProgressMeter   Reads           Base Pairs Aligned
[03:01:36]      5040000         620000000
[03:02:50]      10080000        1260000000
[03:04:05]      15120000        1880000000
[03:05:25]      20160000        2520000000
[03:06:45]      25200000        3160000000
[03:08:01]      30240000        3780000000
[03:09:17]      35280000        4400000000
[03:10:37]      40320000        5030000000
[03:11:59]      45360000        5670000000
[03:13:15]      50400000        6290000000
[03:14:34]      55440000        6910000000
[03:15:53]      60480000        7560000000
[03:17:11]      65520000        8190000000
[03:18:30]      70560000        8820000000
[03:19:46]      75600000        9440000000
[03:21:06]      80640000        10090000000
[03:22:28]      85680000        10710000000
[03:23:47]      90720000        11340000000
[03:25:06]      95760000        11960000000
[03:26:25]      100800000       12600000000
[03:27:42]      105840000       13220000000
[03:28:59]      110880000       13870000000
[03:30:18]      115920000       14480000000
[03:31:36]      120960000       15120000000
[03:32:58]      126000000       15750000000
[03:34:18]      131040000       16390000000
[03:35:37]      136080000       17000000000
[03:36:55]      141120000       17640000000
[03:38:13]      146160000       18280000000
[03:39:33]      151200000       18900000000
[03:40:51]      156240000       19520000000
[03:42:08]      161280000       20160000000
[03:43:29]      166320000       20790000000
[03:44:47]      171360000       21410000000
[03:46:05]      176400000       22040000000
[03:47:22]      181440000       22680000000
[03:48:42]      186480000       23320000000
[03:48:42]      186480000       23310000000
[03:50:00]      191600000       23940000000
[03:51:19]      196640000       24570000000
[03:52:36]      201680000       25220000000
[03:53:55]      206720000       25860000000
[03:55:13]      211760000       26480000000
[03:56:30]      216800000       27090000000
[03:57:50]      221840000       27720000000
[03:59:12]      226880000       28360000000
[04:00:31]      231920000       28980000000
[04:01:50]      236960000       29610000000
[04:03:09]      242000000       30240000000
[04:04:30]      247040000       30900000000
[04:05:49]      252080000       31520000000
[04:07:06]      257120000       32150000000
[04:08:24]      262160000       32770000000
[04:09:41]      267200000       33400000000
[04:11:03]      272240000       34030000000
[04:12:26]      277280000       34660000000
[04:13:43]      282320000       35300000000
[04:15:04]      287360000       35940000000
[04:16:16]      292400000       36550000000
[04:17:36]      297440000       37180000000
[04:18:53]      302480000       37800000000
[04:20:12]      307520000       38440000000
[04:21:30]      312560000       39050000000
[04:22:51]      317600000       39710000000
[04:24:08]      322640000       40330000000
[04:25:33]      327680000       40970000000
[04:26:45]      332720000       41590000000
[04:28:05]      337760000       42230000000
[04:29:25]      342800000       42860000000
[04:30:46]      347840000       43480000000
[04:32:06]      352880000       44120000000
[04:33:25]      357920000       44740000000
[04:34:43]      362960000       45360000000
[04:36:03]      368000000       46000000000
[04:37:24]      373040000       46630000000
[04:38:43]      378080000       47260000000
[04:39:59]      383120000       47880000000
[04:41:16]      388160000       48520000000
[04:42:32]      393200000       49140000000
[04:43:50]      398240000       49780000000
[04:45:09]      403280000       50410000000
[04:46:26]      408320000       51030000000
[04:47:47]      413360000       51670000000
[04:49:05]      418400000       52290000000

GPU-BWA Mem time: 6649.537225 seconds
GPU-BWA Mem is finished.

GPU Sorting, Marking Dups, BQSR
ProgressMeter   SAM Entries Completed

Total GPU-BWA Mem + Sorting + MarkingDups + BQSR Generation + BAM writing
Processing time: 6649.540243 seconds

[main] CMD: PARABRICKS mem -Z ./pbOpts.txt /scale_wlg_persistent/filesets/project/nesi03181/PBDATA/REFERENCE/human_g1k_v37_decoy.fasta /scale_wlg_persistent/filesets/project/nesi03181/PBD
ATA/MPHG007-23100082/HGHG007_S6_L006_R1_001.fastq.gz /scale_wlg_persistent/filesets/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R2_001.fastq.gz @RG\tID:C6UP4ANXX.6\tLB:lib1\
tPL:bar\tSM:sample\tPU:C6UP4ANXX.6
[main] Real time: 6653.146 sec; CPU: 51807.084 sec
------------------------------------------------------------------------------
||        Program:                      GPU-BWA mem, Sorting Phase-I        ||
||        Version:                                     v3.5.0_ampere        ||
||        Start Time:                       Thu Apr  8 03:00:10 2021        ||
||        End Time:                         Thu Apr  8 04:51:04 2021        ||
||        Total Time:                         110 minutes 54 seconds        ||
------------------------------------------------------------------------------
------------------------------------------------------------------------------
||                 Parabricks accelerated Genomics Pipeline                 ||
||                           Version v3.5.0_ampere                          ||
||                             Sorting Phase-II                             ||
||                  Contact: Parabricks-Support@nvidia.com                  ||
------------------------------------------------------------------------------
progressMeter - Percentage
[04:51:05]      0.0      0.00 GB
[04:51:15]      3.6      1.59 GB
[04:51:25]      9.1      1.53 GB
[04:51:35]      13.9     1.59 GB
[04:51:45]      18.4     1.63 GB
[04:51:55]      23.8     1.64 GB
[04:52:05]      27.8     1.59 GB
[04:52:15]      32.9     1.55 GB
[04:52:25]      38.2     1.51 GB
[04:52:35]      42.5     1.65 GB
[04:52:45]      47.8     1.24 GB
[04:52:55]      53.6     1.56 GB
[04:53:05]      58.3     1.61 GB
[04:53:15]      62.9     1.28 GB
[04:53:25]      69.5     1.41 GB
[04:53:35]      74.4     1.92 GB
[04:53:45]      79.1     1.60 GB
[04:53:55]      84.8     1.31 GB
[04:54:05]      90.8     1.58 GB
[04:54:15]      96.9     3.40 GB
[04:54:25]      98.9     2.27 GB
Sorting and Marking: 210.009 seconds
------------------------------------------------------------------------------
||        Program:                                  Sorting Phase-II        ||
||        Version:                                     v3.5.0_ampere        ||
||        Start Time:                       Thu Apr  8 04:51:05 2021        ||
||        End Time:                         Thu Apr  8 04:54:35 2021        ||
||        Total Time:                           3 minutes 30 seconds        ||
------------------------------------------------------------------------------
------------------------------------------------------------------------------
||                 Parabricks accelerated Genomics Pipeline                 ||
||                           Version v3.5.0_ampere                          ||
||                         Marking Duplicates, BQSR                         ||
||                  Contact: Parabricks-Support@nvidia.com                  ||
------------------------------------------------------------------------------
progressMeter - Percentage
[04:54:45]      0.0      6.61 GB
[04:54:55]      0.0      12.97 GB
[04:55:05]      0.0      19.31 GB
[04:55:15]      0.0      25.73 GB
[04:55:25]      0.0      31.96 GB
[04:55:35]      0.0      38.48 GB
[04:55:45]      0.0      44.32 GB
[04:55:55]      1.0      48.92 GB
[04:56:05]      3.1      51.20 GB
[04:56:15]      5.8      54.21 GB
[04:56:25]      7.9      56.64 GB
[04:56:35]      10.0     59.43 GB
[04:56:45]      12.4     61.76 GB
[04:56:55]      14.4     64.60 GB
[04:57:05]      16.6     66.82 GB
[04:57:15]      18.8     69.38 GB
[04:57:25]      20.6     72.28 GB
[04:57:35]      22.7     74.85 GB
[04:57:45]      24.7     77.55 GB
[04:57:55]      26.7     79.80 GB
[04:58:05]      28.9     82.59 GB
[04:58:15]      31.0     85.20 GB
[04:58:25]      32.9     87.78 GB
[04:58:35]      35.1     90.45 GB
[04:58:45]      37.3     92.10 GB
[04:58:55]      39.2     95.62 GB
Please contact Parabricks-Support@nvidia.com for any questions
There is a forum for Q&A as well at https://forums.developer.nvidia.com/c/healthcare/Parabricks/290
Exiting...

Could not run fq2bam as part of germline pipeline
Exiting pbrun ...

```

It ended up erroring out, likely due to a lack of memory again...same error as before

I asked for more memory from slurm

```bash
# Start interactive slurm session with 8 GPUS
srun --account nesi03181 --job-name pbtest --mem 200G --cpus-per-task 8 --gres=gpu:A100:2 --qos=testing --time 03:00:00 --pty bash

# Load modules
module load Singularity/3.7.1 CUDA/11.2.0 Python

# Set paths
export SINGULARITYENV_LD_LIBRARY_PATH="\${LD_LIBRARY_PATH}:${LD_LIBRARY_PATH}"
export SINGULARITY_BIND="/cm/local/apps/cuda,${EBROOTCUDA}"

# Run pipeline
cd /nesi/project/nesi03181/

./SW/parabricks/pbrun germline \
--ref /nesi/project/nesi03181/PBDATA/REFERENCE/human_g1k_v37_decoy.fasta \
--in-fq /nesi/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R1_001.fastq.gz /nesi/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R2_001.fastq.gz \
--knownSites /nesi/project/nesi03181/PBDATA/REFERENCE/1000G_phase1.indels.b37.vcf.gz \
--knownSites /nesi/project/nesi03181/PBDATA/REFERENCE/Mills_and_1000G_gold_standard.indels.b37.vcf.gz \
--out-bam /nesi/project/nesi03181/PBDATA/output.bam \
--out-variants /nesi/project/nesi03181/PBDATA/output.vcf \
--out-recal-file /nesi/project/nesi03181/PBDATA/report.txt
```

### A single whole genome on 8 GPUS

Looks like I can't request that many GPU's

```bash
[lkemp@mahuika01 nesi03181]$ srun --account nesi03181 --job-name pbtest --mem 200G --cpus-per-task 8 --gres=gpu:A100:8 --qos=testing --time 01:00:00 --pty bash
srun: error: MaxGRESPerAccount
srun: error: Unable to allocate resources: Job violates accounting/QOS policy (job submit limit, user's size and/or time limits)
```


```bash
# Start screen
screen -S pbricks

# Start interactive slurm session with 8 GPUS
srun --account nesi03181 --job-name pbtest --mem 200G --cpus-per-task 8 --gres=gpu:A100:8 --qos=testing --time 01:00:00 --pty bash

# Load modules
module load Singularity/3.7.1 CUDA/11.2.0 Python

# Set paths
export SINGULARITYENV_LD_LIBRARY_PATH="\${LD_LIBRARY_PATH}:${LD_LIBRARY_PATH}"
export SINGULARITY_BIND="/cm/local/apps/cuda,${EBROOTCUDA}"
```

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

CMake/3.14.1
   CMake/3.16.5
   CMake/3.19.5 


