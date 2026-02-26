---
title: "Untitled"
output: html_document
date: "2026-02-26"
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Install software packages

```{r}
if (!requireNamespace("devtools", quietly = TRUE)){install.packages("devtools")}
if (!"devtools" %in% installed.packages()){install.packages("devtools")}

BiocManager::install("qiime2R") # install decontam
BiocManager::install("decontam") # install decontam
BiocManager::install("phyloseq") # install phyloseq
BiocManager::install("ggplot2") # install ggplot2
BiocManager::install("tidyr") # install ggplot2
devtools::install_github("gmteunisse/fantaxtic")

library(qiime2R)
library(phyloseq)
library(decontam)
library(tidyr)
library(ggplot2)
library(fantaxtic)
```

## Import data 
```{r}
asvs <- read_qza("data/05-16S-rRNA-denoise-dada2-feature-table.qza")
taxonomy <- read_qza("data/06-16S-rRNA-taxonomy-assignment-blast.qza")
tree <- read_qza("data/07-16S-rRNA-fastree-midrooted-tree.qza")
metadata <- read.delim2("metadata/2026-02-25-16S-rRNA-metadata.txt", sep = "\t", row.names = 1)
```

## extract data from files
```{r}
asv_df <- asvs$data # we can view and save the actual data by specifying '$'
taxonomy_df <- taxonomy$data # save taxonomy info as a dataframe
phylo_tree <- tree$data # tree data is stored as a phyloseq object
```

## Fix taxonomy strings
```{r}
taxonomy_fixed_df <- taxonomy_df %>% separate_wider_delim(Taxon, delim = ";", names_sep = "", too_few = "align_start") # split taxonomy into different columns 
taxonomy_fixed_df[is.na(taxonomy_fixed_df)] <- "Unassigned" # rename NAs into unassigned
head(taxonomy_fixed_df) # few the first 5 lines

# convert the data into a matrix datatype
taxonomy_fixed_df <- as.data.frame(taxonomy_fixed_df) # force into a dataframe
row.names(taxonomy_fixed_df) <- taxonomy_fixed_df$Feature.ID # make first column into row names
taxonomy_fixed_df$Feature.ID <- NULL # remove the first column
taxonomy_matrix <- as.matrix(taxonomy_fixed_df) # convert to a matrix
head(taxonomy_matrix)
```

# Merge files into phyloseq object

```{r}
physeq_asv <- otu_table(asv_df, taxa_are_rows = T) # convert into phyloseq object
physeq_tax <- tax_table(taxonomy_matrix) # convert into phyloseq object
physeq_meta <- sample_data(metadata) # convert into phyloseq object

phylo_object <- phyloseq(physeq_asv, physeq_tax, physeq_meta) # merge into phyloseq object
phylo_object_tree <- merge_phyloseq(phylo_object, phylo_tree) # add tree into phyloseq object

phylo_object_tree # get summary of the data
```





