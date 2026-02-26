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
