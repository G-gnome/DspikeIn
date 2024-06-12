# DspikeIn
The importance of converting relative to absolute abundance in the context of microbial ecology: Introducing the user-friendly DspikeIn R package

![How it works](https://github.com/mghotbi/DspikeIn/blob/MitraGhotbi/DspikeIn.png)


---

## Introduction

In our study, *Tetragenococcus halophilus* and *Dekkera bruxellensis* were selected as taxa to spike into gut microbiome samples  based on our previous studies.
## Methodology

### Growth of Stock Cell Suspensions
- **Tetragenococcus halophilus**: Cultivated in tryptic soy broth.
- **Dekkera bruxellensis**: Cultivated in potato dextrose broth.
- Both microbial cultures were serially diluted, and optical density (OD) measurements were obtained using a ClarioStar plate reader.

### DNA Extraction

- DNA was extracted using the Qiagen DNeasy Powersoil Pro Kit.
- These DNA isolations served as standards to determine the appropriate spike-in volume of cells to represent 0.1-10% of a sample, as detailed in [Roa et al., 2021](https://www.nature.com/articles/s41586-021-03241-8).

---

## Bioinformatics Processing and Downstream Analyses

### Gene Marker Analysis
- 16S rRNA and ITS rDNA gene markers were analyzed.

### Normalization
- Normalization was performed for community-weighted mean ribosomal operon copy numbers.

### DspikeIn Package
The **DspikeIn** package was developed to facilitate:
- Verifying the phylogenetic distances of ASVs/OTUs rooted from spiked species.
- Preprocessing data.
- Calculating the spike-in scaling factor.
- Converting relative abundance to absolute abundance.
- Data transformation and visualization.

We provide a step-by-step walkthrough of each procedure within the **DspikeIn** package.

For more detailed methodology and results, please refer to our soon-to-be-published paper.

---



Similar to the debate on the application of OTUs vs. ASVs, which has many contrasting views on their benefits and drawbacks ([Callahan et al., 2017](https://doi.org/10.1038/ismej.2017.119); [Schloss et al., 2021](https://doi.org/10.1128/msphere.00191-21)), there are positive ([Schirrmeister et al., 2012](https://doi.org/10.1186/1471-2180-12-177); [Stoddard et al., 2015](https://doi.org/10.1093/nar/gku1201)) and opposing ([Louca et al., 2018](https://doi.org/10.1038/s43705-023-00266-0); [Gao and Wu et al., 2023](https://doi.org/10.1038/s43705-023-00266-0)) opinions on gene copy number correction for the 16S rRNA marker. Meanwhile, several novelties and modifications have been added to copy number correction to improve accuracy ([Perisin et al., 2016](https://doi.org/10.1038/ismej.2015.161); [Gao and Wu, 2023](https://doi.org/10.1038/s43705-023-00266-0)).



Using our spike-in positive controls and assessing the percentage of retrieved spiked species, we were able to compare the results at each step before selecting a pathway. We utilized q2-gcn-norm based on rrnDB database (version 5.7) to normalize for 16S rRNA gene marker copy numbers ([qiime2 plugin; gcn-norm](https://github.com/Jiung-Wen/q2-gcn-norm)). Due to the variability in rDNA gene copy numbers, straightforward translation of rDNA read counts into the abundance of individual organisms is precluded ([Lavrinienko et al., 2021](https://doi.org/10.1016/j.tim.2020.05.019)).



In fact, for ITS we did not need to use copy number correction. However, we recommend conducting a literature review before deciding whether to sum or select the maximum abundance of OTUs/ASVs rooted from spiked-in species and calculating spike-in factors. We believe systematic evaluation before selecting or refuting each method can help prevent miscalculations ([Lofgren et al., 2018](https://doi.org/10.1111/mec.14995)), which can aid in establishing a system-dependent method for copy number correction in ITS markers.

---
```r
# Make a new directory and set it as your working directory
create_directory("ExampleITS", set_working_dir = TRUE)
getwd()

# Please note that these functions have been primarily written based on the 
# phyloseq(https://github.com/joey711/phyloseq) and microbiome (https://github.com/microbiome/microbiome) packages.
# Therefore, please start by creating a phyloseq object and follow the instructions.
# To create your phyloseq object, please refer to the phyloseq tutorial (https://joey711.github.io/phyloseq).
# The phyloseq object needs to include OTU/ASV, Taxa, phylogenetic tree, DNA reference, 
# and metadata containing spiked species volume, starting from 0 (no spike species added) to 4 (4 μl of spike cell added).

# Briefly:
otu <- read.csv("otu.csv", header = TRUE, sep = ",", row.names = 1)
tax <- read.csv("tax.csv", header = TRUE, sep = ",", row.names = 1)
meta <- read.csv("metadata.csv", header = TRUE, sep = ",")

# Convert data to appropriate formats
meta <- as.data.frame(meta)
taxmat <- as.matrix(tax)
otumat <- as.matrix(otu)
colnames(taxmat) <- c("Kingdom", "Phylum", "Class", "Order", "Family", "Genus", "Species")
OTU <- otu_table(otumat, taxa_are_rows = TRUE)
TAX <- phyloseq::tax_table(taxmat)

row.names(meta) <- sample_names(OTU)
metadata <- sample_data(meta)
physeq <- phyloseq(OTU, TAX, metadata)
MyTree <- read.tree("tree.nwk")
reference_seqs <- readDNAStringSet(file = "dna-sequences.fasta", format = "fasta")
physeq_16S <- merge_phyloseq(physeq, reference_seqs, MyTree)
physeq_16S <- subset_taxa(physeq_16S, apply(tax_table(physeq_16S), 1, function(x) all(x != "" & !is.na(x))))
physeq_16S <- tidy_phyloseq(physeq_16S)

saveRDS(physeq_16S, file = "physeq_16S.rds")
physeq_16S <- readRDS("physeq_16S.rds")

# Ensure your metadata contains spiked volumes:
# physeq_ITS@sam_data$spiked_volume


```


# Prepare the required information 


```r
# Required Information

# 16S rRNA
spiked_cells <- 1847
species_name <- spiked_species <- c("Tetragenococcus_halophilus", "Tetragenococcus_sp")
merged_spiked_species <- "Tetragenococcus_halophilus"
Tetragenococcus_halophilus <- subset_taxa(physeq_16S, Species == "Tetragenococcus_halophilus" | Species == "Tetragenococcus_sp")
hashcodes <- row.names(tax_table(Tetragenococcus_halophilus))

# If you intend to use the hashcodes to identify your spiked species, please skip this line of code:
# taxa_names(physeq_16S) <- paste0("ASV", seq(ntaxa(physeq_16S)))

# ITS rDNA
spiked_cells <- 733
species_names <- spiked_species <- merged_spiked_species <- "Dekkera_bruxellensis"
```


# Plot phylogenetic tree



```r
## This step can be helpful for handling ASVs with/without Gene Copy Number Correction
# This section demonstrates how to use various functions from the package
# to plot and analyze phylogenetic trees.
# In case there are still several ASVs rooted from the spiked species, you may want to check the phylogenetic distances.
# Phylogenetic tree using a Neighbor-Joining method based on a Jukes-Cantor distance matrix with tip labels represent different Species, olored # by OTU IDs. Let's start by comparing the Sanger read of Tetragenococcus halophilus with the FASTA sequence of Tetragenococcus halophilus from # our phyloseq object.

# Read the DNA sequences from a FASTA file
sequence <- readDNAStringSet("tetra_2.fasta")
 my_alignment <- msa(sequence)
# Convert alignment to DNAStringSet
 aligned_sequences <- as(my_alignment, "DNAStringSet")

# then convert DNAStringSet to phyDat format
phyDat_alignment <- phyDat(as(aligned_sequences, "matrix"), type = "DNA")

# now compute distance matrix using maximum likelihood 
distance_matrix <- dist.ml(phyDat_alignment)

# check the phylogenetic tree cunstructed by the Neighbor-Joining method
phylo_tree <- nj(distance_matrix)
plot(phylo_tree, main = "Neighbor Joining Tree", cex = 1, tip.color = "navy")
# Save the plot
dev.copy(png, "neighbor_joining_tree.png")
dev.off()

```


Figure 1.
![Pattern Comparison](https://github.com/mghotbi/DspikeIn/blob/MitraGhotbi/pattern.comparision.png)



```markdown
## Aligned Sequences

The result of the aligned sequences is shown below:
DNAStringSet object of length 5:
    width seq                                                                                                                                         names               
[1]   292 -------------------TACGTAGGTGGCAAGCGTTGTCCGGATTTATTGGGCGTAAAGCGAGCGC...CTGGTCTGTAACTGACGCTGAGGCTCGAAAGCGTGGGTAGCAAACAGG-------------------- 2ddb215ff668b6a24...
[2]   292 GTGCCAGCAGCCGCGGTAATACGTAGGTGGCAAGCGTTGTCCGGATTTATTGGGCGTAAAGCGAGCGC...CTGGTCTGTAACTGACGCTGAGGCTCGAAAGCGTGGGTAGCAAACAGGATTAGATACCCTGGTAGTCC Tetragenococcus h...
[3]   292 -------------------TACGTAGGTGGCAAGCGTTGTCCGGATTTATTGGGCGTAAAGCGAGCGC...CTGGTCTGTAACTGACGCTGAGGCTCGAAAGCGTAGGTAGCAAACAGG-------------------- 65ab824f29da71010...
[4]   292 -------------------TACGTAGGTGGCAAGCGTTGTCCGGATTTATTGGGCGTAAAGCGAGCGC...CTGGTCTGTAACTGACGCTGAGACTCGAAAGCGTGGGTAGCAAACAGG-------------------- e49935179f23c00fb...
[5]   292 -------------------TACGTAGGTGGCAAGCGTTGTCCGGATTTATTGGGCGTAAAGCGAGCGC...CTGGACTGTAACTGACGCTGAGGCTCGAAAGCGTGGGGAGCAAACAGG-------------------- 0350f990080b4757a...
```



```r

## Additional ways to plot phylogenetic distances using different methods
# Plot phylogenetic tree
plot_tree(Tetragenococcus, output_prefix = "p0", width = 24, height = 26)

# Plot the tree with glommed OTUs at 0.2 resolution
plot_glommed_tree(Tetragenococcus, resolution = 0.2, output_prefix = "top", width = 18, height = 18)

# Plot the phylogenetic tree with multiple sequence alignment
plot_tree_with_alignment(Tetragenococcus, output_prefix = "tree_alignment", width = 15, height = 15)

# If you prefer to check the cophenetic distance:
# Cophenetic distance is the total length of the path connecting two tips through their common ancestor.

# Extract the phylogenetic tree
tree <- phy_tree(Tetragenococcus)
plot(tree)

# Calculate cophenetic distances
tree_dist <- cophenetic.phylo(tree)
print(tree_dist)

```



Figure 2.
![Tree Alignment](https://github.com/mghotbi/DspikeIn/blob/MitraGhotbi/tetra.png)




