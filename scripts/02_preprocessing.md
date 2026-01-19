---
title: "02_preprocessing"
output: html_document
date: "2026-01-18"
---

# Pre-Processing

Done using fastp on our Raw data. The data itself consist of sample files with forward and reverse reads of sanger and MiSeq sequences.

#### MiSeq Sequencing

The MiSeq sequences where demultiplexed beforehand into separate sample files based on Fi5 and Ri7 indexes thus allowing us to skip the indexing process.

## Fastp

Pre-Procesing is performed using fastp to check the quality of the data and trim it to remove:

-   Illumina adapter sequences

-   Low quality bases in leading and trailing ends

-   Low quality sequences below:

    -   The minimum Phred quality score of 20

    -   The minimum length of 100 bp

We then align the paired-end reads and create a merged sequence. This was done without indexing the samples, since it was done during MiSeq.

This process was combined into a sbatch script, and can be found in `02_fastp_trigonomas.sbatch` and `02_fastp_parasite.sbatch`. It can also be found in the bottom of this documentation. The process can be broken down into the following parts.

#### 1. Setup sbatch job, file directories and paths

Containing the sbatch header, paths and creates directories if missing.

Start job by typing: `sbatch myfile.sbatch`

check your job status: `squeue --me`

Cancel job: `scancel <job_id>`

#### 2. Sample directive selection

Gets and runs each sample in the directive. Loop runs for the whole fastp command for the current sample.

#### 3. Fastp

ultra fast all-in-one FASTQ preporcessor. It is used for quality control and preprocessing of our raw data. See more on: [fastp manual](https://open.bioqueue.org/home/knowledge/showKnowledge/sig/fastp).

#### 3.1. Input and Output files

We used it with paired-end (PE) raw data in fastq files.

`-i`,`-I`,`-out1`,`-out2`

#### 3.2. Merging paired-end reads

We merged the RE reads and got the merged file as an output.

`--merge`,`--merged_out`

#### 3.3. Removing illumina adaptor sequences

During the first run of PCR two illumina sequencing primer sites were added to each forward and reverse primers:

F: 5′-TCTACA CGTTCAGAGTTCTACAGTCCGACGATC-3′

R: 5′–GTGACTG GAGTTCAGACGTGTGCTCTTCCGATCT-3′

Which we removed by defining then and then removing them using:

`-- adapter_sequence`,`--adapter_sequence_r2`

#### 3.4. Trimming poly_g and ploy_x

`--trim_poly_g`,`--trim_poly_x`

#### 3.5. cut_front and cut_tail

`--cut_front`,`--cut_tail`

#### 3.6. specifying thread number, min_length and min Phred score

`-w 4`,`-l 100`,`-q 20`

#### 3.7 Reporting as html and json file

`-h`,`-j`

#### 4. fastp: limiting to sequences at or above 250 bp

[DESCRIBE DIFFERENCES]

`-l 250`,`--disable_adapter_trimming`,`--disable_quality_filtering`,`-w 4`

## Fastp sbatch Script Trigonomas

```{bash}

# put new sbatch script for trigonomas

```

## Fastp sbatch Script Parasites

```{bash}

# new bash script for triginomas

```
