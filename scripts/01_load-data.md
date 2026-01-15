# Data loading

Data loaded using information from the article: *Trichomonas gallinae sequence data are available on GenBank under Accession Numbers MN587086–MN587101, MT418239–MT418250 and MT720718. Data from NGS sequencing runs and metadata are available from the Sequence Read Archive (SRA) under Bioproject accession number PRJNA578480; file accession numbers SRR16955911–967 (Trichomonas gallinae) and SRR17676090–131 (blood parasites).*

SRA data retrieved from ENA. (https://www.ebi.ac.uk/ena/browser/view/PRJNA578480?show=reads ; 15. 1. 2026)
GenBank data retrieved from NCBI.

/data directory structure:

```
/data
├── data_raw
│   ├── sra_fastq
│   │   ├── trichomonas
│   │   │   └── ...        # FASTQ files (forward and reverse reads)
│   │   └── parasites
│   │   │   └── ...        # FASTQ files (forward and reverse reads)  
│   └── genbank_refs
│   │   ├── trichomonas
│   │   │   └── ...        # FASTA files - references
├── metadata
└── article
```


## Data from this study

### Fastq files 

Retrieving data for ***Trichomonas gallinae (SRR16955911–SRR16955967)***:

```bash
for run in $(seq 16955911 16955967); do
  srr="SRR${run}"
  prefix=${srr:0:6}         # SRR169
  subdir=$(printf "%03d" ${run:6:3})  # pad to 3 digits: 911 -> 911, 966 -> 966
  base="ftp://ftp.sra.ebi.ac.uk/vol1/fastq/${prefix}/${subdir}/${srr}"
  wget -nc "${base}/${srr}_1.fastq.gz"
  wget -nc "${base}/${srr}_2.fastq.gz"
done
```

Retrieving data for ***Blood parasites (SRR17676090–SRR17676131)***:

```bash
for run in $(seq 17676090 17676131); do
  srr="SRR${run}"
  prefix=${srr:0:6}         
  subdir=$(printf "%03d" ${run:6:3})  
  base="ftp://ftp.sra.ebi.ac.uk/vol1/fastq/${prefix}/${subdir}/${srr}"
  wget -nc "${base}/${srr}_1.fastq.gz"
  wget -nc "${base}/${srr}_2.fastq.gz"
done
```

### Metadata

yeah i have not figured this out yet


## GenBank data

Command taken and adapted for our case, from user Pierre Lindenbaum's response at https://www.biostars.org/p/418569/ .


```bash
for acc in MN587086 MN587087 MN587088 MN587089 MN587090 MN587091 \
           MN587092 MN587093 MN587094 MN587095 MN587096 MN587097 \
           MN587098 MN587099 MN587100 MN587101 \
           MT418239 MT418240 MT418241 MT418242 MT418243 MT418244 \
           MT418245 MT418246 MT418247 MT418248 MT418249 MT418250 \
           MT720718
do
    wget -q -O ${acc}.fasta \
    "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nuccore&id=${acc}&rettype=fasta"
done
```


