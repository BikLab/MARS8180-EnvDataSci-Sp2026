## Data analysis in R

We are going to use the DESeq2 R pacakge [https://genomebiology.biomedcentral.com/articles/10.1186/s13059-014-0550-8]() to conduct a differential gene abundance analysis to identify the genes that are more abundant in the surface water compared to the deep chlorophyll maximum.

First, we need to download three files from the computing cluster.

* The salmon merged quant files
* The emapper gene annotations
* The metadata file

First, we are going to use bash to keep only the first two columns of the eggnog results. This will make it easier for us to process the data in R. 

```
cd /work/mars8180/instructor_data/metatranscriptomics/analysis/15-eggnog

awk '{print $1, $2}' tara.emapper.annotations > tara.emapper.annotations-contig-gene.txt
```

Now, we can download them to our computer using the following commands:

```
mkdir data metadata

scp userid@txfer.gacrc.uga.edu:/work/mars8180/instructor_data/metatranscriptomics/analysis/15-eggnog/tara.emapper.annotations-contig-gene.txt .

scp userid@txfer.gacrc.uga.edu:/work/mars8180/instructor_data/metatranscriptomics/analysis/12-salmon-all-sample-quant-counts.txt analysis/

scp userid@txfer.gacrc.uga.edu:/work/mars8180/instructor_data/metatranscriptomics/metadata/2025-04-05-metatranscriptomics-tara-oceans-metadata.txt metadata/
```


```r
#### load your packages 
#### install them if not previosly installed
library(tximport)
library(DESeq2)
library(tidyverse)
library(data.table)
library(EnhancedVolcano)

#### METADATA FILE #####
# import the metadata file
metadata <- read.delim2("metadata/2025-04-05-metatranscriptomics-tara-oceans-metadata.txt", row.names = 1)
#rownames(metadata) <- metadata$sampleID
metadata <- metadata[ order(row.names(metadata)), ] # order the metadata file based on the row name



#### SALMON FILE ##### 
# import salmon quants
salmon_data <- fread("data/12-salmon-all-sample-quant-counts.txt", header = T)
# coerce to a dataframe
setDF(salmon_data) 
# change the contig name to row name
rownames(salmon_data) <- salmon_data$Name
# remove the column 
salmon_data$Name <- NULL 



#### EMAPPER FILE #####
# import the emapper annotations
tx2gene <- fread("data/tara.emapper.annotations-contig-gene.txt", header = F, skip = 5)
# coerce to a dataframe
setDF(tx2gene)
# remove ".p#" after contig
tx2gene$V1 <- gsub(".p[0-9]", "", tx2gene$V1)




#### MERGE DATA ####
# merge the data by contig
merged_data <- merge(salmon_data, tx2gene, by.x = "row.names", by.y = "V1")
# remove row.names column
merged_data$Row.names <- NULL




#### EXTRACT SAMPLE NAMES ####
sample_cols <- names(salmon_data)




#### AGGREGATE DATA ####
# aggregate data by gene 
gene_abundance_dt <- aggregate(. ~ V2, data = merged_data[, c(sample_cols, "V2")], FUN = sum)
# make gene name the row name
rownames(gene_abundance_dt) <- gene_abundance_dt$V2
# remove the column V2
gene_abundance_dt$V2 <- NULL
# convert to a matrix 
gene_abundance_matrix <- as.matrix(gene_abundance_dt)
# round the matrix 
gene_abundance_matrix_rounded <- round(gene_abundance_matrix)
# convert to DESeq Data Set 
dds <- DESeqDataSetFromMatrix(countData = gene_abundance_matrix_rounded,
                              colData = metadata,
                              design = ~ notes)




#### DESeq analysis #####
dds <- DESeq(dds)
resultsNames(dds)
# save results to res
res <- results(dds)
# summarize results 
summary(res)

#### VISUALIZE ####
# let's visualize using a Volcano plot
p <- EnhancedVolcano(res,
                     lab = rownames(res),
                     x = 'log2FoldChange',
                     y = 'pvalue',
                     title = 'Volcano plot',
                     pCutoff = 0.05,
                     FCcutoff = 2,
                     pointSize = 1,
                     labSize = 2)
p

#### SAVE ####
# save as png
png("results/2025-04-22-salmon-emapper-volcano-plots.png", width=11, height=8.5, units = "in", res = 300)
print(p)
dev.off()
```

Now we are going to create a heatmap based on the sample counts of the transcripts to identify 1) how well the samples cluster together and 2) the variance of the top 100 genes.

```r
library("pheatmap")
library("RColorBrewer")

# rerun dds comparing depth and accounting for locational difference
dds <- DESeqDataSetFromMatrix(countData = gene_abundance_matrix_rounded,
                              colData = metadata,
                              design = ~ location + notes)



# DESeq analysiss
dds <- DESeq(dds)
# save results to res
res <- results(dds)
# summarize results 
summary(res)

# transform scale
vsd <- vst(dds, blind = T)

# plot PCA
plotPCA(vsd, intgroup=c("notes"), pcsToUse = 1:2)
plotPCA(vsd, intgroup=c("notes"), pcsToUse = 2:3)

# get the top most abundant transcripts
select <- order(rowMeans(counts(dds,normalized=FALSE)), decreasing=T)[1:10000]

# save the columns you want to compare
df <- as.data.frame(colData(dds)[,c("location","notes")])

# plot the heatmap
p <- pheatmap(assay(vsd)[select,], cluster_rows=TRUE, show_rownames=FALSE,
              cluster_cols=TRUE, annotation_col=df)
p

png("results/2026-04-22-deseq-heatmap-10000-most-abundant.png", width=8.5, height=11, units = "in", res = 300)
print(p)
dev.off()
```

###But, what if we only plot the top 100 genes that have the most variance from the mean?

```r
# transform scale
vsd <- vst(dds, blind = F)

# order by variance and get the top 100 genes
topVarGenes <- head(order(rowVars(assay(vsd)), decreasing = TRUE), 100)

# extract the information from the vsd file
mat  <- assay(vsd)[ topVarGenes, ]

# difference from the total mean
mat  <- mat - rowMeans(mat)

# get annotation information 
anno <- as.data.frame(colData(vsd)[, c("location","notes")])

# plot the data
p <- pheatmap(mat, annotation_col = anno)
p

png("results/2026-04-22-deseq-heatmap-100-most-variant.png", width=8.5, height=11, units = "in", res = 300)
print(p)
dev.off()
```

