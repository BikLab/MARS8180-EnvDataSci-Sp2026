# Overview of metabarcoding workflow
[MARS8180 Bioinformatics Workflows-2.pdf](https://github.com/user-attachments/files/25701411/MARS8180.Bioinformatics.Workflows-2.pdf)

# Before we get started

### Transferring Data files

All scripts, data, and metadata files for this section are hosted on the teaching cluster in the following file path: `/work/mars8180/instructor_data/metabarcoding-16S/` (you can always recopy this directory and start over if you make any mistakes on Sapelo2)

First, we will need to set up our directory structure on Sapelo2 in your `/home/myID/` directory, using the following commands:

1. Make an overall course folder in this location using the command `mkdir mars8180-course`
2. Next, move into that directory using `cd mars8180-course`
3. Now we will make a new folder for our script outputs using the command `mkdir analysis-results`
4. Next, we will copy three directories over from the teaching cluster using the `scp` file transfer command, as following:
5. `scp -r myid@teach.gacrc.uga.edu:/work/mars8180/instructor_data/metabarcoding-16S/scripts .`
6. `scp -r myid@teach.gacrc.uga.edu:/work/mars8180/instructor_data/metabarcoding-16S/metadata .`
7. `scp -r myid@teach.gacrc.uga.edu:/work/mars8180/instructor_data/metabarcoding-16S/raw-data .`

The `scp` command always needs an origin where the files are (the teaching cluster, in this instance) and a destination location (here, we specify ` . ` to indicate that the files should be copied to the current folder). The `-r` flag specifies that that it should recursively roll down the list of files and directories and copy everything contained within in the origin folder.

### Demultiplex Data 

After completing the above steps, your filepath for your raw data should be located on Sapelo2 at the following file path `/home/myID/mars8180-course/metabarcoding-16S/raw-data` - if you look in that directory 

The sequencing facility will give you your data as 1) multiplexed or 2) demultiplexed sequences. If your sequencing data is a multiplexed file, all of your sample sequencing data will be located in a single file. Demultiplexing is a process of seperating your samples using your unique sequencing barcodes. 

**If the sequencing facility demultiplexes your sequencing data, they will also send you the multiplexed raw sequencing data**

If you have multiplexed paired-end samples with barcodes in separate fastq file, you will typically recieve 4 files: 

1. Forward sequences (`project_L001_R1.fastq.gz`)
2. Reverse sequences (`project_L001_R2.fastq.gz`)
3. Forward barcodes (`barcodes_L001_I1.fastq.gz`)
4. Reverse barcodes (`barcodes_L001_I2.fastq.gz`)

Additionally, you can have multiplexed paired-end samples with barcodes within your sequences. In this case, you will have two sequencing files: 

1. Forward sequences (`project_L001_R1.fastq.gz`)
2. Reverse sequences (`project_L001_R2.fastq.gz`)

If you are working with single-end data you will receive the following files: 

1. A single sequences (`project_L001.fastq.gz`)
2. A single barcodes file (`barcodes.fastq.gz`) 

Regardless of the format, you will need to use the barcodes in the metadata file to indicate which barcodes are associated with which samples. **Note: Nowadays, the sequencing facility typically demultiplexes your sample free of cost. However, you must share your metadata and barcodes with the facility**

A typically metadata file with barcode(s) for paired-end sequence looks like this: 

 
|SampleID|BarcodeSequence|LinkerPrimerSequence|ReverseBarcode|
|--------|---------------|--------------------|--------------|
|sampleA |GCTAGCCTTCGTCGC|TATGGTAATTGTGTGYCAGCMGCCGCGGTAA|GATCGGGACACCCGA|
|sampleB |GCTAGCCTTCGTCGC|TATGGTAATTGTGTGYCAGCMGCCGCGGTAA|GATCTGTCTATACTA|
|sampleC |GCTCCTAACGGTCCA|TATGGTAATTGTGTGYCAGCMGCCGCGGTAA|GATAATAACTAGGGT|
|sampleD |GCTCGCGCCTTAAAC|TATGGTAATTGTGTGYCAGCMGCCGCGGTAA|GATTACGGATTATGG|
|sampleE |GCTACTACTGAGGAT|TATGGTAATTGTGTGYCAGCMGCCGCGGTAA|GATGTGGAGTCTCAT|


