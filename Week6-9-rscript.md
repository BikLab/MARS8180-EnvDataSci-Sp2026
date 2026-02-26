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

## Extract data from files
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

## Merge files into phyloseq object

```{r}
physeq_asv <- otu_table(asv_df, taxa_are_rows = T) # convert into phyloseq object
physeq_tax <- tax_table(taxonomy_matrix) # convert into phyloseq object
physeq_meta <- sample_data(metadata) # convert into phyloseq object

phylo_object <- phyloseq(physeq_asv, physeq_tax, physeq_meta) # merge into phyloseq object
phylo_object_tree <- merge_phyloseq(phylo_object, phylo_tree) # add tree into phyloseq object

phylo_object_tree # get summary of the data
```

## Remove contaminant sequences
```{r}
sample_data(phylo_object_tree)$is.neg <- sample_data(phylo_object_tree)$SampleType == "Control" # create a sample-variable for contaminants
phylo_object_contaminants <- isContaminant(phylo_object_tree, method = "prevalence", neg="is.neg", threshold=0.1, detailed = TRUE, normalize = TRUE) # detect contaminants based on control samples and their ASV prevalance
table(phylo_object_contaminants$contaminant) # check number of ASVs that are contaminents

# Make phyloseq object of presence-absence in negative controls and true samples
phylo_object_contaminants.pa <- transform_sample_counts(phylo_object_tree, function(abund) 1 * (abund > 0)) # convert phyloseq table to presence-absence
ps.pa.neg <- subset_samples(phylo_object_contaminants.pa, SampleType=="Control")
ps.pa.pos <- subset_samples(phylo_object_contaminants.pa, SampleType=="True Sample")
df.pa <- data.frame(pa.pos=taxa_sums(ps.pa.pos), pa.neg=taxa_sums(ps.pa.neg), contaminant=phylo_object_contaminants$contaminant) # convert into a dataframe

# Let's plot our contaminants 
# Make phyloseq object of presence-absence in negative controls and true samples
ggplot(data=df.pa, aes(x=pa.neg, y=pa.pos, color=contaminant)) + geom_point() + xlab("Prevalence (Negative Controls)") + ylab("Prevalence (True Samples)")

#Now, we can filter out these contaminants from our dataset.
phylo_obj_tree_sans_contam <- prune_taxa(!phylo_object_contaminants$contaminant, phylo_object_tree) # remove ASVs identified as decontaminants from the dataset
phylo_obj_tree_sans_contam_sans_controls <- subset_samples(phylo_obj_tree_sans_contam, SampleType != "Control") ## Remove blanks and positive controls
phylo_obj_tree_sans_contam_sans_controls
```

## Generate taxonomy barplots
```{r}
# We are are using the fantaxtic package to create a nest taxonomy barplot
top_nested <- nested_top_taxa(phylo_obj_tree_sans_contam_sans_controls,
                              top_tax_level = "Taxon2",
                              nested_tax_level = "Taxon3",
                              n_top_taxa = 5, # Top x Taxon2 (Phylum)
                              n_nested_taxa = 3, # Top x Taxon3 (Class)
                              nested_merged_label = "Other <tax>")

plot_nested_bar(ps_obj = top_nested$ps_obj,
                top_level = "Taxon2",
                nested_level = "Taxon3",
                nested_merged_label = "NA and other <taxa>")

# Let's create a facet and group by a common variable
# We can save the plot to a variable
taxonomy_barplot <- plot_nested_bar(ps_obj = top_nested$ps_obj,
                top_level = "Taxon2",
                nested_level = "Taxon3",
                nested_merged_label = "NA and other <taxa>") +
  facet_wrap(Habitat~Site,
           scales = "free_x", ncol = 6) + 
  guides(fill = guide_legend(ncol = 1)) +
  theme(axis.text.x = element_text(size = 6))
taxonomy_barplot

#save file
ggsave(filename = "figures/2026-02-25-taxonomy-barplot-top-taxa.png", dpi = 300, height = 8.5, width = 11)
```

## Alpha diversity analysis
```{r}
# We can estimate diversity using phyloseq
# Let's estimate number of observed ASVs 
alpha_div_observed <- phyloseq::estimate_richness(phylo_obj_tree_sans_contam_sans_controls, measures = "Observed") # calculate alpha diversity
head(alpha_div_observed) # view the first few lines of our R object

# Let's combine our results with our metadata
alpha_div_observed_metadata <- data.frame( # save as a dataframe
  phyloseq::sample_data(phylo_obj_tree_sans_contam_sans_controls), # access metadata 
  "Observed" = phyloseq::estimate_richness(phylo_obj_tree_sans_contam_sans_controls, measures = "Observed")) # calculate Observed diversity and save it to a column names "Observed"
alpha_div_observed_metadata

# plot our results
alpha_div_observed_plot <- ggplot(alpha_div_observed_metadata, aes(x=Site, y=Observed)) + # what are we plotting
  geom_boxplot() + # make it into a boxplot plot
  theme_minimal() # and lets make sure its "pretty"  
alpha_div_observed_plot

# Lets calculate alpha diversity using other common metrics
alpha_div_observed_metadata <- data.frame(
  phyloseq::sample_data(phylo_obj_tree_sans_contam_sans_controls), # get metadata
  "Reads" = phyloseq::sample_sums(phylo_obj_tree_sans_contam_sans_controls), # number of reads
  "Observed" = phyloseq::estimate_richness(phylo_obj_tree_sans_contam_sans_controls, measures = "Observed"), # count observed ASVs
  "Shannon" = phyloseq::estimate_richness(phylo_obj_tree_sans_contam_sans_controls, measures = "Shannon"), # Calculate Shannon Diversity
  "InvSimpson" = phyloseq::estimate_richness(phylo_obj_tree_sans_contam_sans_controls, measures = "InvSimpson")) # calculate InvSimpson
alpha_div_observed_metadata
```

### Beta diversity analysis
```{r}
# normalize dataset by relative abundance
phylo_obj_tree_sans_contam_sans_controls_relab <- transform_sample_counts(phylo_obj_tree_sans_contam_sans_controls, function(x) x / sum(x) )

bray_dist <- phyloseq::distance(phylo_obj_tree_sans_contam_sans_controls_relab, method="bray") # calculate bray-curtis metric
ordination <- ordinate(phylo_obj_tree_sans_contam_sans_controls_relab, method="PCoA", distance=bray_dist) # Perform ordination using bray-curtis metric
plot_ordination(phylo_obj_tree_sans_contam_sans_controls_relab, ordination, color="Site", shape = "Habitat") + theme(aspect.ratio=1) # plot the ordination using ggplot2
```



