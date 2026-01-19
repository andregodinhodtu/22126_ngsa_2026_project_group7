# Project for course 22126 Next Generation Sequencing: group 7

Set project directry as HOME in the server:

```bash
export HOME=/home/projects/22126_NGS/projects/group7/22126_ngsa_2026_project_group7
cd
```

We are going to replicate the biological analysis workflow of the following scientific article:

Thomas RC, Dunn JC, Dawson DA, et al. **Assessing rates of parasite coinfection and spatiotemporal strain variation via metabarcoding: Insights for the conservation of European turtle doves Streptopelia turtur.**
Mol Ecol. 2022;31(9):2730-2751. doi:10.1111/mec.16421

We will follow the analysis steps as outlined in the paper, using tools introduced to us in the course or those of our own choice. 

## Poster Link

https://app.biorender.com/illustrations/6969014082c88e22eefb6cea?isPoster=true&slideId=fc4a1b24-1cc9-4d6f-beac-0e0912f79cd0


### INTRODUCTION: Understanding the article - terms, concepts and methods 


#### ITS and Fe-Hyd genes

**ITS**: internal transcribed spacer
- it is a part of the rRNA gene cluster with the structure 18S rRNA — ITS1 — 5.8S rRNA — ITS2 — 28S rRNA
- it is a noncoding sequence that is highly variable between strains and is used as a genetic fingerprint
- in the paper used to detect coinfection, count parasite strains and compare diversity

**Fe-hyd**: ferredoxin hydrogenase gene
- codes a protein involved in anaerobic metabolism
- it does not mutate as fast as ITS, so it is less likely to have extreme variation
- in the paper used to confirm species identity and validate what was concluded using ITS
- also good for phylogenetic support


#### Use of Sanger sequencing

In the article, Sanger sequencing was used on a subset of positive samples to later allow for validation of the HTS methods. The rest of the positive samples and 25 of the Sanger subset was then used to perform MiSeq sequencing. 

The sequences read using Sanger are available in GenBank and we will be using them as reference sequences.


#### Clarification of Fi5 and Ri7 indexes

Referring to the first sentence of the second paragraph in the 2.9 Sequence analysis section: *MiSeq sequences were demultiplexed into sample files according
to Fi5 and Ri7 indexes by the Illumina miseq control software
(v2.5.0.5).* 

In this case, indexing refers to the process of barcoding samples. 

Barcode is a small sequence added to the forward and reverse primers (for example ATCG):
- forward primer gets Fi5 barcode
- reverse primer gets Ri7 barcode

Based on the barcode combinations, the machine sorted them into separate fastq files - 2 fastq files per sample, which we will merge later. 

**Sample**: DNA extracted from one host individual at one time and place