### What is a FASTQ file
A fastq file is a text file that stores information about the sequence and its quality score of each base. 

A typical fastq file looks like the following: 

```
@SRR2584863.1 HWI-ST957:244:H73TDADXX:1:1101:4712:2181/1
TTCACATCCTGACCATTCAGTTGAGCAAAATAGTTCTTCAGTGCCTGTTTAACCGAGTCACGCAGGGGTTTTTGGGTTACCTGATCCTGAGAGTTAACGGTAGAAACGGTCAGTACGTCAGAATTTACGCGTTGTTCGAACATAGTTCTG
+
CCCFFFFFGHHHHJIJJJJIJJJIIJJJJIIIJJGFIIIJEDDFEGGJIFHHJIJJDECCGGEGIIJFHFFFACD:BBBDDACCCCAA@@CA@C>C3>@5(8&>C:9?8+89<4(:83825C(:A#########################
```

1. The first line always starts with `@` and then information about the read - the instrument name, the coordinates of the flow cells, and the members of a pair. Typically, `/1` refers to a forward read and `/2` are the reverse reads. 
2. The second line is the raw sequences.
3. The third line is `+` character that seperates the sequences and sequence quality score.
4. The last line, is the sequence quality encoded by characters that represent a specific PHRED score.

From lowest (99.999999% probability of an error) to highest quality score (0.0001% probability of an error): 
```!"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHI```

Lets use the following command to get the first 8 lines of one of our metabarcoding samples. 

```
zcat raw-data/16S-BOBA-MM.B.3.2-1.fastq.gz | head -n 8
```

```
@VH00301:398:AAFT7FLM5:1:1101:23893:1095 1:N:0:CTTCAAGATTTC+ATGATACGTAAT
ATGCACGTCCCAGTATTGGTTTCAACATTGTGAAACTGTGAATTGCTCAATAAAACAGCTATTGCATATATGACTGCATATTTACATGGCTAGCCGTGGCAATTCTAGAGCTAATACATGTACGGAGCCTAACTTTGTGGGGAGGGTATTGTTTATTAGTTGTGGAACCAGTCCAGGTTATCCTTGGTTTCTTCTGATTGGTTGTAACTAAATGAATCGCATGGCATCAGTTGGTGGTGCATCATTCAAGCATCTGACCTATCAGCTTCCGACGGTAAGGTATTGGCCTACCTTGGCAATG
+
C5CCCCCCCCCCC*CCCCCCCCCCCCCCCCCCCCCCCC*CCCCCCCCCCCCCCCCCCCC5CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC5CCCCCCCCCCCC*CCCCCCCCCCCCCCCCCCCCCCCCCCCCC5CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC5CCCCCCCCCCCCCCCCCCCCCCCCCCCC5CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC5CCCCCCCCCCCCC5CCCCCCCCCCCCCC
@VH00301:398:AAFT7FLM5:1:1101:39761:1170 1:N:0:CTTCAAGATTTC+ATGATACGTAAT
GTGCATGTCTCAGTATAAGTGTTTCACTGCGAAACTGCGAATGGCTCATTAAAACAGTTATAGTTTCCATGTCAGTTGTTTATTACCTGGATATCCACGGTAATTCTAGAGCTAATACATGCGTCCAAACCCGACTTTTGCGGAAGGGTTGTGCTTATTAGACACTGAACCATCCCGGGCTTGCCCGGTTTCGAGGTGATTGATGGTAATCGAACGAATCGCATGCTTCGGCGGCGATGATTCATTCAAGTTTCTGACCTATCAGCTTCCGACGGTAGGGTATTGGCCTACCGTGGCTTTG
+
CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC5CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC*CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC*
```

