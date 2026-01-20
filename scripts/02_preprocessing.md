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
To remove poly_g chains commonly found in illumina sequences and poly_a chains. 

`--trim_poly_g`,`--trim_poly_x`

#### 3.5.  --trim_front1 and cut_tail
cut the first and last bases on the sequences. 
trim_front1 with 10 positions trimmed was chosen based on looking at our sequences and alternatives like cut_front
cut_tail removes bases with a quality score below 20 (default) from the 3' end. 

` --trim_front1`,`--cut_tail`

#### 3.6. specifying thread number, min_length and min Phred score
Selected based on article requirements. 

`-w 4`,`-l 100`,`-q 20`

#### 3.7 Reporting as html and json file
Get output as html and json.

`-h`,`-j`

#### 4. fastp: limiting to sequences at or above 250 bp
Done to follow procedure in article. 
A new fastp command is run with these new commands. It limits sequences to only the songer sequences above 250 bp.
It is seen in the last part of the sbatch files. 

`-l 250`,`--disable_adapter_trimming`,`--disable_quality_filtering`,`-w 4`

## Fastp sbatch Script Trigonomas

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
FASTQ_DIR="${BASE_DIR}/data/data_raw/sra_fastq/trichomonas"
OUT_DIR="${BASE_DIR}/data/preprocessing_fastq/trichomonas"
REP_DIR="${BASE_DIR}/results/trim_merge_reports/"
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
  --adapter_sequence "${ADAPTER_FWD}" \
  --adapter_sequence_r2 "${ADAPTER_REV}" \
  --trim_poly_g \
  --trim_poly_x \
  --trim_front1 10 \
  --cut_tail \
  -w 4 \
  -l 100 \
  -q 20 \
  --merge \
  -h "${REP_DIR}/trichomonas/${sample}.html" \
  -j "${REP_DIR}/trichomonas/${sample}.json"

fastp \
  -i "${OUT_DIR}/${sample}_merged.fastq.gz" \
  -o "${OUT_DIR}/${sample}_merged_250bp.fastq.gz" \
  -l 250 \
  --disable_adapter_trimming \
  --disable_quality_filtering \
  --disable_trim_poly_g \
  -w 4 \
  -h "${REP_DIR}/trichomonas/${sample}_merged_filter.html" \
  -j "${REP_DIR}/trichomonas/${sample}_merged_filter.json"

done

```

## Fastp sbatch Script Parasites

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
OUT_DIR="${BASE_DIR}/data/preprocessing_fastq/parasites"
REP_DIR="${BASE_DIR}/results/trim_merge_reports"
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
  --adapter_sequence "${ADAPTER_FWD}" \
  --adapter_sequence_r2 "${ADAPTER_REV}" \
  --trim_poly_g \
  --trim_poly_x \
  --trim_front1 10 \
  --cut_tail \
  -w 4 \
  -l 100 \
  -q 20 \
  --merge \
  -h "${REP_DIR}/parasites/${sample}.html" \
  -j "${REP_DIR}/parasites/${sample}.json"

fastp \
  -i "${OUT_DIR}/${sample}_merged.fastq.gz" \
  -o "${OUT_DIR}/${sample}_merged_250bp.fastq.gz" \
  -l 250 \
  --disable_adapter_trimming \
  --disable_quality_filtering \
  -w 4 \
  -h "${REP_DIR}/parasites/${sample}_merged_filter.html" \
  -j "${REP_DIR}/parasites/${sample}_merged_filter.json"

done

```
