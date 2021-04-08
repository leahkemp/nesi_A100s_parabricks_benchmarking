# Calculate mapping metrics

Created: 2021/04/06 17:17:47
Last modified: 2021/04/08 12:42:57

- **Aim:** Benchmarking the performance (speed) of the A100's on NeSi analysing public whole genome sequence (WSG) data
- **Prerequisite software:**
- **OS:** ESR cluster (CentOS Linux 7)

## Table of contents

- [Calculate mapping metrics](#calculate-mapping-metrics)
  - [Table of contents](#table-of-contents)
  - [Overview](#overview)
  - [Getting onto NeSi](#getting-onto-nesi)
  - [Setup](#setup)
    - [Load modules](#load-modules)
    - [Setup interactive slurm session](#setup-interactive-slurm-session)
    - [Test parabricks is running](#test-parabricks-is-running)
      - [Input data](#input-data)
      - [Parabricks software](#parabricks-software)
  - [Run genomes through parabricks](#run-genomes-through-parabricks)

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

Parabricks etc is installed on mahuika so that's were we want to be

## Setup

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

Modules that are available

```bash
module avail
```

Part of my output:

```bash
...
   HISAT2/2.1.0-gimkl-2017a                                   Singularity/3.4.0
   HISAT2/2.1.0-gimkl-2018b                                   Singularity/3.5.2
   HISAT2/2.2.0-gimkl-2020a                                   Singularity/3.6.1
   HISAT2/2.2.1-gimkl-2020a                            (D)    Singularity/3.6.4
   HMMER/3.1b2-gimkl-2017a                                    Singularity/3.7.1                                             (D)
...
```

Parabricks is containerised (in a singularity container) so we're gonna need Singularity to interact with it

```bash
module load Singularity/3.7.1 
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

### Setup interactive slurm session

```bash

```

### Test parabricks is running

#### Input data

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

#### Parabricks software

The Parabricks binary can be found at `/nesi/project/nesi03181/SW/parabricks/pbrun` and can be called directly

```bash
/nesi/project/nesi03181/SW/parabricks/pbrun --help
```

## Run genomes through parabricks

```bash
SW/parabricks/pbrun germline --ref /nesi/project/nesi03181/PBDATA/REFERENCE/human_g1k_v37_decoy.fasta \
                 --in-fq /nesi/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R1_001.fastq.gz /nesi/project/nesi03181/PBDATA/MPHG007-23100082/HGHG007_S6_L006_R2_001.fastq.gz \
                 --knownSites /nesi/project/nesi03181/PBDATA/REFERENCE/1000G_phase1.indels.b37.vcf.gz \
                 --knownSites /nesi/project/nesi03181/PBDATA/REFERENCE/Mills_and_1000G_gold_standard.indels.b37.vcf.gz \
                 --knownSites /nesi/project/nesi03181/PBDATA/REFERENCE/dbsnp_138.b37.vcf.gz \
                 --out-bam /nesi/project/nesi03181/PBDATA/output.bam \
                 --out-variants /nesi/project/nesi03181/PBDATA/output.vcf \
                 --out-recal-file /nesi/project/nesi03181/PBDATA/report.txt
```

```bash

SW/parabricks/pbrun fq2bam --help
```