If we look in our `raw-data` directory for the metabarcoding course folder, and `ls` the files, we can see that our class dataset is paired-end Illumina samples which have already been demultiplexed. In this case, you will see two FASTQ files for each sample, where the same sample ID (for example, `16S-BOBA-MM.B.3.2`) has two files associated with it which are the read pairs (`-1.fastq.gz` for forward Illumina reads and `-2.fastq.gz` for reverse Illumina reads).

1. Forward sequences (`16S-BOBA-MM.B.3.2-1.fastq.gz`)
2. Reverse sequences (`16S-BOBA-MM.B.3.2-2.fastq.gz`)


# 16S Metabarcoding Analysis 

### Import data 
Let's cat the first script `01-import-raw-data.sh`

```
$ cat scripts/01-import-raw-data.sh

#!/bin/sh

#SBATCH --job-name="import-data"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=12G
#SBATCH --time=01:00:00
#SBATCH --mail-user=userid@uga.edu
#SBATCH --mail-type=END,FAIL
#SBATCH -e 01-import-data.err-%N
#SBATCH -o 01-import-data.out-%N

# load module
module load QIIME2/2025.10-amplicon

# set paths to project directory and data subdirectory
INPUT="/work/mars8180/instructor_data/metabarcoding-16S/metadata/2026-01-15-BOBA-16S-fastq-manifest-file.txt"
OUTPUT="/work/mars8180/instructor_data/metabarcoding-16S/analysis/01-16S-rRNA-metabarcoding-data.qza"
export DATA="/work/mars8180/instructor_data/metabarcoding-16S/raw-data/"

# use qiime tools to import data
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path $INPUT \
  --output-path $OUTPUT \
  --input-format PairedEndFastqManifestPhred33V2
  ```

We are using a manifest file to import our data into a QIIME2 QZA format. The manifest file is a tab-delimited file located in the metadata directory with 3 columns, `sample-id`, `forward-absolute-filepath`, and `reverse-absolute-filepath`. We can use head to see the first 10 lines. 

```
$ head metadata/2026-01-15-BOBA-16S-fastq-manifest-file.txt

sample-id	forward-absolute-filepath	reverse-absolute-filepath
16S-BOBA-Blank.1	$DATA/16S-BOBA-Blank.1-1.fastq.gz	$DATA/16S-BOBA-Blank.1-2.fastq.gz
16S-BOBA-Blank.10	$DATA/16S-BOBA-Blank.10-1.fastq.gz	$DATA/16S-BOBA-Blank.10-2.fastq.gz
16S-BOBA-Blank.11	$DATA/16S-BOBA-Blank.11-1.fastq.gz	$DATA/16S-BOBA-Blank.11-2.fastq.gz
16S-BOBA-Blank.12	$DATA/16S-BOBA-Blank.12-1.fastq.gz	$DATA/16S-BOBA-Blank.12-2.fastq.gz
16S-BOBA-Blank.2	$DATA/16S-BOBA-Blank.2-1.fastq.gz	$DATA/16S-BOBA-Blank.2-2.fastq.gz
16S-BOBA-Blank.3	$DATA/16S-BOBA-Blank.3-1.fastq.gz	$DATA/16S-BOBA-Blank.3-2.fastq.gz
16S-BOBA-Blank.4	$DATA/16S-BOBA-Blank.4-1.fastq.gz	$DATA/16S-BOBA-Blank.4-2.fastq.gz
16S-BOBA-Blank.5	$DATA/16S-BOBA-Blank.5-1.fastq.gz	$DATA/16S-BOBA-Blank.5-2.fastq.gz
16S-BOBA-Blank.6	$DATA/16S-BOBA-Blank.6-1.fastq.gz	$DATA/16S-BOBA-Blank.6-2.fastq.gz
```

