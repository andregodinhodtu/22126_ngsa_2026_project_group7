# Pipeline carried to perform phylogenomics analysis 

Analysis of the ITS Region Following the Original Study

The original study published its polished ITS region dataset on GenBank, which combined publicly available reference sequences with the authors’ processed data derived from their raw sequence analyses. The GenBank sequences referenced in the study were retrieved by the authors using specific keyword searches, after which representative sequences for each strain were selected. These sequences were used to construct Figure 1 in the original paper and served as the basis for the dataset utilized in this analysis.

In the first stage of this work, the same dataset was retrieved and analyzed in order to replicate the results reported in the original publication and to verify the consistency of the methodology.

In the second stage, a new dataset was constructed by combining the same GenBank reference sequences with results from independent analyses of the raw sequences. This dataset represented the newly generated data within the present study and was used to perform comparative analyses with the original findings.

## 1. Local alignment and trimming of fasta files - Script_01

Local alignment of FASTA sequences were performed using the CLUSTAL W algorithm implemented in the msa R package (version 1.42.0). The resulting alignments were visually inspected in MEGA (version 12), and sequences were trimmed between positions 162 and 384, yielding the same alignment length as reported in the original study (224 bp).

```{r}
fasta_file <-  "path/to/fasta_file.fasta"
its_biostrings <- readDNAStringSet(fasta_file)
alignment <- msa(its_biostrings, method = "ClustalW")

alignment_ape <- as.DNAbin(alignment)
image(alignment_ape)

# Trimming indexes were chosen using MEGA software for accurate visualization 
its_trimmed <- alignment_ape[, 162:384]
image(its_trimmed) 
```

## 2. Best-fitting nucleotide substituiton model

In the original study, the optimal nucleotide substitution model for the ITS region was determined using jModelTest, which identified the Hasegawa–Kishino–Yano model with a gamma distribution and equal rate weights (HKY+G). Model selection performed with the phangorn R package (version 2.12.1) supported the same model.

```{r}

phylo_data <- phyDat(its_trimmed, type = "DNA")

mt <- modelTest(phylo_data)
head(mt[order(mt$BIC),])

# Model         df  logLik    AIC      AICw        AICc       AICcw     BIC
# 15   HKY+G(4) 48 -802.7573 1701.515 0.0192777954 1728.549 0.082182212 1865.059
# 43 TPM2u+G(4) 49 -800.6062 1699.212 0.0609512337 1727.536 0.136380764 1866.164
# 39  TPM2+G(4) 46 -809.2713 1710.543 0.0002111871 1735.111 0.003089856 1867.272
# 35 TPM1u+G(4) 49 -801.3030 1700.606 0.0303634294 1728.930 0.067939358 1867.557
# 23   TrN+G(4) 49 -801.3930 1700.786 0.0277508310 1729.110 0.062093567 1867.737
# 16 HKY+G(4)+I 49 -801.7760 1701.552 0.0189203852 1729.876 0.042335100 1868.503

```

## 3. Running BEAUTi

Analyses were conducted in BEAUTi (version 10.5.0) using a strict molecular clock and the Hasegawa–Kishino–Yano substitution model with a gamma distribution and equal rate weights (HKY+G). A Yule process was specified as the tree prior. The Markov chain was run for 25,000,000 generations, with parameters sampled every 1,000 generations.

## 4. Running BEAST

BEAST (version 10.5.0) was used for Bayesian phylogenetic analysis. The BEAGLE library (version 4.0.0) was installed to optimize computational performance during likelihood calculations.

## 5. Tracer to confirm convergence

Convergence diagnostics were intended to be performed using Tracer. However, compatibility issues arose due to the requirement for an older Java version. To address this, a custom script was developed to evaluate parameter convergence.

```{r}

logfile <- read.table("path/to/log_file.log")

burnin_fraction <- 0.1
n_samples <- nrow(logfile)
start_index <- round(n_samples * burnin_fraction)
chain_data <- logfile[start_index:n_samples, ]

mcmc_chain <- as.mcmc(chain_data)

ess_values <- effectiveSize(mcmc_chain)
print("ESS Values (aim for > 200):")
print(ess_values)

```

## 6. TreeAnnotator to build the tree

TreeAnnotator (version 10.5.0) was used to summarize the sampled trees and generate the maximum clade credibility (MCC) tree. A 10% burn-in (2500 trees) was applied to exclude the initial portion of the MCMC samples prior to summarization, ensuring only post-convergence trees were included in the final analysis.

## 7. Visualization of the tree - TO BE CHANGED

```{r}

tree <- read.beast("path/to/beast_output.tree")

tree$node.label <- round(as.numeric(tree$node.label), 2)
tree@phylo$tip.label <- sub(" .*", "", tree@phylo$tip.label)

ggtree(tree) +
  geom_tiplab(size=2.5, align=TRUE, linesize=.5) +        
  geom_nodelab(aes(label=round(posterior,2)), vjust=-0.5, size=2) + 
  theme_tree2() +
  xlim(0, 0.5) + 
  labs(title="T. gallinae phylogeny(ITS)")

```

The resulting maximum clade credibility (MCC) tree was visualized in R using the output file generated by TreeAnnotator. Visualization and annotation were carried out with the treeio, ggtree, ape, and tidyverse packages, enabling detailed inspection of tree topology and clade support.


