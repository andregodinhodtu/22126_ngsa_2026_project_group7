# Generate Tables

We aim to generate a csv file of the form:

| filename | matches               |
|----------|-----------------------|
| SRR{}    | {refId1};{refId2};... |
| SRR{}    | {refId1};{refId2};... |

To do so, for each file we:
1. Select the sequences
2. Count how many times they are repeated
3. Keep only those repeated >= 50 times
4. Save in a temporary fasta file
5. Use `blastn` to identify the sequences (reference genome ID)
6. Add a new row to `matches.csv` containing the `filename` and reference genomes

We can then load this csv into R to perform statistics on the presence of strains.


```{bash}
#!/bin/bash

echo "filename,matches" >> /home/projects/22126_NGS/projects/group7/22126_ngsa_2026_project_group7/results/fasta/matches.csv

# for: Trichomonas gallinae (SRR16955911–SRR16955967):
for run in $(seq 16955911 16955967); do
  zcat "/home/projects/22126_NGS/projects/group7/22126_ngsa_2026_project_group7/results/preprocessing_fastp/trichomonas/SRR${run}_merged_250bp.fastq.gz" \
    | awk 'NR%4==2' \
    | sort \
    | uniq -c \
    | awk '$1>=50 {printf(">seq_%d_count_%d\n%s\n", NR, $1, $2)}' > /home/projects/22126_NGS/projects/group7/22126_ngsa_2026_project_group7/results/fasta/tmp.fasta


  blastn \
    -query /home/projects/22126_NGS/projects/group7/22126_ngsa_2026_project_group7/results/fasta/tmp.fasta \
    -db nt \
    -remote \
    -max_target_seqs 5 \
    -outfmt "6 qseqid sseqid pident" \
    -out /home/projects/22126_NGS/projects/group7/22126_ngsa_2026_project_group7/results/fasta/tmp.tsv

  matches=$(grep 100.000 /home/projects/22126_NGS/projects/group7/22126_ngsa_2026_project_group7/results/fasta/tmp.tsv | cut -f2 | sed 's/.*|gb|//; s/|//' | sort | uniq -c | paste -sd ";" -)
  filename="SRR${run}"
  echo "$filename,\"$matches\"" >> /home/projects/22126_NGS/projects/group7/22126_ngsa_2026_project_group7/results/fasta/matches.csv
  echo $run
done
```