Notice that the filepath contains `$DATA`. This is an environment path that we define in the beginning of our script `export DATA="/work/mars8180/instructor_data/metabarcoding-16S/raw-data/"`

Let's update the paths in the beginning of the file to the correct aboslute paths that are located in your `mars8180-class-data` directory.

### Visualizing data
Now, we can run the second script `02-visualize-raw-data.sh` to visualize the data quality of the metabarcoding dataset. Once again, change the file paths so that they match the location in your directory. 

```
$ cat scripts/02-visualize-raw-data.sh

#!/bin/sh

#SBATCH --job-name="viz-data"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=12G
#SBATCH --time=01:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=END,FAIL
#SBATCH -e 02-visualize-data.err-%N
#SBATCH -o 02-visualize-data.out-%N

# load module
module load QIIME2/2025.10-amplicon

# set paths to project directory and data subdirectory
INPUT="/work/mars8180/instructor_data/metabarcoding-16S/analysis/01-16S-rRNA-metabarcoding-data.qza"
OUTPUT="/work/mars8180/instructor_data/metabarcoding-16S/analysis/02-16S-rRNA-visualize-data-quality.qzv"

# use qiime tools to import data
qiime demux summarize \
  --i-data ${INPUT} \
  --o-visualization ${OUTPUT}
```

In this script, the input file is the imported QIIME2 QZA filetype. This is the output of our first script `01-import-raw-data.sh`. The output is a QIIME2 QZV filetype. It's similar to QZA, except that it is made to visualize data using the QIIME2 Website: [https://view.qiime2.org](https://view.qiime2.org)

After you submit this job, you can download the output file to your personal computer using PuTTy or the CommandLine. We will go over the interactive quality plots in class. 

### Removing adapter sequences
Now, we will run script `03-remove-primers-adapters-cutadapt.sh` to remove primers from our dataset. This is an important part of our computational pipeline. 

```$ scripts 03-remove-primers-adapters-cutadapt.sh

#!/bin/sh

#SBATCH --job-name="cutadapt"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=12
#SBATCH --mem-per-cpu=5G
#SBATCH --time=01:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=END,FAIL
#SBATCH -e 03-remove-primers.err-%N
#SBATCH -o 03-remove-primers.out-%N

# load module
module load QIIME2/2025.10-amplicon

# set paths to project directory and data subdirectory
INPUT="/work/mars8180/instructor_data/metabarcoding-16S/analysis/01-16S-rRNA-metabarcoding-data.qza"
OUTPUT="/work/mars8180/instructor_data/metabarcoding-16S/analysis/03-16S-rRNA-sans-primers-adapters.qza"

# use qiime tools to import data
qiime cutadapt trim-paired \
  --i-demultiplexed-sequences ${INPUT}\
  --p-front-f GTGCCAGCMGCCGCGGTAA \
  --p-front-r GGACTACHVGGGTWTCTAAT \
  --p-error-rate 0.1 \
  --o-trimmed-sequences ${OUTPUT} \
  --p-cores 12
```

In this script, we specify the forward (5'-GTGCCAGCMGCCGCGGTAA-3') and reverse sequences (5'-GGACTACHVGGGTWTCTAAT-3') using the appropriate flags `--p-front-f` and `--p-front-r`. We also need to specify the output file for our trimmed sequences and the error rate (0.1 if the default error rate for the cutadapt software). 

Let's replace the `INPUT` and `OUTPUT` filepaths. The input is the raw data in the QIIME2 QZA format. 

Now, we can visualize the data to see if our data changed using the script `04-visualize-data-quality.sh`

