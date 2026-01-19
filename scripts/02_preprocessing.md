---
title: "02_preprocessing"
output: html_document
date: "2026-01-18"
---

# Pre-Processing

Done using fastp on our Raw data. Details below.

#### Raw Data

The samples consist of DNA sequences extracted from doves's blood, where DNA from *Trichomonas gallinae (T. gallinae)*, a parasite, was isolated and PCR amplified for specific regions. These include:

-   Trichomonas gallinae ITS 1/5.8S/ITS 2 ribosomal region

-   Trichomonas gallinae Fe-hydrogenase region

-   PCR amplification of apicomplexan cytochrome b region

More details can be found here [INSERT REFERENCE TO ARTICLE].

The data itself consist of sample files with forward and reverse reads of sanger and MiSeq sequences.

A small subset of positive PCR T. gallinae samples was purified using Sanger sequencing, while all other positive samples and 25 of the previously mentioned samples where sequenced using illumina MiSeq. The precise numbers and extraction and PCR methods can be found in [REFERENCE TO APPENDIX 1 OF ARTICLE].

#### DNA Barcodes

There has also been added sample-specific-indexes, Fi5 and Ri7, during PCR to add unique DNA barcodes to the sequences to allow for metabarcoding later.

#### Some Library Preparation Details

The PCR used a library with 250 paired-end reads and was performed using MiSeq benchtop sequencer (illumina). Squences were run over four different MISeq runs, combined with samples from other larger projects. These runs contained around 10% dublicates.

#### MiSeq Sequencing

The MiSeq sequences where then demultiplexed into seperate sample files based on Fi5 and Ri7 indexes, which we download, thus allowing us to skip a process further along, indexing.

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

We merged the RE reads and hot the merged file as an output.

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

## Fastp sbatch Script

```{bash}
#!/bin/bash -l
#SBATCH --job-name=fastp_trim
#SBATCH --output=/home/projects/22126_NGS/projects/group7/22126_ngsa_2026_project_group7/scripts/log/slurm-%x.%j.out
#SBATCH --error=/home/projects/22126_NGS/projects/group7/22126_ngsa_2026_project_group7/scripts/log/slurm-%x.%j.err
#SBATCH -c 4
#SBATCH --mem=10G
#SBATCH --time=01:00:00

set -euxo pipefail

# Define Directories
BASE_DIR="/home/projects/22126_NGS/projects/group7/22126_ngsa_2026_project_group7"
FASTQ_DIR="${BASE_DIR}/data/data_raw/sra_fastq/parasites"
OUT_DIR="${BASE_DIR}/results/preprocessing_fastp/parasites"
LOG_DIR="${BASE_DIR}/scripts/log"

# Create directories if they don't exist
mkdir -p "${OUT_DIR}"
mkdir -p "${LOG_DIR}"

for file in "${FASTQ_DIR}"/*_1.fastq.gz; do

filename=$(basename "$file")
sample="${filename%_1.fastq.gz}"

ADAPTER_FWD="TCTACACGTTCAGAGTTCTACAGTCCGACGATC"
ADAPTER_REV="GTGACTGGAGTTCAGACGTGTGCTCTTCCGATCT"

# Run fastp
fastp \
  -i "${FASTQ_DIR}/${sample}_1.fastq.gz" \
  -I "${FASTQ_DIR}/${sample}_2.fastq.gz" \
  --merged_out "${OUT_DIR}/${sample}_merged.fastq.gz"\
  --out1 "${OUT_DIR}/${sample}_unmerged1.fastq.gz" \
  --out2 "${OUT_DIR}/${sample}_unmerged2.fastq.gz" \
  --merge \
  --adapter_sequence "${ADAPTER_FWD}" \
  --adapter_sequence_r2 "${ADAPTER_REV}" \
  --trim_poly_g \
  --trim_poly_x \
  --cut_front \
  --cut_tail \
  -w 4 \
  -l 100 \
  -q 20 \
  -h "${OUT_DIR}/trim_merge_reports/parasites/${sample}.html" \
  -j "${OUT_DIR}/trim_merge_reports/parasites/${sample}.json"

fastp \
  -i "${OUT_DIR}/${sample}_merged.fastq.gz" \
  -o "${OUT_DIR}/${sample}_merged_250bp.fastq.gz" \
  -l 250 \
  --disable_adapter_trimming \
  --disable_quality_filtering \
  -w 4 \
  -h "${OUT_DIR}/trim_merged_reports/parasites/${sample}_merged_filter.html" \
  -j "${OUT_DIR}/trim_merged_reports/parasites/${sample}_merged_filter.json"

done
```

## 
