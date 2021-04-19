# A100's parabricks benchmarking

Created: 2021/04/06 17:17:47
Last modified: 2021/04/19 12:21:22

- **Aim:** Benchmarking the performance (speed) of the [A100's](https://www.nvidia.com/en-us/data-center/a100/) on [NeSi](https://www.nesi.org.nz/) analysing public whole genome sequence (WSG) data
- **Prerequisite software:**
- **OS:**

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
    - [A single whole genome on 2 GPUs (repeat to test for variance)](#a-single-whole-genome-on-2-gpus-repeat-to-test-for-variance)
    - [A single whole genome on 8 GPUS](#a-single-whole-genome-on-8-gpus)
  - [Watch the gpus running](#watch-the-gpus-running)
  - [Cancel interactive slurm session](#cancel-interactive-slurm-session)
  - [More documentation](#more-documentation)

## Overview

NeSi has 8x [NVIDIA A100 GPU's](https://www.nvidia.com/en-us/data-center/a100/)

## Getting onto NeSi

I've currently got it set up so that I can connect to NeSi via Wintermute (one of ESR's research servers). I ended up doing this because I had some difficulties installing a linux subsystem on my ESR laptop, because Windows and IT

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

Let's have a look:

```bash
ls -lh /nesi/project/nesi03181/PBDATA/REFERENCE/
```

My output:

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

My output:

```bash
total 36G
-rw-rw----+ 1 dsen018 nesi03181 18G Jul 31  2015 HGHG007_S6_L006_R1_001.fastq.gz
-rw-rw----+ 1 dsen018 nesi03181 18G Jul 31  2015 HGHG007_S6_L006_R2_001.fastq.gz 
```

```bash
ls -lh /nesi/project/nesi03181/PBDATA/MPHG007-23110112/
```

My output:

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

*Note. the above code didn't work when run on Mahuika (NeSi) or Wintermute (ESR research server)*

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

Parabricks is containerised (in a singularity container) so we're gonna need Singularity to interact with parabricks

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

This is required to bind the paths to the singularity container, Dini could set this up to be done automatically in the build for the non-trial version of parabricks, but it's too much unnecessary work for now while we're just using a trial licence

```bash
export SINGULARITYENV_LD_LIBRARY_PATH="\${LD_LIBRARY_PATH}:${LD_LIBRARY_PATH}"
export SINGULARITY_BIND="/cm/local/apps/cuda,${EBROOTCUDA}"
```

### Test parabricks is running

The Parabricks binary can be found at `/nesi/project/nesi03181/SW/parabricks/pbrun` and can be called directly

```bash
/nesi/project/nesi03181/SW/parabricks/pbrun --help
```

Looks good

## Run genomes through parabricks

### Sample data from NVIDIA on 2 GPUs

```bash
cd /nesi/project/nesi03181/SW/parabricks

./pbrun fq2bam \
--ref /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/Ref/Homo_sapiens_assembly38.fasta \
--in-fq /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/Data/sample_1.fq.gz /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/Data/sample_2.fq.gz \
--out-bam /nesi/nobackup/nesi03181/PB_SAMPLE_DATA/parabricks_sample/out/output.bam
```

My output:

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

Error out again, slurm ran out of time this time, I'll give it more time

```bash
# Start interactive slurm session with 8 GPUS
srun --account nesi03181 --job-name pbtest --mem 200G --cpus-per-task 8 --gres=gpu:A100:2 --qos=testing --time 07:00:00 --pty bash

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

My output:

```bash
[lkemp@wbl009 nesi03181]$ ./SW/parabricks/pbrun germline \
> --ref /nesi/project/nesi03181/PBDATA/REFERENCE/human_g1k_v37_decoy.fasta \
> --in-fq /nesi/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R1_001.fastq.gz /nesi/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R2_001.fastq.gz \
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

GPU-BWA mem
ProgressMeter   Reads           Base Pairs Aligned
[08:45:09]      5040000         630000000
[08:46:27]      10080000        1250000000
[08:47:44]      15120000        1870000000
[08:49:03]      20160000        2520000000
[08:50:21]      25200000        3140000000
[08:51:37]      30240000        3780000000
[08:52:55]      35280000        4410000000
[08:54:15]      40320000        5040000000
[08:55:34]      45360000        5670000000
[08:56:51]      50400000        6310000000
[08:58:08]      55440000        6930000000
[08:59:24]      60480000        7560000000
[09:00:42]      65520000        8190000000
[09:02:00]      70560000        8810000000
[09:03:16]      75600000        9450000000
[09:04:33]      80640000        10070000000
[09:04:33]      80640000        10100000000
[09:05:49]      85760000        10720000000
[09:07:06]      90800000        11340000000
[09:08:24]      95840000        11980000000
[09:09:45]      100880000       12620000000
[09:11:03]      105920000       13240000000
[09:12:20]      110960000       13870000000
[09:13:38]      116000000       14500000000
[09:14:55]      121040000       15130000000
[09:16:17]      126080000       15760000000
[09:17:37]      131120000       16390000000
[09:18:54]      136160000       17020000000
[09:20:14]      141200000       17650000000
[09:21:34]      146240000       18280000000
[09:22:52]      151280000       18920000000
[09:24:12]      156320000       19540000000
[09:25:30]      161360000       20170000000
[09:26:50]      166400000       20790000000
[09:28:06]      171440000       21430000000
[09:29:24]      176480000       22070000000
[09:30:41]      181520000       22690000000
[09:31:59]      186560000       23320000000
[09:33:16]      191600000       23950000000
[09:34:34]      196640000       24580000000
[09:35:52]      201680000       25220000000
[09:37:13]      206720000       25860000000
[09:38:29]      211760000       26460000000
[09:39:49]      216800000       27090000000
[09:41:06]      221840000       27720000000
[09:42:25]      226880000       28360000000
[09:43:41]      231920000       28990000000
[09:44:58]      236960000       29620000000
[09:46:12]      242000000       30250000000
[09:47:33]      247040000       30900000000
[09:48:49]      252080000       31500000000
[09:50:06]      257120000       32140000000
[09:51:23]      262160000       32780000000
[09:52:40]      267200000       33400000000
[09:53:58]      272240000       34040000000
[09:55:22]      277280000       34670000000
[09:56:37]      282320000       35290000000
[09:57:55]      287360000       35920000000
[09:59:15]      292400000       36550000000
[10:00:35]      297440000       37180000000
[10:01:53]      302480000       37810000000
[10:03:10]      307520000       38430000000
[10:04:27]      312560000       39060000000
[10:05:44]      317600000       39690000000
[10:07:00]      322640000       40330000000
[10:08:17]      327680000       40970000000
[10:09:37]      332720000       41580000000
[10:10:56]      337760000       42220000000
[10:12:14]      342800000       42850000000
[10:13:32]      347840000       43480000000
[10:14:50]      352880000       44110000000
[10:16:10]      357920000       44740000000
[10:17:28]      362960000       45370000000
[10:18:48]      368000000       46020000000
[10:20:09]      373040000       46630000000
[10:21:26]      378080000       47260000000
[10:22:44]      383120000       47900000000
[10:24:02]      388160000       48510000000
[10:25:19]      393200000       49150000000
[10:26:36]      398240000       49770000000
[10:27:53]      403280000       50420000000
[10:29:13]      408320000       51070000000
[10:30:30]      413360000       51670000000
[10:31:50]      418400000       52300000000

GPU-BWA Mem time: 6590.479159 seconds
GPU-BWA Mem is finished.

GPU Sorting, Marking Dups, BQSR
ProgressMeter   SAM Entries Completed

Total GPU-BWA Mem + Sorting + MarkingDups + BQSR Generation + BAM writing
Processing time: 6590.482307 seconds

[main] CMD: PARABRICKS mem -Z ./pbOpts.txt /scale_wlg_persistent/filesets/project/nesi03181/PBDATA/REFERENCE/human_g1k_v37_decoy.fasta /scale_wlg_persistent/filesets/project/ple\tPU:C6UP4ANXX.6
[main] Real time: 6594.127 sec; CPU: 52210.417 sec
------------------------------------------------------------------------------
||        Program:                      GPU-BWA mem, Sorting Phase-I        ||
||        Version:                                     v3.5.0_ampere        ||
||        Start Time:                       Thu Apr  8 08:43:36 2021        ||
||        End Time:                         Thu Apr  8 10:33:31 2021        ||
||        Total Time:                         109 minutes 55 seconds        ||
------------------------------------------------------------------------------
------------------------------------------------------------------------------
||                 Parabricks accelerated Genomics Pipeline                 ||
||                           Version v3.5.0_ampere                          ||
||                             Sorting Phase-II                             ||
||                  Contact: Parabricks-Support@nvidia.com                  ||
------------------------------------------------------------------------------
progressMeter - Percentage
[10:33:32]      0.0      0.00 GB
[10:33:42]      3.7      1.61 GB
[10:33:52]      9.1      1.54 GB
[10:34:02]      14.2     1.58 GB
[10:34:12]      18.4     1.63 GB
[10:34:22]      24.2     1.64 GB
[10:34:32]      28.3     1.63 GB
[10:34:42]      33.3     1.57 GB
[10:34:52]      38.9     1.54 GB
[10:35:02]      43.7     1.59 GB
[10:35:12]      49.1     1.50 GB
[10:35:22]      54.1     1.56 GB
[10:35:32]      59.5     1.56 GB
[10:35:42]      65.8     1.57 GB
[10:35:52]      69.7     1.31 GB
[10:36:02]      75.9     1.49 GB
[10:36:12]      82.0     1.45 GB
[10:36:22]      87.8     1.26 GB
[10:36:32]      93.2     0.99 GB
[10:36:42]      97.1     3.36 GB
[10:36:52]      99.9     0.39 GB
Sorting and Marking: 210.004 seconds
------------------------------------------------------------------------------
||        Program:                                  Sorting Phase-II        ||
||        Version:                                     v3.5.0_ampere        ||
||        Start Time:                       Thu Apr  8 10:33:32 2021        ||
||        End Time:                         Thu Apr  8 10:37:02 2021        ||
||        Total Time:                           3 minutes 30 seconds        ||
------------------------------------------------------------------------------
------------------------------------------------------------------------------
||                 Parabricks accelerated Genomics Pipeline                 ||
||                           Version v3.5.0_ampere                          ||
||                         Marking Duplicates, BQSR                         ||
||                  Contact: Parabricks-Support@nvidia.com                  ||
------------------------------------------------------------------------------
progressMeter - Percentage
[10:37:12]      0.0      8.38 GB
[10:37:22]      0.0      16.09 GB
[10:37:32]      0.0      23.90 GB
[10:37:42]      0.0      31.25 GB
[10:37:52]      0.0      38.83 GB
[10:38:02]      0.0      46.26 GB
[10:38:12]      0.0      53.81 GB
[10:38:22]      0.0      61.22 GB
[10:38:32]      2.2      64.58 GB
[10:38:42]      4.9      67.89 GB
[10:38:52]      7.0      71.78 GB
[10:39:02]      9.2      74.52 GB
[10:39:12]      11.3     77.94 GB
[10:39:22]      13.4     81.08 GB
[10:39:32]      15.2     84.49 GB
[10:39:42]      17.4     87.45 GB
[10:39:52]      19.8     90.24 GB
[10:40:02]      21.7     93.60 GB
[10:40:12]      23.8     97.07 GB
[10:40:22]      25.8     100.01 GB
[10:40:32]      27.7     103.11 GB
[10:40:42]      30.0     107.14 GB
[10:40:52]      32.1     111.08 GB
[10:41:02]      34.3     109.45 GB
[10:41:12]      36.5     105.75 GB
[10:41:22]      38.4     102.23 GB
[10:41:32]      40.9     98.19 GB
[10:41:42]      43.0     94.57 GB
[10:41:52]      45.2     90.81 GB
[10:42:02]      47.2     87.24 GB
[10:42:12]      50.3     82.86 GB
[10:42:22]      52.6     79.06 GB
[10:42:32]      55.1     75.05 GB
[10:42:42]      57.2     71.40 GB
[10:42:52]      59.5     67.70 GB
[10:43:02]      61.7     63.93 GB
[10:43:12]      63.7     60.44 GB
[10:43:22]      67.6     54.64 GB
[10:43:32]      73.9     45.77 GB
[10:43:42]      79.4     36.88 GB
[10:43:52]      84.7     28.33 GB
[10:44:02]      91.2     19.12 GB
[10:44:12]      97.5     8.77 GB
[10:44:22]      100.0    0.00 GB
BQSR and writing final BAM:  440.061 seconds
------------------------------------------------------------------------------
||        Program:                          Marking Duplicates, BQSR        ||
||        Version:                                     v3.5.0_ampere        ||
||        Start Time:                       Thu Apr  8 10:37:02 2021        ||
||        End Time:                         Thu Apr  8 10:44:22 2021        ||
||        Total Time:                           7 minutes 20 seconds        ||
------------------------------------------------------------------------------
Please visit https://docs.nvidia.com/clara/#parabricks for detailed documentation

------------------------------------------------------------------------------
||                 Parabricks accelerated Genomics Pipeline                 ||
||                           Version v3.5.0_ampere                          ||
||                         GPU-GATK4 HaplotypeCaller                        ||
||                  Contact: Parabricks-Support@nvidia.com                  ||
------------------------------------------------------------------------------
0 /scale_wlg_persistent/filesets/project/nesi03181/PBDATA/output.bam /scale_wlg_persistent/filesets/project/nesi03181/PBDATA/output.vcf
ProgressMeter - Current-Locus   Elapsed-Minutes Regions-Processed       Regions/Minute
[10:44:54]      1:8481546       0.2     58703   352218
[10:45:04]      1:17361526      0.3     120569  361707
[10:45:14]      1:25343970      0.5     179247  358494
[10:45:24]      1:34871960      0.7     245920  368880
[10:45:34]      1:44035172      0.8     310964  373156
[10:45:44]      1:51906978      1.0     365155  365155
[10:45:54]      1:59073468      1.2     416626  357108
[10:46:04]      1:66057529      1.3     466549  349911
[10:46:14]      1:73348692      1.5     517903  345268
[10:46:24]      1:80591855      1.7     570160  342096
[10:46:34]      1:87489500      1.8     619878  338115
[10:46:44]      1:94900741      2.0     672524  336262
[10:46:54]      1:102100733     2.2     724088  334194
[10:47:04]      1:108710367     2.3     772114  330906
[10:47:14]      1:116409574     2.5     825605  330242
[10:47:24]      1:145166294     2.7     874286  327857
[10:47:34]      1:152092588     2.8     917290  323749
[10:47:44]      1:158750324     3.0     964306  321435
[10:47:54]      1:164625600     3.2     1006980 317993
[10:48:04]      1:170894380     3.3     1051696 315508
[10:48:14]      1:177537560     3.5     1097953 313700
[10:48:24]      1:183950359     3.7     1143483 311859
[10:48:34]      1:190142166     3.8     1187799 309860
[10:48:44]      1:196358377     4.0     1232169 308042
[10:48:54]      1:202449459     4.2     1276623 306389
[10:49:04]      1:209347031     4.3     1324221 305589
[10:49:14]      1:215774396     4.5     1369760 304391
[10:49:24]      1:222086263     4.7     1414911 303195
[10:49:34]      1:228585586     4.8     1461179 302312
[10:49:44]      1:234791942     5.0     1506089 301217
[10:49:54]      1:241055868     5.2     1551536 300297
[10:50:04]      1:247439854     5.3     1598600 299737
[10:50:14]      2:4958285       5.5     1647075 299468
[10:50:24]      2:11539120      5.7     1694402 299012
[10:50:34]      2:18153550      5.8     1741727 298581
[10:50:44]      2:25036687      6.0     1790899 298483
[10:50:54]      2:32169528      6.2     1841467 298616
[10:51:04]      2:38323127      6.3     1887069 297958
[10:51:14]      2:44745530      6.5     1933621 297480
[10:51:24]      2:51187022      6.7     1980587 297088
[10:51:34]      2:57595200      6.8     2027076 296645
[10:51:44]      2:64775872      7.0     2076821 296688
[10:51:54]      2:71375859      7.2     2124073 296382
[10:52:04]      2:78345562      7.3     2173242 296351
[10:52:14]      2:85281377      7.5     2222017 296268
[10:52:24]      2:97699133      7.7     2275713 296832
[10:52:34]      2:104111742     7.8     2319640 296124
[10:52:44]      2:109770945     8.0     2360510 295063
[10:52:54]      2:117638365     8.2     2413121 295484
[10:53:04]      2:124406388     8.3     2461930 295431
[10:53:14]      2:131049533     8.5     2510511 295354
[10:53:24]      2:138316801     8.7     2562064 295622
[10:53:34]      2:144772723     8.8     2608866 295343
[10:53:44]      2:151809601     9.0     2657803 295311
[10:53:54]      2:158486309     9.2     2705362 295130
[10:54:04]      2:165326217     9.3     2753408 295008
[10:54:14]      2:171527929     9.5     2799343 294667
[10:54:24]      2:178276766     9.7     2847187 294536
[10:54:34]      2:184301832     9.8     2890961 293996
[10:54:44]      2:190718388     10.0    2936783 293678
[10:54:54]      2:196996799     10.2    2981829 293294
[10:55:04]      2:203734339     10.3    3028990 293128
[10:55:14]      2:210614367     10.5    3077126 293059
[10:55:24]      2:216777361     10.7    3121862 292674
[10:55:34]      2:223238175     10.8    3168229 292451
[10:55:44]      2:229545501     11.0    3213524 292138
[10:55:54]      2:236030293     11.2    3259955 291936
[10:56:04]      2:242567973     11.3    3307353 291825
[10:56:14]      3:4895956       11.5    3348424 291167
[10:56:24]      3:10895818      11.7    3392407 290777
[10:56:34]      3:17289535      11.8    3438604 290586
[10:56:44]      3:23783931      12.0    3484334 290361
[10:56:54]      3:30273587      12.2    3530632 290188
[10:57:04]      3:36638196      12.3    3576234 289964
[10:57:14]      3:43641430      12.5    3625011 290000
[10:57:24]      3:51364710      12.7    3678655 290420
[10:57:34]      3:58171157      12.8    3726969 290413
[10:57:44]      3:64631739      13.0    3773849 290296
[10:57:54]      3:70713575      13.2    3817771 289957
[10:58:04]      3:76982249      13.3    3863556 289766
[10:58:14]      3:83452776      13.5    3909629 289602
[10:58:24]      3:90163148      13.7    3956732 289516
[10:58:34]      3:99498915      13.8    4001879 289292
[10:58:44]      3:105398218     14.0    4044282 288877
[10:58:54]      3:111700534     14.2    4089392 288662
[10:59:04]      3:118060594     14.3    4135160 288499
[10:59:14]      3:124578998     14.5    4181586 288385
[10:59:24]      3:131395153     14.7    4229949 288405
[10:59:34]      3:137548574     14.8    4274187 288147
[10:59:44]      3:143918387     15.0    4319299 287953
[10:59:54]      3:149995133     15.2    4362895 287663
[11:00:04]      3:156738933     15.3    4410039 287611
[11:00:14]      3:163425536     15.5    4457119 287556
[11:00:24]      3:170174293     15.7    4504849 287543
[11:00:34]      3:176540339     15.8    4550284 287386
[11:00:44]      3:183523168     16.0    4598488 287405
[11:00:54]      3:189767947     16.2    4643772 287243
[11:01:04]      3:195556701     16.3    4686551 286931
[11:01:14]      4:4223825       16.5    4734074 286913
[11:01:24]      4:10506925      16.7    4780970 286858
[11:01:34]      4:16195182      16.8    4823048 286517
[11:01:44]      4:22151893      17.0    4866489 286264
[11:01:54]      4:28099189      17.2    4909634 285998
[11:02:04]      4:34070135      17.3    4952873 285742
[11:02:14]      4:39681587      17.5    4994449 285397
[11:02:24]      4:45633490      17.7    5037945 285166
[11:02:34]      4:54566337      17.8    5079784 284847
[11:02:44]      4:60503913      18.0    5123053 284614
[11:02:54]      4:65980785      18.2    5164798 284300
[11:03:04]      4:71711872      18.3    5206488 283990
[11:03:14]      4:77769460      18.5    5250352 283802
[11:03:24]      4:83985483      18.7    5294970 283659
[11:03:34]      4:90398374      18.8    5340104 283545
[11:03:44]      4:96868695      19.0    5386393 283494
[11:03:54]      4:103823898     19.2    5433603 283492
[11:04:04]      4:110591626     19.3    5480950 283497
[11:04:14]      4:116523443     19.5    5524662 283316
[11:04:24]      4:122419071     19.7    5568052 283121
[11:04:34]      4:128212723     19.8    5610781 282896
[11:04:44]      4:134299163     20.0    5654237 282711
[11:04:54]      4:139958362     20.2    5695953 282443
[11:05:04]      4:145295983     20.3    5734330 282016
[11:05:14]      4:151943771     20.5    5780345 281968
[11:05:24]      4:158155048     20.7    5824521 281831
[11:05:34]      4:163833492     20.8    5866698 281601
[11:05:44]      4:169641547     21.0    5909235 281392
[11:05:54]      4:175377526     21.2    5950728 281136
[11:06:04]      4:180719871     21.3    5990949 280825
[11:06:14]      4:186278196     21.5    6033083 280608
[11:06:24]      5:1114982       21.7    6077197 280486
[11:06:34]      5:7156509       21.8    6122046 280399
[11:06:44]      5:13319974      22.0    6166864 280312
[11:06:54]      5:19166399      22.2    6209886 280145
[11:07:04]      5:24676735      22.3    6250811 279887
[11:07:14]      5:30282952      22.5    6292027 279645
[11:07:24]      5:36355180      22.7    6335833 279522
[11:07:34]      5:42527841      22.8    6379377 279388
[11:07:44]      5:52329479      23.0    6426772 279424
[11:07:54]      5:58295855      23.2    6470249 279291
[11:08:04]      5:64938956      23.3    6515748 279246
[11:08:14]      5:73247940      23.5    6570169 279581
[11:08:24]      5:79492773      23.7    6615525 279529
[11:08:34]      5:85847743      23.8    6660203 279449
[11:08:44]      5:92779004      24.0    6706693 279445
[11:08:54]      5:99427181      24.2    6752680 279421
[11:09:04]      5:105916720     24.3    6797517 279350
[11:09:14]      5:111921555     24.5    6840396 279199
[11:09:24]      5:117647754     24.7    6881763 278990
[11:09:34]      5:123945355     24.8    6925776 278890
[11:09:44]      5:129782306     25.0    6967987 278719
[11:09:54]      5:136300773     25.2    7013961 278700
[11:10:04]      5:143563169     25.3    7063407 278818
[11:10:14]      5:149779031     25.5    7107786 278736
[11:10:24]      5:156167977     25.7    7152908 278684
[11:10:34]      5:162388610     25.8    7196880 278588
[11:10:44]      5:168268765     26.0    7239504 278442
[11:10:54]      5:174311979     26.2    7283114 278335
[11:11:04]      6:374224        26.3    7332324 278442
[11:11:14]      6:6297317       26.5    7375652 278326
[11:11:24]      6:12052509      26.7    7418078 278177
[11:11:34]      6:18330811      26.8    7462890 278120
[11:11:44]      6:24321521      27.0    7507357 278050
[11:11:54]      6:30407978      27.2    7551842 277981
[11:12:04]      6:34756947      27.3    7585007 277500
[11:12:14]      6:40483158      27.5    7625916 277306
[11:12:24]      6:46626991      27.7    7669725 277218
[11:12:34]      6:52372578      27.8    7711125 277046
[11:12:44]      6:57715126      28.0    7750763 276812
[11:12:54]      6:66033596      28.2    7787821 276490
[11:13:04]      6:71707099      28.3    7830221 276360
[11:13:14]      6:77731181      28.5    7873576 276265
[11:13:24]      6:82982361      28.7    7912733 276025
[11:13:34]      6:88886137      28.8    7954932 275893
[11:13:44]      6:94665423      29.0    7996668 275747
[11:13:54]      6:100598339     29.2    8038420 275602
[11:14:04]      6:106502270     29.3    8080147 275459
[11:14:14]      6:112727992     29.5    8124357 275401
[11:14:24]      6:118627118     29.7    8166473 275274
[11:14:34]      6:124123025     29.8    8206620 275082
[11:14:44]      6:130247976     30.0    8248713 274957
[11:14:54]      6:135974354     30.2    8289617 274793
[11:15:04]      6:142074995     30.3    8332342 274692
[11:15:14]      6:148019117     30.5    8373609 274544
[11:15:24]      6:153460669     30.7    8413795 274362
[11:15:34]      6:159201560     30.8    8454764 274208
[11:15:44]      6:164611109     31.0    8494761 274024
[11:15:54]      6:170006335     31.2    8535193 273856
[11:16:04]      7:5452719       31.3    8582720 273916
[11:16:14]      7:11222238      31.5    8625436 273823
[11:16:24]      7:16435140      31.7    8664447 273614
[11:16:34]      7:21830380      31.8    8703733 273415
[11:16:44]      7:27014299      32.0    8741949 273185
[11:16:54]      7:32831952      32.2    8783465 273061
[11:17:04]      7:38294229      32.3    8823099 272879
[11:17:14]      7:44015965      32.5    8863371 272719
[11:17:24]      7:49627026      32.7    8904490 272586
[11:17:34]      7:55310330      32.8    8945833 272461
[11:17:44]      7:65260715      33.0    8994816 272570
[11:17:54]      7:71447906      33.2    9040502 272577
[11:18:04]      7:79142349      33.3    9093200 272796
[11:18:14]      7:84767929      33.5    9133325 272636
[11:18:24]      7:90407947      33.7    9173410 272477
[11:18:34]      7:96124484      33.8    9213345 272315
[11:18:44]      7:102811055     34.0    9260263 272360
[11:18:54]      7:108302393     34.2    9300214 272201
[11:19:04]      7:114609535     34.3    9344229 272162
[11:19:14]      7:121123107     34.5    9388766 272138
[11:19:24]      7:127257536     34.7    9432218 272083
[11:19:34]      7:133665597     34.8    9477305 272075
[11:19:44]      7:139540726     35.0    9519838 271995
[11:19:54]      7:146735862     35.2    9568424 272087
[11:20:04]      7:152774329     35.3    9612979 272065
[11:20:14]      7:158659058     35.5    9656542 272015
[11:20:24]      8:4401531       35.7    9695334 271831
[11:20:34]      8:9379111       35.8    9734085 271648
[11:20:44]      8:14644564      36.0    9774584 271516
[11:20:54]      8:19444776      36.2    9811652 271289
[11:21:04]      8:25420794      36.3    9854123 271214
[11:21:14]      8:31065535      36.5    9895320 271104
[11:21:24]      8:37329478      36.7    9939111 271066
[11:21:34]      8:43838196      36.8    9984753 271079
[11:21:44]      8:53284544      37.0    10030002        271081
[11:21:54]      8:59423720      37.2    10073699        271041
[11:22:04]      8:65385001      37.3    10116000        270964
[11:22:14]      8:72057519      37.5    10161450        270972
[11:22:24]      8:78235073      37.7    10205185        270934
[11:22:34]      8:84119939      37.8    10246806        270840
[11:22:44]      8:90494327      38.0    10290555        270804
[11:22:54]      8:96619064      38.2    10333405        270744
[11:23:04]      8:103199999     38.3    10378542        270744
[11:23:14]      8:109468730     38.5    10422419        270712
[11:23:24]      8:115538824     38.7    10465601        270662
[11:23:34]      8:121579933     38.8    10508259        270598
[11:23:44]      8:127747145     39.0    10551944        270562
[11:23:54]      8:133459149     39.2    10592894        270456
[11:24:04]      8:138974276     39.3    10633438        270341
[11:24:14]      8:144931042     39.5    10677290        270311
[11:24:24]      9:4511826       39.7    10719923        270250
[11:24:34]      9:9911954       39.8    10759868        270122
[11:24:44]      9:15239996      40.0    10800143        270003
[11:24:54]      9:20582310      40.2    10840070        269877
[11:25:04]      9:26481600      40.3    10882166        269805
[11:25:14]      9:31795150      40.5    10921299        269661
[11:25:24]      9:38087708      40.7    10965199        269636
[11:25:34]      9:69963005      40.8    11034995        270244
[11:25:44]      9:75873535      41.0    11073831        270093
[11:25:54]      9:81633356      41.2    11114937        269998
[11:26:04]      9:87609326      41.3    11157227        269932
[11:26:14]      9:94295811      41.5    11202986        269951
[11:26:24]      9:100689344     41.7    11249177        269980
[11:26:34]      9:106646393     41.8    11292292        269935
[11:26:44]      9:112833495     42.0    11336246        269910
[11:26:54]      9:118727951     42.2    11379405        269867
[11:27:04]      9:124723098     42.3    11422127        269814
[11:27:14]      9:131457590     42.5    11469205        269863
[11:27:24]      9:138182246     42.7    11517005        269929
[11:27:34]      10:3446215      42.8    11563441        269963
[11:27:44]      10:8639909      43.0    11602620        269828
[11:27:54]      10:13886348     43.2    11641797        269694
[11:28:04]      10:19723103     43.3    11684079        269632
[11:28:14]      10:26207956     43.5    11729097        269634
[11:28:24]      10:31756661     43.7    11770187        269546
[11:28:34]      10:37857572     43.8    11813514        269509
[11:28:44]      10:48849516     44.0    11863181        269617
[11:28:54]      10:55583951     44.2    11909530        269649
[11:29:04]      10:61084760     44.3    11949966        269548
[11:29:14]      10:66964763     44.5    11992174        269487
[11:29:24]      10:72676746     44.7    12034654        269432
[11:29:34]      10:79117646     44.8    12080009        269442
[11:29:44]      10:85559970     45.0    12126023        269467
[11:29:54]      10:91641585     45.2    12169707        269440
[11:30:04]      10:97814385     45.3    12213301        269411
[11:30:14]      10:104548801    45.5    12260923        269470
[11:30:24]      10:111206253    45.7    12306681        269489
[11:30:34]      10:117427109    45.8    12350860        269473
[11:30:44]      10:123004701    46.0    12392148        269394
[11:30:54]      10:128951990    46.2    12434266        269334
[11:31:04]      10:134548777    46.3    12475094        269246
[11:31:14]      11:5726375      46.5    12524435        269342
[11:31:24]      11:10876550     46.7    12561762        269180
[11:31:34]      11:15533438     46.8    12595600        268945
[11:31:44]      11:20294278     47.0    12630266        268729
[11:31:54]      11:24379120     47.2    12661316        268437
[11:32:04]      11:29102320     47.3    12695816        268221
[11:32:14]      11:35044777     47.5    12738098        268170
[11:32:24]      11:40694145     47.7    12779234        268095
[11:32:34]      11:46706422     47.8    12822175        268059
[11:32:44]      11:56102305     48.0    12867243        268067
[11:32:54]      11:62327770     48.2    12911570        268060
[11:33:04]      11:69511017     48.3    12962516        268189
[11:33:14]      11:76324726     48.5    13009909        268245
[11:33:24]      11:82348703     48.7    13053006        268212
[11:33:34]      11:88372789     48.8    13095259        268162
[11:33:44]      11:94377425     49.0    13138394        268130
[11:33:54]      11:99993585     49.2    13178477        268036
[11:34:04]      11:105105550    49.3    13216810        267908
[11:34:14]      11:111143964    49.5    13259465        267867
[11:34:24]      11:117259061    49.7    13302897        267843
[11:34:34]      11:122975953    49.8    13344574        267784
[11:34:44]      11:128471910    50.0    13383992        267679
[11:34:54]      11:134209973    50.2    13424928        267606
[11:35:04]      12:4876580      50.3    13465554        267527
[11:35:14]      12:10670339     50.5    13508385        267492
[11:35:24]      12:16343906     50.7    13549570        267425
[11:35:34]      12:21719719     50.8    13589464        267333
[11:35:44]      12:27287718     51.0    13629604        267247
[11:35:54]      12:32467156     51.2    13668592        267138
[11:36:04]      12:41793582     51.3    13713702        267150
[11:36:14]      12:47865513     51.5    13757166        267129
[11:36:24]      12:54383879     51.7    13803926        267172
[11:36:34]      12:61060724     51.8    13850863        267219
[11:36:44]      12:66988776     52.0    13893674        267186
[11:36:54]      12:73003003     52.2    13935666        267137
[11:37:04]      12:78383940     52.3    13974482        267028
[11:37:14]      12:83798158     52.5    14012734        266909
[11:37:24]      12:89692759     52.7    14054025        266848
[11:37:34]      12:95145553     52.8    14093415        266752
[11:37:44]      12:100967940    53.0    14134605        266690
[11:37:54]      12:106396649    53.2    14174322        266601
[11:38:04]      12:112929530    53.3    14219360        266613
[11:38:14]      12:118780800    53.5    14261871        266577
[11:38:24]      12:125226991    53.7    14308008        266608
[11:38:34]      12:130425420    53.8    14347145        266510
[11:38:44]      13:21441554     54.0    14389819        266478
[11:38:54]      13:26447832     54.2    14427834        266360
[11:39:04]      13:31833541     54.3    14467671        266276
[11:39:14]      13:37627114     54.5    14508965        266219
[11:39:24]      13:43228706     54.7    14550534        266168
[11:39:34]      13:49171109     54.8    14593123        266135
[11:39:44]      13:55843171     55.0    14639248        266168
[11:39:54]      13:61972675     55.2    14682947        266156
[11:40:04]      13:68039887     55.3    14726125        266134
[11:40:14]      13:73569578     55.5    14766537        266063
[11:40:24]      13:79147063     55.7    14807303        265999
[11:40:34]      13:84887706     55.8    14848716        265947
[11:40:44]      13:90547158     56.0    14889444        265882
[11:40:54]      13:96331115     56.2    14931644        265845
[11:41:04]      13:102662362    56.3    14976357        265852
[11:41:14]      13:108244653    56.5    15017679        265799
[11:41:24]      13:113822359    56.7    15058521        265738
[11:41:34]      14:24057359     56.8    15103538        265751
[11:41:44]      14:29951958     57.0    15145827        265716
[11:41:54]      14:35726332     57.2    15187965        265678
[11:42:04]      14:41332747     57.3    15229312        265627
[11:42:14]      14:47198260     57.5    15271260        265587
[11:42:24]      14:52857544     57.7    15312663        265537
[11:42:34]      14:58737536     57.8    15354921        265502
[11:42:44]      14:65001326     58.0    15399072        265501
[11:42:54]      14:71390352     58.2    15444481        265521
[11:43:04]      14:77423813     58.3    15488069        265509
[11:43:14]      14:83346969     58.5    15530854        265484
[11:43:24]      14:89193556     58.7    15572882        265446
[11:43:34]      14:95155075     58.8    15616479        265435
[11:43:44]      14:101169555    59.0    15660067        265424
[11:43:54]      15:20203148     59.2    15706153        265456
[11:44:04]      15:26255993     59.3    15745708        265377
[11:44:14]      15:33657425     59.5    15796708        265490
[11:44:24]      15:39417386     59.7    15838697        265453
[11:44:34]      15:46319780     59.8    15886448        265511
[11:44:44]      15:52444790     60.0    15929682        265494
[11:44:54]      15:58017457     60.2    15970726        265441
[11:45:04]      15:63374219     60.3    16010282        265363
[11:45:14]      15:69388753     60.5    16053567        265348
[11:45:24]      15:76103934     60.7    16100403        265391
[11:45:34]      15:82127976     60.8    16143997        265380
[11:45:44]      15:89274957     61.0    16192471        265450
[11:45:54]      15:95083039     61.2    16234850        265419
[11:46:04]      15:100502331    61.3    16274732        265348
[11:46:14]      16:4660680      61.5    16323197        265417
[11:46:24]      16:9537547      61.7    16361992        265329
[11:46:34]      16:15796606     61.8    16406819        265339
[11:46:44]      16:23524645     62.0    16460232        265487
[11:46:54]      16:30921562     62.2    16511540        265601
[11:47:04]      16:46742223     62.3    16542099        265381
[11:47:14]      16:47092680     62.5    16544529        264712
[11:47:24]      16:48023847     62.7    16550236        264099
[11:47:34]      16:51023901     62.8    16571450        263736
[11:47:44]      16:55627134     63.0    16605405        263577
[11:47:54]      16:61070384     63.2    16645032        263509
[11:48:04]      16:66633501     63.3    16683981        263431
[11:48:14]      16:73089531     63.5    16728528        263441
[11:48:24]      16:77947199     63.7    16764637        263318
[11:48:34]      16:82243181     63.8    16797785        263150
[11:48:44]      16:86683081     64.0    16832567        263008
[11:48:54]      17:2116785      64.2    16875257        262991
[11:49:04]      17:7934333      64.3    16918335        262979
[11:49:14]      17:13291126     64.5    16957924        262913
[11:49:24]      17:19190296     64.7    16999140        262873
[11:49:34]      17:28785522     64.8    17043670        262884
[11:49:44]      17:34646312     65.0    17085154        262848
[11:49:54]      17:41183991     65.2    17130506        262872
[11:50:04]      17:48220748     65.3    17179033        262944
[11:50:14]      17:54191872     65.5    17221383        262921
[11:50:24]      17:60854376     65.7    17267187        262952
[11:50:34]      17:66839810     65.8    17309194        262924
[11:50:44]      17:72259053     66.0    17347643        262843
[11:50:54]      17:78647762     66.2    17393994        262881
[11:51:04]      18:3153569      66.3    17436180        262856
[11:51:14]      18:8798337      66.5    17476669        262807
[11:51:24]      18:14246358     66.7    17517252        262758
[11:51:34]      18:23740754     66.8    17562354        262778
[11:51:44]      18:29323112     67.0    17602671        262726
[11:51:54]      18:35620746     67.2    17646364        262725
[11:52:04]      18:41275012     67.3    17686891        262676
[11:52:14]      18:47140792     67.5    17728040        262637
[11:52:24]      18:52545488     67.7    17766686        262561
[11:52:34]      18:57969470     67.8    17805529        262489
[11:52:44]      18:63359850     68.0    17844031        262412
[11:52:54]      18:68462345     68.2    17882036        262328
[11:53:04]      18:73530929     68.3    17919807        262241
[11:53:14]      19:974165       68.5    17960013        262189
[11:53:24]      19:7617600      68.7    18009437        262273
[11:53:34]      19:14433516     68.8    18057656        262338
[11:53:44]      19:20942143     69.0    18104923        262390
[11:53:54]      19:29217598     69.2    18144071        262323
[11:54:04]      19:35078389     69.3    18186738        262308
[11:54:14]      19:41347199     69.5    18232061        262331
[11:54:24]      19:47663961     69.7    18277998        262363
[11:54:34]      19:53995042     69.8    18324922        262409
[11:54:44]      20:844761       70.0    18368765        262410
[11:54:54]      20:6340631      70.2    18409092        262362
[11:55:04]      20:11759812     70.3    18448076        262294
[11:55:14]      20:16895853     70.5    18485371        262203
[11:55:24]      20:22199727     70.7    18523721        262128
[11:55:34]      20:31665403     70.8    18567442        262128
[11:55:44]      20:38524702     71.0    18614901        262181
[11:55:54]      20:44678145     71.2    18658123        262175
[11:56:04]      20:50697471     71.3    18702095        262178
[11:56:14]      20:56447949     71.5    18743714        262149
[11:56:24]      20:62361422     71.7    18786126        262131
[11:56:34]      21:14831945     71.8    18806341        261805
[11:56:44]      21:19636730     72.0    18841184        261683
[11:56:54]      21:24249483     72.2    18875838        261558
[11:57:04]      21:29020736     72.3    18910965        261441
[11:57:14]      21:34161546     72.5    18947874        261349
[11:57:24]      21:39119954     72.7    18984064        261248
[11:57:34]      21:44222337     72.8    19021512        261164
[11:57:44]      22:18119991     73.0    19063588        261145
[11:57:54]      22:24724567     73.2    19110393        261189
[11:58:04]      22:30782209     73.3    19153487        261183
[11:58:14]      22:36575849     73.5    19194835        261154
[11:58:24]      22:42767876     73.7    19238906        261161
[11:58:34]      22:48364543     73.8    19280977        261141
[11:58:44]      X:2803084       74.0    19322139        261109
[11:58:54]      X:9110242       74.2    19365460        261107
[11:59:04]      X:15791901      74.3    19409613        261115
[11:59:14]      X:23063945      74.5    19456733        261164
[11:59:24]      X:29227045      74.7    19499224        261150
[11:59:34]      X:35044652      74.8    19539162        261102
[11:59:44]      X:41783706      75.0    19582684        261102
[11:59:54]      X:48628757      75.2    19628433        261132
[12:00:04]      X:56529420      75.3    19676655        261194
[12:00:14]      X:67003123      75.5    19723929        261244
[12:00:24]      X:74030360      75.7    19770510        261284
[12:00:34]      X:80299119      75.8    19812766        261267
[12:00:44]      X:86606191      76.0    19855799        261260
[12:00:54]      X:93273514      76.2    19901592        261290
[12:01:04]      X:99508539      76.3    19944108        261276
[12:01:14]      X:107553558     76.5    19995689        261381
[12:01:24]      X:114441554     76.7    20041081        261405
[12:01:34]      X:120950297     76.8    20085097        261411
[12:01:44]      X:127204693     77.0    20127936        261401
[12:01:54]      X:134572751     77.2    20175678        261455
[12:02:04]      X:141355142     77.3    20221791        261488
[12:02:14]      X:147566330     77.5    20264892        261482
[12:02:24]      X:154967965     77.7    20313110        261542
[12:02:34]      GL000209.1:38203        77.8    20355649        261528
[12:02:44]      GL000199.1:19177        78.0    20361203        261041
[12:02:54]      GL000215.1:4801 78.2    20365102        260534
[12:03:04]      GL000223.1:100630       78.3    20370591        260050
[12:03:14]      GL000225.1:23841        78.5    20378791        259602
[12:03:24]      hs37d5:2668742  78.7    20401919        259346
[12:03:34]      hs37d5:6743953  78.8    20428665        259137
[12:03:44]      hs37d5:11097512 79.0    20455711        258933
[12:03:54]      hs37d5:14198128 79.2    20476379        258648
[12:04:04]      hs37d5:16775948 79.3    20493602        258322
[12:04:14]      hs37d5:19089090 79.5    20509308        257978
[12:04:24]      hs37d5:19766377 79.7    20514707        257506
[12:04:34]      hs37d5:20236727 79.8    20518436        257015
[12:04:44]      hs37d5:21307015 80.0    20525898        256573
[12:04:54]      hs37d5:21950274 80.2    20531031        256104
[12:05:04]      hs37d5:23020768 80.3    20539966        255684
[12:05:14]      hs37d5:24475177 80.5    20550699        255288
[12:05:24]      hs37d5:26092751 80.7    20562364        254905
[12:05:34]      hs37d5:26519759 80.8    20565582        254419
[12:05:44]      hs37d5:27278273 81.0    20571708        253971
[12:05:54]      hs37d5:27911931 81.2    20576628        253510
[12:06:04]      hs37d5:28454383 81.3    20580884        253043
[12:06:14]      hs37d5:29284734 81.5    20587191        252603
[12:06:24]      hs37d5:29846373 81.7    20591592        252141
[12:06:34]      hs37d5:30335800 81.8    20595564        251676
[12:06:44]      hs37d5:35466926 82.0    20628317        251564
Total time taken: 4933.03
------------------------------------------------------------------------------
||        Program:                         GPU-GATK4 HaplotypeCaller        ||
||        Version:                                     v3.5.0_ampere        ||
||        Start Time:                       Thu Apr  8 10:44:27 2021        ||
||        End Time:                         Thu Apr  8 12:06:55 2021        ||
||        Total Time:                          82 minutes 28 seconds        ||
------------------------------------------------------------------------------
[lkemp@wbl009 nesi03181]$ srun: Force Terminated job 18958691
srun: Job step aborted: Waiting up to 32 seconds for job step to finish.
slurmstepd: error: *** STEP 18958691.0 ON wbl009 CANCELLED AT 2021-04-08T15:43:02 DUE TO TIME LIMIT ***
srun: error: wbl009: task 0: Killed
```

Total runtime:

- 201 minutes
- 3.35 hours

### A single whole genome on 2 GPUs (repeat to test for variance)

```bash
# Start interactive slurm session with 8 GPUS
srun --account nesi03181 --job-name pbtest --mem 200G --cpus-per-task 8 --gres=gpu:A100:2 --qos=testing --time 04:00:00 --pty bash

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

My output:

```bash
[lkemp@wbl004 nesi03181]$ ./SW/parabricks/pbrun germline \
> --ref /nesi/project/nesi03181/PBDATA/REFERENCE/human_g1k_v37_decoy.fasta \
> --in-fq /nesi/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R1_001.fastq.gz /nesi/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R2_001.fastq.gz \
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

GPU-BWA mem
ProgressMeter   Reads           Base Pairs Aligned
[22:21:11]      5040000         630000000
[22:22:33]      10080000        1270000000
[22:23:54]      15120000        1900000000
[22:25:12]      20160000        2520000000
[22:26:30]      25200000        3150000000
[22:27:47]      30240000        3780000000
[22:29:03]      35280000        4420000000
[22:30:20]      40320000        5040000000
[22:31:35]      45360000        5660000000
[22:32:47]      50400000        6310000000
[22:34:03]      55440000        6930000000
[22:35:16]      60480000        7580000000
[22:36:34]      65520000        8200000000
[22:37:49]      70560000        8810000000
[22:39:04]      75600000        9450000000
[22:40:17]      80640000        10080000000
[22:41:35]      85680000        10710000000
[22:42:52]      90720000        11340000000
[22:44:06]      95760000        11970000000
[22:45:19]      100800000       12610000000
[22:46:36]      105840000       13230000000
[22:47:50]      110880000       13870000000
[22:49:07]      115920000       14490000000
[22:50:21]      120960000       15110000000
[22:51:39]      126000000       15730000000
[22:52:52]      131040000       16380000000
[22:54:08]      136080000       17020000000
[22:55:22]      141120000       17640000000
[22:56:35]      146160000       18280000000
[22:57:50]      151200000       18900000000
[22:59:07]      156240000       19520000000
[23:00:24]      161280000       20170000000
[23:01:44]      166320000       20790000000
[23:02:58]      171360000       21420000000
[23:04:14]      176400000       22050000000
[23:05:31]      181440000       22680000000
[23:06:46]      186480000       23320000000
[23:08:03]      191520000       23940000000
[23:09:20]      196560000       24570000000
[23:10:37]      201600000       25200000000
[23:11:54]      206640000       25820000000
[23:13:11]      211680000       26460000000
[23:14:29]      216720000       27090000000
[23:15:52]      221760000       27720000000
[23:17:13]      226800000       28350000000
[23:18:30]      231840000       28970000000
[23:19:50]      236880000       29620000000
[23:21:07]      241920000       30240000000
[23:22:27]      246960000       30820000000
[23:23:42]      252000000       31500000000
[23:25:00]      257040000       32130000000
[23:26:18]      262080000       32760000000
[23:27:36]      267120000       33400000000
[23:28:52]      272160000       34010000000
[23:30:11]      277200000       34650000000
[23:31:26]      282240000       35280000000
[23:32:41]      287280000       35910000000
[23:33:56]      292320000       36540000000
[23:35:13]      297360000       37170000000
[23:36:28]      302400000       37810000000
[23:37:43]      307440000       38440000000
[23:38:59]      312480000       39050000000
[23:40:15]      317520000       39680000000
[23:41:30]      322560000       40340000000
[23:42:46]      327600000       40940000000
[23:44:01]      332640000       41580000000
[23:45:14]      337680000       42200000000
[23:46:28]      342720000       42840000000
[23:47:46]      347760000       43470000000
[23:49:03]      352800000       44100000000
[23:50:19]      357840000       44720000000
[23:51:34]      362880000       45360000000
[23:52:51]      367920000       45980000000
[23:54:08]      372960000       46620000000
[23:55:22]      378000000       47240000000
[23:56:36]      383040000       47880000000
[23:57:53]      388080000       48520000000
[23:59:07]      393120000       49150000000
[00:00:20]      398160000       49770000000
[00:01:35]      403200000       50400000000
[00:02:54]      408240000       51030000000
[00:04:11]      413280000       51660000000
[00:05:26]      418320000       52290000000

GPU-BWA Mem time: 6465.937493 seconds
GPU-BWA Mem is finished.

GPU Sorting, Marking Dups, BQSR
ProgressMeter   SAM Entries Completed

Total GPU-BWA Mem + Sorting + MarkingDups + BQSR Generation + BAM writing
Processing time: 6465.941282 seconds

[main] CMD: PARABRICKS mem -Z ./pbOpts.txt /scale_wlg_persistent/filesets/project/nesi03181/PBDATA/REFERENCE/human_g1k_v37_decoy.fasta /scale_wlg_persistent/filesets/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R1_001.fastq.gz /scale_wlg_persistent/filesets/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R2_001.fastq.gz @RG\tID:C6UP4ANXX.6\tLB:lib1\tPL:bar\tSM:sample\tPU:C6UP4ANXX.6
[main] Real time: 6469.877 sec; CPU: 51065.044 sec
------------------------------------------------------------------------------
||        Program:                      GPU-BWA mem, Sorting Phase-I        ||
||        Version:                                     v3.5.0_ampere        ||
||        Start Time:                       Thu Apr  8 22:19:40 2021        ||
||        End Time:                         Fri Apr  9 00:07:30 2021        ||
||        Total Time:                         107 minutes 50 seconds        ||
------------------------------------------------------------------------------
------------------------------------------------------------------------------
||                 Parabricks accelerated Genomics Pipeline                 ||
||                           Version v3.5.0_ampere                          ||
||                             Sorting Phase-II                             ||
||                  Contact: Parabricks-Support@nvidia.com                  ||
------------------------------------------------------------------------------
progressMeter - Percentage
[00:07:31]      0.0      0.00 GB
[00:07:41]      4.0      1.61 GB
[00:07:51]      9.3      1.54 GB
[00:08:01]      14.9     1.58 GB
[00:08:11]      20.4     1.55 GB
[00:08:21]      25.4     1.61 GB
[00:08:31]      30.3     1.61 GB
[00:08:41]      35.7     1.60 GB
[00:08:51]      41.1     1.58 GB
[00:09:01]      46.5     1.58 GB
[00:09:11]      52.9     1.52 GB
[00:09:21]      58.0     1.56 GB
[00:09:31]      62.7     1.31 GB
[00:09:41]      69.5     1.41 GB
[00:09:51]      75.8     1.48 GB
[00:10:01]      81.5     1.47 GB
[00:10:11]      87.6     1.26 GB
[00:10:21]      93.5     0.84 GB
[00:10:31]      97.2     3.24 GB
Sorting and Marking: 190.003 seconds
------------------------------------------------------------------------------
||        Program:                                  Sorting Phase-II        ||
||        Version:                                     v3.5.0_ampere        ||
||        Start Time:                       Fri Apr  9 00:07:31 2021        ||
||        End Time:                         Fri Apr  9 00:10:41 2021        ||
||        Total Time:                           3 minutes 10 seconds        ||
------------------------------------------------------------------------------
------------------------------------------------------------------------------
||                 Parabricks accelerated Genomics Pipeline                 ||
||                           Version v3.5.0_ampere                          ||
||                         Marking Duplicates, BQSR                         ||
||                  Contact: Parabricks-Support@nvidia.com                  ||
------------------------------------------------------------------------------
progressMeter - Percentage
[00:10:51]      0.0      6.61 GB
[00:11:01]      0.0      13.14 GB
[00:11:11]      0.0      19.31 GB
[00:11:21]      0.0      25.26 GB
[00:11:31]      0.0      31.43 GB
[00:11:41]      1.6      35.10 GB
[00:11:51]      4.6      37.30 GB
[00:12:01]      7.4      38.76 GB
[00:12:11]      9.8      40.72 GB
[00:12:21]      12.4     42.74 GB
[00:12:31]      14.8     44.48 GB
[00:12:41]      17.3     46.22 GB
[00:12:51]      20.0     47.92 GB
[00:13:01]      22.3     50.19 GB
[00:13:11]      24.6     52.46 GB
[00:13:21]      26.8     54.05 GB
[00:13:31]      29.2     56.23 GB
[00:13:41]      31.7     58.13 GB
[00:13:51]      34.4     59.29 GB
[00:14:01]      37.3     60.21 GB
[00:14:11]      39.6     62.62 GB
[00:14:21]      42.3     64.42 GB
[00:14:31]      44.8     65.83 GB
[00:14:41]      47.6     66.53 GB
[00:14:51]      51.0     67.81 GB
[00:15:01]      53.4     72.43 GB
[00:15:11]      55.9     73.51 GB
[00:15:21]      59.0     68.51 GB
[00:15:31]      61.9     63.40 GB
[00:15:41]      64.8     58.53 GB
[00:15:51]      68.5     53.09 GB
[00:16:01]      71.9     48.24 GB
[00:16:11]      75.3     43.41 GB
[00:16:21]      78.9     37.58 GB
[00:16:31]      81.7     33.05 GB
[00:16:41]      84.5     28.65 GB
[00:16:51]      87.8     23.82 GB
[00:17:01]      91.6     18.26 GB
[00:17:11]      94.5     13.32 GB
[00:17:21]      97.8     7.01 GB
[00:17:31]      100.0    0.00 GB
BQSR and writing final BAM:  410.035 seconds
------------------------------------------------------------------------------
||        Program:                          Marking Duplicates, BQSR        ||
||        Version:                                     v3.5.0_ampere        ||
||        Start Time:                       Fri Apr  9 00:10:41 2021        ||
||        End Time:                         Fri Apr  9 00:17:31 2021        ||
||        Total Time:                           6 minutes 50 seconds        ||
------------------------------------------------------------------------------
Please visit https://docs.nvidia.com/clara/#parabricks for detailed documentation

------------------------------------------------------------------------------
||                 Parabricks accelerated Genomics Pipeline                 ||
||                           Version v3.5.0_ampere                          ||
||                         GPU-GATK4 HaplotypeCaller                        ||
||                  Contact: Parabricks-Support@nvidia.com                  ||
------------------------------------------------------------------------------
0 /scale_wlg_persistent/filesets/project/nesi03181/PBDATA/output.bam /scale_wlg_persistent/filesets/project/nesi03181/PBDATA/output.vcf
ProgressMeter - Current-Locus   Elapsed-Minutes Regions-Processed       Regions/Minute
[00:18:00]      1:8956743       0.2     61984   371904
[00:18:10]      1:19089517      0.3     133550  400650
[00:18:20]      1:29634947      0.5     208831  417662
[00:18:30]      1:39364720      0.7     278173  417259
[00:18:40]      1:49626958      0.8     350250  420300
[00:18:50]      1:59563187      1.0     420157  420157
[00:19:00]      1:68807756      1.2     486589  417076
[00:19:10]      1:78643048      1.3     555814  416860
[00:19:20]      1:87427173      1.5     619388  412925
[00:19:30]      1:96647967      1.7     685024  411014
[00:19:40]      1:105441480     1.8     747743  407859
[00:19:50]      1:114647888     2.0     813202  406601
[00:20:00]      1:145075197     2.2     873458  403134
[00:20:10]      1:153763112     2.3     929150  398207
[00:20:20]      1:162916617     2.5     994607  397842
[00:20:30]      1:170471976     2.7     1048775 393290
[00:20:40]      1:175746905     2.8     1085602 383153
[00:20:50]      1:181247917     3.0     1124731 374910
[00:21:00]      1:187190312     3.2     1165602 368084
[00:21:10]      1:193507187     3.3     1211195 363358
[00:21:20]      1:199852605     3.5     1257442 359269
[00:21:30]      1:207009439     3.7     1307584 356613
[00:21:40]      1:213882917     3.8     1356622 353901
[00:21:50]      1:220631957     4.0     1404735 351183
[00:22:00]      1:227678258     4.2     1454732 349135
[00:22:10]      1:234436633     4.3     1503544 346971
[00:22:20]      1:241204758     4.5     1552596 345021
[00:22:30]      1:247996711     4.7     1602917 343482
[00:22:40]      2:5899065       4.8     1653295 342061
[00:22:50]      2:12479960      5.0     1701320 340264
[00:23:00]      2:19103846      5.2     1748640 338446
[00:23:10]      2:25938948      5.3     1797261 336986
[00:23:20]      2:33119836      5.5     1848221 336040
[00:23:30]      2:39230389      5.7     1893404 334130
[00:23:40]      2:45796638      5.8     1941074 332755
[00:23:50]      2:52127961      6.0     1987563 331260
[00:24:00]      2:58607734      6.2     2034018 329840
[00:24:10]      2:65707157      6.3     2083592 328988
[00:24:20]      2:72383854      6.5     2131236 327882
[00:24:30]      2:78527846      6.7     2174575 326186
[00:24:40]      2:85132779      6.8     2221024 325027
[00:24:50]      2:97238400      7.0     2272559 324651
[00:25:00]      2:104759845     7.2     2324156 324300
[00:25:10]      2:112468791     7.3     2376063 324008
[00:25:20]      2:119380780     7.5     2425469 323395
[00:25:30]      2:125764795     7.7     2471837 322413
[00:25:40]      2:131990381     7.8     2516706 321281
[00:25:50]      2:138311935     8.0     2562040 320255
[00:26:00]      2:144278325     8.2     2605448 319034
[00:26:10]      2:150614301     8.3     2648981 317877
[00:26:20]      2:156465545     8.5     2691266 316619
[00:26:30]      2:162594939     8.7     2734802 315554
[00:26:40]      2:168945343     8.8     2779832 314697
[00:26:50]      2:174642904     9.0     2822137 313570
[00:27:00]      2:181094365     9.2     2867740 312844
[00:27:10]      2:187276518     9.3     2912352 312037
[00:27:20]      2:193934362     9.5     2959675 311544
[00:27:30]      2:200894321     9.7     3009154 311291
[00:27:40]      2:208238326     9.8     3060657 311253
[00:27:50]      2:215126343     10.0    3109480 310948
[00:28:00]      2:221798292     10.2    3157912 310614
[00:28:10]      2:228479897     10.3    3205648 310224
[00:28:20]      2:235315183     10.5    3254631 309964
[00:28:30]      2:242049560     10.7    3303669 309718
[00:28:40]      3:4852772       10.8    3348084 309053
[00:28:50]      3:10876788      11.0    3392270 308388
[00:29:00]      3:16511975      11.2    3433406 307469
[00:29:10]      3:22487744      11.3    3475112 306627
[00:29:20]      3:28363158      11.5    3517115 305836
[00:29:30]      3:34511881      11.7    3560712 305203
[00:29:40]      3:40982309      11.8    3605834 304718
[00:29:50]      3:47347046      12.0    3650956 304246
[00:30:00]      3:55238339      12.2    3705958 304599
[00:30:10]      3:61857559      12.3    3753606 304346
[00:30:20]      3:68236594      12.5    3799641 303971
[00:30:30]      3:74572796      12.7    3845882 303622
[00:30:40]      3:80937513      12.8    3892054 303276
[00:30:50]      3:87705373      13.0    3939823 303063
[00:31:00]      3:97684757      13.2    3988435 302919
[00:31:10]      3:103924786     13.3    4033357 302501
[00:31:20]      3:110308614     13.5    4079234 302165
[00:31:30]      3:116687855     13.7    4125101 301836
[00:31:40]      3:123091193     13.8    4170841 301506
[00:31:50]      3:130276749     14.0    4222306 301593
[00:32:00]      3:136996602     14.2    4270163 301423
[00:32:10]      3:144345545     14.3    4322126 301543
[00:32:20]      3:151223943     14.5    4371738 301499
[00:32:30]      3:158419185     14.7    4422021 301501
[00:32:40]      3:165465456     14.8    4471283 301434
[00:32:50]      3:172310202     15.0    4519566 301304
[00:33:00]      3:178675021     15.2    4565446 301018
[00:33:10]      3:185999992     15.3    4616386 301068
[00:33:20]      3:192124593     15.5    4661119 300717
[00:33:30]      4:455722        15.7    4707020 300448
[00:33:40]      4:7358364       15.8    4758044 300508
[00:33:50]      4:13766232      16.0    4804919 300307
[00:34:00]      4:19939170      16.2    4850296 300018
[00:34:10]      4:26639938      16.3    4898837 299928
[00:34:20]      4:33124764      16.5    4946318 299776
[00:34:30]      4:38995173      16.7    4989188 299351
[00:34:40]      4:45119801      16.8    5034263 299065
[00:34:50]      4:54287870      17.0    5077596 298682
[00:35:00]      4:60503913      17.2    5123053 298430
[00:35:10]      4:66225388      17.3    5166606 298073
[00:35:20]      4:72417497      17.5    5211488 297799
[00:35:30]      4:78714942      17.7    5257063 297569
[00:35:40]      4:85243166      17.8    5303845 297411
[00:35:50]      4:91963190      18.0    5351428 297301
[00:36:00]      4:98817331      18.2    5399540 297222
[00:36:10]      4:105969560     18.3    5448233 297176
[00:36:20]      4:112511943     18.5    5495153 297035
[00:36:30]      4:118497471     18.7    5539108 296737
[00:36:40]      4:124723158     18.8    5584791 296537
[00:36:50]      4:131087871     19.0    5630955 296366
[00:37:00]      4:137236739     19.2    5675655 296121
[00:37:10]      4:143371192     19.3    5720840 295905
[00:37:20]      4:150393336     19.5    5769525 295873
[00:37:30]      4:157084667     19.7    5816815 295770
[00:37:40]      4:163156751     19.8    5861406 295533
[00:37:50]      4:169204642     20.0    5905886 295294
[00:38:00]      4:175295989     20.2    5950125 295047
[00:38:10]      4:180964778     20.3    5992910 294733
[00:38:20]      4:186724538     20.5    6036440 294460
[00:38:30]      5:1574377       20.7    6080580 294221
[00:38:40]      5:7751994       20.8    6126435 294068
[00:38:50]      5:13756751      21.0    6170120 293815
[00:39:00]      5:19742395      21.2    6214210 293584
[00:39:10]      5:25876734      21.3    6259607 293419
[00:39:20]      5:31919761      21.5    6303741 293197
[00:39:30]      5:38356695      21.7    6349746 293065
[00:39:40]      5:44846341      21.8    6395530 292925
[00:39:50]      5:54340739      22.0    6441910 292814
[00:40:00]      5:60815950      22.2    6487300 292660
[00:40:10]      5:67641453      22.3    6534447 292587
[00:40:20]      5:75854368      22.5    6588822 292836
[00:40:30]      5:82444595      22.7    6635883 292759
[00:40:40]      5:89092770      22.8    6681757 292631
[00:40:50]      5:96239965      23.0    6730005 292608
[00:41:00]      5:102724655     23.2    6775474 292466
[00:41:10]      5:109209380     23.3    6821076 292331
[00:41:20]      5:115132734     23.5    6864080 292088
[00:41:30]      5:121430331     23.7    6908283 291899
[00:41:40]      5:127660780     23.8    6952525 291714
[00:41:50]      5:134164757     24.0    6998703 291612
[00:42:00]      5:141619185     24.2    7049905 291720
[00:42:10]      5:147964719     24.3    7094999 291575
[00:42:20]      5:154339140     24.5    7140256 291439
[00:42:30]      5:161010937     24.7    7186762 291355
[00:42:40]      5:167164799     24.8    7231433 291198
[00:42:50]      5:173443198     25.0    7276685 291067
[00:43:00]      5:180436631     25.2    7327557 291161
[00:43:10]      6:5995030       25.3    7373288 291050
[00:43:20]      6:11865373      25.5    7416683 290850
[00:43:30]      6:18100686      25.7    7461288 290699
[00:43:40]      6:24148708      25.8    7506058 290557
[00:43:50]      6:30273409      26.0    7550797 290415
[00:44:00]      6:34895984      26.2    7586016 289911
[00:44:10]      6:41548777      26.3    7633396 289875
[00:44:20]      6:48167837      26.5    7680686 289837
[00:44:30]      6:54225538      26.7    7724895 289683
[00:44:40]      6:62409491      26.8    7761897 289263
[00:44:50]      6:67771192      27.0    7801656 288950
[00:45:00]      6:73123150      27.2    7840243 288597
[00:45:10]      6:78503990      27.3    7879450 288272
[00:45:20]      6:83529403      27.5    7916502 287872
[00:45:30]      6:89015734      27.7    7955826 287559
[00:45:40]      6:94377590      27.8    7994641 287232
[00:45:50]      6:99906963      28.0    8033419 286907
[00:46:00]      6:105326214     28.2    8072123 286584
[00:46:10]      6:111331148     28.3    8114465 286392
[00:46:20]      6:116956772     28.5    8154786 286132
[00:46:30]      6:122347155     28.7    8193755 285828
[00:46:40]      6:127991912     28.8    8233656 285560
[00:46:50]      6:133745696     29.0    8273993 285310
[00:47:00]      6:139502295     29.2    8314410 285065
[00:47:10]      6:145204681     29.3    8354575 284815
[00:47:20]      6:150887702     29.5    8394882 284572
[00:47:30]      6:156134279     29.7    8433280 284267
[00:47:40]      6:161419060     29.8    8471402 283957
[00:47:50]      6:166651121     30.0    8510051 283668
[00:48:00]      7:1151959       30.2    8550582 283444
[00:48:10]      7:7310176       30.3    8596487 283400
[00:48:20]      7:12258938      30.5    8633118 283053
[00:48:30]      7:17121394      30.7    8669625 282705
[00:48:40]      7:22209529      30.8    8706608 282376
[00:48:50]      7:27364768      31.0    8744214 282071
[00:49:00]      7:32971176      31.2    8784440 281853
[00:49:10]      7:38337552      31.3    8823425 281598
[00:49:20]      7:43910400      31.5    8862648 281353
[00:49:30]      7:49444597      31.7    8903063 281149
[00:49:40]      7:54513521      31.8    8940217 280844
[00:49:50]      7:64185377      32.0    8986881 280840
[00:50:00]      7:69998329      32.2    9029369 280705
[00:50:10]      7:77399783      32.3    9080285 280833
[00:50:20]      7:82910343      32.5    9120052 280616
[00:50:30]      7:88588800      32.7    9160228 280415
[00:50:40]      7:94238363      32.8    9199711 280194
[00:50:50]      7:100074951     33.0    9241391 280042
[00:51:00]      7:106089599     33.2    9284738 279941
[00:51:10]      7:111652770     33.3    9324004 279720
[00:51:20]      7:117739030     33.5    9365445 279565
[00:51:30]      7:123601810     33.7    9405884 279382
[00:51:40]      7:129215971     33.8    9446210 279198
[00:51:50]      7:134918134     34.0    9486645 279018
[00:52:00]      7:140467199     34.2    9526276 278817
[00:52:10]      7:146870233     34.3    9569390 278720
[00:52:20]      7:152279981     34.5    9609575 278538
[00:52:30]      7:157857323     34.7    9650299 278374
[00:52:40]      8:3748567       34.8    9689575 278169
[00:52:50]      8:7991821       35.0    9723252 277807
[00:53:00]      8:13392363      35.2    9764110 277652
[00:53:10]      8:17683201      35.3    9798459 277314
[00:53:20]      8:23135957      35.5    9837843 277122
[00:53:30]      8:28612762      35.7    9877769 276946
[00:53:40]      8:34199856      35.8    9917817 276776
[00:53:50]      8:40300772      36.0    9960043 276667
[00:54:00]      8:49552825      36.2    10003763        276601
[00:54:10]      8:55099047      36.3    10043384        276423
[00:54:20]      8:60859145      36.5    10083848        276269
[00:54:30]      8:66404986      36.7    10122970        276081
[00:54:40]      8:72398367      36.8    10163867        275942
[00:54:50]      8:77951968      37.0    10203088        275759
[00:55:00]      8:83535949      37.2    10242278        275576
[00:55:10]      8:89188542      37.3    10281345        275393
[00:55:20]      8:94972627      37.5    10321636        275243
[00:55:30]      8:100886271     37.7    10362570        275112
[00:55:40]      8:106804734     37.8    10403356        274978
[00:55:50]      8:112406304     38.0    10442697        274807
[00:56:00]      8:117668011     38.2    10480806        274606
[00:56:10]      8:123283176     38.3    10520630        274451
[00:56:20]      8:128932770     38.5    10560452        274297
[00:56:30]      8:134222140     38.7    10598453        274097
[00:56:40]      8:139190376     38.8    10635173        273867
[00:56:50]      8:144724569     39.0    10675740        273736
[00:57:00]      9:3959948       39.2    10715717        273592
[00:57:10]      9:8755184       39.3    10751363        273339
[00:57:20]      9:13516586      39.5    10787202        273093
[00:57:30]      9:18479963      39.7    10824349        272882
[00:57:40]      9:23659146      39.8    10862218        272691
[00:57:50]      9:29092787      40.0    10901367        272534
[00:58:00]      9:34291072      40.2    10938864        272336
[00:58:10]      9:44068788      40.3    10996826        272648
[00:58:20]      9:73223920      40.5    11055011        272963
[00:58:30]      9:78782333      40.7    11094764        272822
[00:58:40]      9:84158356      40.8    11133440        272655
[00:58:50]      9:90163084      41.0    11175138        272564
[00:59:00]      9:95855933      41.2    11214558        272418
[00:59:10]      9:101728559     41.3    11256799        272341
[00:59:20]      9:107073412     41.5    11295189        272173
[00:59:30]      9:112718228     41.7    11335395        272049
[00:59:40]      9:118137403     41.8    11375052        271913
[00:59:50]      9:123537540     42.0    11413426        271748
[01:00:00]      9:129460774     42.2    11455380        271669
[01:00:10]      9:135863823     42.3    11500284        271660
[01:00:20]      10:1459029      42.5    11547981        271717
[01:00:30]      10:6187153      42.7    11584225        271505
[01:00:40]      10:11294312     42.8    11622037        271331
[01:00:50]      10:16776001     43.0    11663050        271233
[01:01:00]      10:24014205     43.2    11713775        271361
[01:01:10]      10:30575840     43.3    11761795        271426
[01:01:20]      10:37109306     43.5    11808613        271462
[01:01:30]      10:48441511     43.7    11860627        271617
[01:01:40]      10:55871949     43.8    11911780        271751
[01:01:50]      10:61934333     44.0    11955909        271725
[01:02:00]      10:68039705     44.2    12000397        271707
[01:02:10]      10:74558391     44.3    12048142        271762
[01:02:20]      10:81724766     44.5    12098492        271876
[01:02:30]      10:88166327     44.7    12144673        271895
[01:02:40]      10:94766162     44.8    12191906        271938
[01:02:50]      10:101601574    45.0    12240692        272015
[01:03:00]      10:108782351    45.2    12290278        272109
[01:03:10]      10:115382283    45.3    12336867        272136
[01:03:20]      10:121387105    45.5    12379919        272086
[01:03:30]      10:127195107    45.7    12422341        272022
[01:03:40]      10:132930287    45.8    12463856        271938
[01:03:50]      11:3921532      46.0    12510747        271972
[01:04:00]      11:8783885      46.2    12547086        271778
[01:04:10]      11:14188627     46.3    12586335        271647
[01:04:20]      11:20164679     46.5    12629309        271598
[01:04:30]      11:25521594     46.7    12670257        271505
[01:04:40]      11:32001440     46.8    12715708        271509
[01:04:50]      11:38366395     47.0    12761524        271521
[01:05:00]      11:44414202     47.2    12806243        271510
[01:05:10]      11:51451113     47.3    12855788        271601
[01:05:20]      11:61180551     47.5    12903369        271649
[01:05:30]      11:69019071     47.7    12959214        271871
[01:05:40]      11:76430281     47.8    13010763        272002
[01:05:50]      11:82804758     48.0    13056087        272001
[01:06:00]      11:89011110     48.2    13099931        271970
[01:06:10]      11:95503080     48.3    13146542        271997
[01:06:20]      11:101548612    48.5    13190082        271960
[01:06:30]      11:107332773    48.7    13232823        271907
[01:06:40]      11:113990325    48.8    13279361        271932
[01:06:50]      11:120167907    49.0    13324700        271932
[01:07:00]      11:126426965    49.2    13369518        271922
[01:07:10]      11:132868711    49.3    13415386        271933
[01:07:20]      12:4444705      49.5    13462447        271968
[01:07:30]      12:10718347     49.7    13508767        271988
[01:07:40]      12:16891080     49.8    13553543        271977
[01:07:50]      12:23068704     50.0    13598929        271978
[01:08:00]      12:29313454     50.2    13644363        271980
[01:08:10]      12:38409478     50.3    13690265        271992
[01:08:20]      12:45047986     50.5    13737408        272027
[01:08:30]      12:51902333     50.7    13786083        272093
[01:08:40]      12:58809584     50.8    13835022        272164
[01:08:50]      12:65087996     51.0    13880072        272158
[01:09:00]      12:71553570     51.2    13925685        272163
[01:09:10]      12:77812555     51.3    13970255        272147
[01:09:20]      12:84095964     51.5    14014960        272135
[01:09:30]      12:90695859     51.7    14061083        272149
[01:09:40]      12:96518332     51.8    14103907        272101
[01:09:50]      12:102887738    52.0    14148733        272091
[01:10:00]      12:108887962    52.2    14191884        272048
[01:10:10]      12:115833551    52.3    14240687        272115
[01:10:20]      12:122822261    52.5    14290427        272198
[01:10:30]      12:129167967    52.7    14337575        272232
[01:10:40]      13:21033589     52.8    14386765        272304
[01:10:50]      13:26740777     53.0    14430124        272266
[01:11:00]      13:32831984     53.2    14474952        272256
[01:11:10]      13:39143891     53.3    14520644        272262
[01:11:20]      13:45436620     53.5    14566266        272266
[01:11:30]      13:52180606     53.7    14614228        272314
[01:11:40]      13:59246394     53.8    14663486        272386
[01:11:50]      13:65870399     54.0    14710543        272417
[01:12:00]      13:72148797     54.2    14756160        272421
[01:12:10]      13:78254265     54.3    14800768        272406
[01:12:20]      13:84484611     54.5    14846028        272404
[01:12:30]      13:90748645     54.7    14891040        272397
[01:12:40]      13:97084715     54.8    14936771        272403
[01:12:50]      13:103689507    55.0    14983836        272433
[01:13:00]      13:109420593    55.2    15026515        272383
[01:13:10]      14:20202727     55.3    15074782        272435
[01:13:20]      14:26288762     55.5    15120198        272436
[01:13:30]      14:32980760     55.7    15167751        272474
[01:13:40]      14:39239938     55.8    15213773        272485
[01:13:50]      14:45378989     56.0    15258479        272472
[01:14:00]      14:51796567     56.2    15304700        272487
[01:14:10]      14:58055847     56.3    15349891        272483
[01:14:20]      14:64580747     56.5    15396355        272501
[01:14:30]      14:71370916     56.7    15444334        272547
[01:14:40]      14:78042940     56.8    15492680        272598
[01:14:50]      14:84470158     57.0    15539033        272614
[01:15:00]      14:90955161     57.2    15585511        272632
[01:15:10]      14:97339151     57.3    15632190        272654
[01:15:20]      14:104356630    57.5    15682549        272739
[01:15:30]      15:23735948     57.7    15727419        272729
[01:15:40]      15:31315150     57.8    15780316        272858
[01:15:50]      15:38222155     58.0    15829846        272928
[01:16:00]      15:45628572     58.2    15881611        273036
[01:16:10]      15:52343985     58.3    15928980        273068
[01:16:20]      15:58209332     58.5    15972216        273029
[01:16:30]      15:64051198     58.7    16014973        272982
[01:16:40]      15:70382350     58.8    16060701        272986
[01:16:50]      15:77126379     59.0    16107672        273011
[01:17:00]      15:83635074     59.2    16153007        273008
[01:17:10]      15:90574295     59.3    16201547        273059
[01:17:20]      15:96811167     59.5    16247428        273066
[01:17:30]      16:206174       59.7    16291124        273035
[01:17:40]      16:7027096      59.8    16342340        273131
[01:17:50]      16:12810996     60.0    16386019        273100
[01:18:00]      16:20577443     60.2    16440774        273253
[01:18:10]      16:28055999     60.3    16492414        273354
[01:18:20]      16:46732691     60.5    16542025        273421
[01:18:30]      16:46895890     60.7    16543153        272689
[01:18:40]      16:48172788     60.8    16551135        272073
[01:18:50]      16:53433484     61.0    16589117        271952
[01:19:00]      16:59226961     61.2    16631408        271903
[01:19:10]      16:65092731     61.3    16673192        271845
[01:19:20]      16:72139074     61.5    16722008        271902
[01:19:30]      16:77851180     61.7    16763851        271846
[01:19:40]      16:82631833     61.8    16800844        271711
[01:19:50]      16:87489399     62.0    16838755        271592
[01:20:00]      17:3460661      62.2    16885253        271612
[01:20:10]      17:9403147      62.3    16928948        271587
[01:20:20]      17:14769524     62.5    16969165        271506
[01:20:30]      17:21551846     62.7    17015947        271531
[01:20:40]      17:31003137     62.8    17059122        271497
[01:20:50]      17:37017397     63.0    17101116        271446
[01:21:00]      17:43852644     63.2    17148784        271484
[01:21:10]      17:50318377     63.3    17193614        271478
[01:21:20]      17:55468638     63.5    17230458        271345
[01:21:30]      17:61963165     63.7    17274590        271328
[01:21:40]      17:67569583     63.8    17314314        271242
[01:21:50]      17:73027130     64.0    17353169        271143
[01:22:00]      17:78902318     64.2    17395766        271102
[01:22:10]      18:2985553      64.3    17434875        271008
[01:22:20]      18:8203136      64.5    17472468        270890
[01:22:30]      18:13007895     64.7    17508126        270744
[01:22:40]      18:21945594     64.8    17549604        270687
[01:22:50]      18:27316574     65.0    17588311        270589
[01:23:00]      18:33297504     65.2    17629702        270532
[01:23:10]      18:38956717     65.3    17670204        270462
[01:23:20]      18:44649485     65.5    17710490        270389
[01:23:30]      18:50231825     65.7    17751049        270320
[01:23:40]      18:55915090     65.8    17790703        270238
[01:23:50]      18:61775990     66.0    17832450        270188
[01:24:00]      18:67195119     66.2    17872690        270116
[01:24:10]      18:72455803     66.3    17912118        270031
[01:24:20]      19:177422       66.5    17954020        269985
[01:24:30]      19:7027169      66.7    18005208        270078
[01:24:40]      19:13895904     66.8    18053884        270132
[01:24:50]      19:20750226     67.0    18103369        270199
[01:25:00]      19:29615755     67.2    18147181        270181
[01:25:10]      19:36038353     67.3    18193750        270204
[01:25:20]      19:43003171     67.5    18243852        270279
[01:25:30]      19:49799858     67.7    18293883        270352
[01:25:40]      19:56409524     67.8    18343103        270414
[01:25:50]      20:3643170      68.0    18389069        270427
[01:26:00]      20:9575811      68.2    18432256        270399
[01:26:10]      20:15638276     68.3    18476022        270380
[01:26:20]      20:21542247     68.5    18519169        270352
[01:26:30]      20:31771123     68.7    18568121        270409
[01:26:40]      20:38913560     68.8    18617728        270475
[01:26:50]      20:45451138     69.0    18663504        270485
[01:27:00]      20:51201399     69.2    18705700        270443
[01:27:10]      20:56731092     69.3    18745788        270371
[01:27:20]      21:9950171      69.5    18794143        270419
[01:27:30]      21:16290952     69.7    18816808        270097
[01:27:40]      21:21671901     69.8    18856590        270022
[01:27:50]      21:27071963     70.0    18896695        269952
[01:28:00]      21:32827179     70.2    18938169        269902
[01:28:10]      21:38936780     70.3    18982608        269894
[01:28:20]      21:44903909     70.5    19026012        269872
[01:28:30]      22:20039802     70.7    19077339        269962
[01:28:40]      22:27369383     70.8    19129486        270063
[01:28:50]      22:34439888     71.0    19179387        270132
[01:29:00]      22:41630326     71.2    19230724        270220
[01:29:10]      22:48225464     71.3    19279966        270279
[01:29:20]      X:3355160       71.5    19326256        270297
[01:29:30]      X:10142365      71.7    19372703        270316
[01:29:40]      X:17275005      71.8    19419512        270341
[01:29:50]      X:24710373      72.0    19468349        270393
[01:30:00]      X:31531088      72.2    19514758        270412
[01:30:10]      X:38591981      72.3    19561432        270434
[01:30:20]      X:46007904      72.5    19610601        270491
[01:30:30]      X:54772659      72.7    19664457        270611
[01:30:40]      X:65783802      72.8    19716001        270700
[01:30:50]      X:73252552      73.0    19765319        270757
[01:31:00]      X:80337526      73.2    19813036        270793
[01:31:10]      X:86966279      73.3    19858520        270798
[01:31:20]      X:94540731      73.5    19910487        270890
[01:31:30]      X:101754698     73.7    19958423        270928
[01:31:40]      X:109473370     73.8    20008856        271000
[01:31:50]      X:116793510     74.0    20057200        271043
[01:32:00]      X:123758364     74.2    20104455        271071
[01:32:10]      X:130636778     74.3    20150632        271084
[01:32:20]      X:138365412     74.5    20201304        271158
[01:32:30]      X:144993431     74.7    20247281        271168
[01:32:40]      X:151967851     74.8    20293703        271185
[01:32:50]      GL000191.1:23941        75.0    20351304        271350
[01:33:00]      GL000199.1:28759        75.2    20361269        270881
[01:33:10]      GL000219.1:47861        75.3    20367727        270368
[01:33:20]      GL000225.1:100611       75.5    20379378        269925
[01:33:30]      hs37d5:1603179  75.7    20394538        269531
[01:33:40]      hs37d5:5625364  75.8    20422161        269303
[01:33:50]      hs37d5:11116553 76.0    20455865        269156
[01:34:00]      hs37d5:14135912 76.2    20476075        268832
[01:34:10]      hs37d5:16934386 76.3    20494491        268486
[01:34:20]      hs37d5:19281330 76.5    20510676        268113
[01:34:30]      hs37d5:19967951 76.7    20516390        267605
[01:34:40]      hs37d5:21249336 76.8    20525420        267142
[01:34:50]      hs37d5:22108627 77.0    20532388        266654
[01:35:00]      hs37d5:23284540 77.2    20541938        266202
[01:35:10]      hs37d5:25027172 77.3    20554723        265793
[01:35:20]      hs37d5:26260618 77.5    20563710        265338
[01:35:30]      hs37d5:27240000 77.7    20571383        264867
[01:35:40]      hs37d5:27858939 77.8    20576173        264361
[01:35:50]      hs37d5:28660796 78.0    20582258        263875
[01:36:00]      hs37d5:29659114 78.2    20590222        263414
[01:36:10]      hs37d5:30350136 78.3    20595677        262923
[01:36:20]      hs37d5:35466926 78.5    20628317        262781
Total time taken: 4722.65
------------------------------------------------------------------------------
||        Program:                         GPU-GATK4 HaplotypeCaller        ||
||        Version:                                     v3.5.0_ampere        ||
||        Start Time:                       Fri Apr  9 00:17:36 2021        ||
||        End Time:                         Fri Apr  9 01:36:31 2021        ||
||        Total Time:                          78 minutes 55 seconds        ||
------------------------------------------------------------------------------
[lkemp@wbl004 nesi03181]$ srun: Force Terminated job 18961990
slurmstepd: error: *** STEP 18961990.0 ON wbl004 CANCELLED AT 2021-04-09T02:12:17 DUE TO TIME LIMIT ***
srun: Job step aborted: Waiting up to 32 seconds for job step to finish.
srun: error: wbl004: task 0: Killed
```

Total runtime:

- 194 minutes
- 3.2 hours

### A single whole genome on 8 GPUS

Looks like I can't request that many GPU's

```bash
[lkemp@mahuika01 nesi03181]$ srun --account nesi03181 --job-name pbtest --mem 200G --cpus-per-task 8 --gres=gpu:A100:8 --qos=testing --time 01:00:00 --pty bash
srun: error: MaxGRESPerAccount
srun: error: Unable to allocate resources: Job violates accounting/QOS policy (job submit limit, user's size and/or time limits)
```

We talked to Dini and it turns out the 8xA100's are physically on different machines, parabricks require the GPU's to be on the same physical machine in order to use them for the same sample. Therefore, the maximum number of GPU's we can request from slurm is 2.

## Watch the gpus running

In a new session I can ssh into mahuika cluster again, ssh directly to the wbl009 node and run nvidia-smi (first I'll need to load the CUDA module)

```bash
ssh mahuika

ssh wbl009

module load CUDA
watch nvidia-smi
```

*Note. it wasn't always wbl009 I had to ssh into, sometimes the interactive session sent me to a different machine, I suppose one of the other servers that also have 2xA100's on them*

Or even better, I can get some sweet plots

```bash
nvtop
```

## Cancel interactive slurm session

Once finished, cancel the interactive slurm session and free up the resources

```bash
exit
```

## More documentation

Dini and I (mainly Dini) put together this document while we were going through this process: https://hackmd.io/40uk2vc5TPq6O_-c-gpPbQ?view

It has some sweet plots of the resource use when running tests on parabricks on these A100's