### Desnoising sequences using DADA2 
DADA2 is a denoising algorithm that using estimated error-rates to correct and dereplicate your dataset. There are several denoising algorithims, but we prefer to use this because:
1. It estimates error rates based on the FASTQ PHRED scores and does not assume an equal error rate across runs. This is important because error-rates vary depending on the sample quality, library prep, and inclusion of sequencing controls. 
2. Other sequencing algorithms are too stringent and remove rare taxa that can impact downstream data analysis. 

Let's take a look at the script:

```
$ cat scripts/03-remove-primers-adapters-cutadapt.sh

#!/bin/sh

#SBATCH --job-name="denoise-asvs"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=12
#SBATCH --mem-per-cpu=5G
#SBATCH --time=01:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=END,FAIL
#SBATCH -e 04-denoise-asvs.err-%N
#SBATCH -o 04-denoise-asvs.out-%N

# load module
module load QIIME2/2025.10-amplicon

# set paths to project directory and data subdirectory
INPUT="/work/mars8180/instructor_data/metabarcoding-16S/analysis/03-16S-rRNA-sans-primers-adapters.qza"
OUTPUT="/work/mars8180/instructor_data/metabarcoding-16S/analysis"

# use qiime tools to import data
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs ${INPUT} \
  --p-trunc-len-f 220 \
  --p-trunc-len-r 235 \
  --o-representative-sequences ${OUTPUT}/05-16S-rRNA-denoise-dada2-rep-seq.qza \
  --o-table ${OUTPUT}/05-16S-rRNA-denoise-dada2-feature-table.qza \
  --o-denoising-stats ${OUTPUT}/05-16S-rRNA-denoise-dada2-stats.qza \
  --o-base-transition-stats ${OUTPUT}/05-16S-rRNA-denoise-dada2-transition-stats.qza \
  --p-n-threads 12
```

Based on the data quality plot, I have decided to truncate the forward and reverse reads at 235bp and 215bp, respectively. The quality at the 5' end was high for both the forward and reverse reads, so I will not trim the data. 

We will have three output files:
* `05-16S-rRNA-denoise-dada2-rep-seq.qza`: The representative sequences for each ASV
* `05-16S-rRNA-denoise-dada2-feature-table.qza`: A table with counts of how many times each ASV (row) was observed across each sample (column). 
* `05-16S-rRNA-denoise-dada2-stats.qza`: An in-depth comparison at how many reads were dropped at each step for DADA2. 
* `05-16S-rRNA-denoise-dada2-transition-stats.qza`; A table listing the transition rates of each ordered pair of nucleotides at each quality score.

### Assigning Taxonomy using BLAST+
Next, we are going to assign taxonomy using the BLAST+ software. Here, we will use the SILVA Ref Database clustered at 99% sequence similarity. There are many reference databases out there, and your choice of database will depend on several factors, such as : 
1. Your organism(s) of interested 
2. How robust the reference database is

Greengenes is another commonly used database for bacteria; however, it still has less unique sequences than the SILVA database impacting the classification of ASVs at lower taxonomic levels (family/genus).

Let's take a look at the script:

```
$ cat 06-assign-taxonomy-blast.sh

#!/bin/sh
#SBATCH --job-name="assign-taxonomy"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=12
#SBATCH --mem-per-cpu=4G
#SBATCH --time=3-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 06-assign-taxonomy.err-%N
#SBATCH -o 06-assign-taxonomy.out-%N

module load QIIME2/2025.10-amplicon

INPUT=/work/mars8180/instructor_data/metabarcoding-16S/analysis/05-16S-rRNA-denoise-dada2-rep-seq.qza
REFSEQ=/work/mars8180/instructor_data/metabarcoding-16S/database/SILVA138-nr99_sequences_16S.qza
REFTAX=/work/mars8180/instructor_data/metabarcoding-16S/database/SILVA138-nr99_taxonomy_16S.qza
OUTPUT=/work/mars8180/instructor_data/metabarcoding-16S/analysis/06-16S-rRNA-taxonomy-assignment-blast.qza
SEARCH=/work/mars8180/instructor_data/metabarcoding-16S/analysis/06-16S-rRNA-blast-search-results.qza

qiime feature-classifier classify-consensus-blast \
  --i-query ${INPUT} \
  --i-reference-taxonomy ${REFTAX} \
  --i-reference-reads ${REFSEQ} \
  --p-maxaccepts 5 \
  --p-perc-identity 0.90 \
  --o-classification ${OUTPUT} \
  --o-search-results ${SEARCH} \
  --p-num-threads 12
```

To run this script, we need 3 input files: 
* `05-16S-rRNA-denoise-dada2-rep-seq.qza`: The representative sequences for each ASV
* `SILVA138-nr99_sequences_16S.qza`: Representative sequences from reference database. 
* `SILVA138-nr99_taxonomy_16S.qza`: Taxonomoic labels of the representative sequences from reference database.

We will have two outputs:  
* `06-16S-rRNA-blast-search-results.qza`: Search results of the top X hits for each ASV query sequence. 
* `06-16S-rRNA-taxonomy-assignment-blast.qza`: The final taxonomic classification of the ASV query sequence.

### Constructing phylogenetic tree using the short-length ASV sequences
Building a phylogenetic tree has several steps: 
1. Create a sequence alignment using all ASVs
2. Mask (remove) any positions that are phylogenetically uninformative.
3. Use the masked alignment to contruct the phylogenetic tree
4. Root the tree 

There are several tools and pipelines you can use in QIIME2. Here, we will use the `align-to-tree-mafft-fasttree` pipeline, which uses **MAFFT** to align sequences and **fasttree** to approximate maximum-likelihood trees. Other tools are more time and computationaly expensive, but they yield higher quality trees. We will use fasttree in this class, since it is very fast. But, for your final analysis we recomend using IQ-TREE or RaXML. 

```
$ cat scripts/07-phylogeny-fasttree.sh

#!/bin/sh
#SBATCH --job-name="phylogeny"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=12
#SBATCH --mem-per-cpu=4G
#SBATCH --time=3-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 07-phylogenetic-tree.err-%N
#SBATCH -o 07-phylogenetic-tree.out-%N

module load QIIME2/2025.10-amplicon

INPUT=/work/mars8180/instructor_data/metabarcoding-16S/analysis/05-16S-rRNA-denoise-dada2-rep-seq.qza
OUTPUT=/work/mars8180/instructor_data/metabarcoding-16S/analysis

qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences ${INPUT} \
  --o-alignment ${OUTPUT}/07-16S-rRNA-fasttree-alignment.qza \
  --o-masked-alignment ${OUTPUT}/07-16S-rRNA-fasttree-masked-alignment.qza \
  --o-tree ${OUTPUT}/07-16S-rRNA-fasttree-unrooted-tree.qza \
  --o-rooted-tree ${OUTPUT}/07-16S-rRNA-fasttree-midrooted-tree.qza \
  --p-n-threads 12
```

### Export data into a format for R
QIIME2 files are zipped folders that contain the data and provinance information (what was done to the data). We need to export this to have filetypes that we can use to visualize and analyze in R. 

```
$ cat 08-extract-qiime2-data.sh

#!/bin/sh
#SBATCH --job-name="extract-data"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=12G
#SBATCH --time=01:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 08-extract-data.err-%N
#SBATCH -o 08-extract-data.out-%N

module load QIIME2/2024.10-amplicon

REPSEQ=/work/mars8180/instructor_data/metabarcoding-16S/analysis/05-16S-rRNA-denoise-dada2-rep-seq.qza
TABLE=/work/mars8180/instructor_data/metabarcoding-16S/analysis/05-16S-rRNA-denoise-dada2-feature-table.qza
TAXA=/work/mars8180/instructor_data/metabarcoding-16S/analysis/06-16S-rRNA-taxonomy-assignment-blast.qza
TREE=/work/mars8180/instructor_data/metabarcoding-16S/analysis/07-fasttree-midrooted-tree.qza
OUT=/work/mars8180/instructor_data/metabarcoding-16S/analysis/

module load QIIME2

qiime tools export \
  --input-path ${REPSEQ} \
  --output-path ${OUT}/08-representative-sequences

qiime tools export \
  --input-path ${TABLE} \
  --output-path ${OUT}/08-asv-table

qiime tools export \
  --input-path ${TAXA} \
  --output-path ${OUT}/08-taxonomy-classification

qiime tools export \
  --input-path ${TREE} \
  --output-path ${OUT}/08-midrooted-tree
```

## Downloading our data to our personal computer

Before we download our ASV table, taxonomy table, and tree onto our personal computer, we are going to create a projects directory on our personal laptop. On your desktop, create a folder called `metabarcoding-16` where we will store our QIIME2 files and our downstream data analysis. 

The file path will vary for each user and computing system, so make sure you are using the file path on your computer. 

```
$ cd /Users/userid/Desktop
$ mkdir metabarcoding-16S
$ cd metabarcoding-16S
$ mkdir results data scripts metadata
```

Now, we can download our data using the scp command.

**Download the metadata (`2026-02-25-16S-rRNA-metadata.txt`)** 

```
scp userid@txfer.gacrc.uga.edu:/work/mars8180/instructor_data/metabarcoding-16S/metadata/2026-02-25-16S-rRNA-metadata.txt metadata/
```

**Download the ASV Table (`05-16S-rRNA-denoise-dada2-feature-table.qza`)** 

```
scp userid@txfer.gacrc.uga.edu:/path/to/directory/05-16S-rRNA-denoise-dada2-feature-table.qza results/
```

**Download the Taxonomy Table (`06-16S-rRNA-taxonomy-assignment-blast.qza`)** 

```
scp userid@txfer.gacrc.uga.edu:/path/to/directory/06-16S-rRNA-taxonomy-assignment-blast.qza results/
```

**Download the Phylogenetic Tree (`07-16S-rRNA-fastree-midrooted-tree.qza`)** 

```
scp userid@txfer.gacrc.uga.edu:/path/to/directory/07-16S-rRNA-fastree-midrooted-tree.qza results/
```

## Starting an R project

Now, we can start an R project in the directory we just created.

Generally, for metabarcoding datasets we will need to work with **four** different files:
1. **ASV table** - Our table containing Amplicon Sequence Variants, generated in DADA2 (where each row is an ASV, and columns are our sample names showing the presence/absence (and read counts if present) of each ASV across each sample).
2. **Taxonomy assignments** - a list of database matches (e.g. from BLAST), or classifier results (e.g. from the QIIME2 classifier) for each row (ASV) in your ASV table
3. **Metadata mapping file** - your tab-delimited (or excel) table with your sample names and all the corresponding sample site data, environmental metadata, etc. (e.g. depth, geographic coordinates, sampling date, experimental condition, etc.)
4. **Phylogenetic tree** - for 16S or 18S rRNA metabarcoding datasets, you will build a phylogenetic tree showing evolutionary relationships using a multiple sequence alignment of your ASV nucleotide sequences. Many downstream diversity stats will use "phylogenetic distance" (total branch length between two ASVs on a tree) as a metric within the mathematical equation. A tree is not needed for all analyses, and cannot be generated for some metabarcoding loci (e.g. COI, mitochondrial loci evolve too fast and trees are generally meaningless except perhaps for population-level analysis within one species)

Let's install and import the R packages we are going to use for our downstream analysis. **QIIME2R** allows us to easily import the artifact files into a "phyloseq" object. **Phyloseq** let's us manipulate our metabarcoding dataset. **Decontam** is a package that allows us to use our blank to remove potential contaminants. **Tidyr** and **ggplot** allow us to easily manipulate dataframes and create publication ready plots, respectively.

