# Overview of metabarcoding workflow
<img width="1084" height="621" alt="Screenshot 2026-03-02 at 10 07 16 PM" src="https://github.com/user-attachments/assets/3087a42c-961c-466b-a790-5934b2a0afd4" />

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

---

```
if (!requireNamespace("devtools", quietly = TRUE)){install.packages("devtools")}

devtools::install_github("jbisanz/qiime2R") # install qiime2R 
BiocManager::install("decontam") # install decontam
BiocManager::install("phyloseq") # install phyloseq
BiocManager::install("ggplot2") # install ggplot2
BiocManager::install("tidyr") # install tidyr

library(phyloseq)
library(decontam)
library(qiime2R)
library(tidyr)
library(ggplot2)
```

**QIIME2 resources:
**

* The very extensive QIIME2 documetnation is available here: https://amplicon-docs.qiime2.org/en/stable/ (lots of tutorials, and an active user forum where you can post questions if you get really stumped)

* Information on other available QIIME2 plugins is located here: https://docs.qiime2.org/2024.10/plugins/available/index.html (lots of plugins we won't have time to utilize in this course)

---

## Importing our data into phyloseq object
We are going to read artifact files (.qza) using qiime2R and import our metadata file using the `read.delim2` command.


``` r
asvs <- read_qza("results/05-dada2-feature-table.qza")
taxonomy <- read_qza("results/06-taxonomy-blast-90-1.qza")
tree <- read_qza("results/07-fasttree-midrooted-tree.qza")
metadata <- read.delim2("metadata/2025-01-03-ddt-metadata.csv", sep = ",", row.names = 1)
```

QZA files are zipped folders with many different pieces of information
including data provenance, format, version, and the data. We need to
extract out data from this file type

``` r
asv_df <- asvs$data # we can view and save the actual data by specifying '$'
taxonomy_df <- taxonomy$data # save taxonomy info as a dataframe
phylo_tree <- tree$data # tree data is stored as a phyloseq object
```

The taxonomy file is not formatted correctly. All the taxonomy is in one
column. Ideally, each taxonomic level should have its own column.

``` r
head(taxonomy_df) 
```

```
                        Feature.ID
1 0005b35778658ed77217967b57fdd319
2 0006630fb18ffea20c1bd1c0227f22f3
3 000fb3ff10896c00c9c11d750da7a164
4 0013dd4bef6114837fdf8c29ee55ea50
5 0017c06698f0b928e93ffd3fd8498011
6 001b7c34dad08db30819fdf95a704b3e
                                                                                                                                                                                                                    Taxon
1                                                                                                      D_0__Eukaryota;D_1__Amorphea;D_2__Amoebozoa;D_3__Incertae Sedis;D_4__Apusomonadidae;D_5__uncultured Apusomonadidae
2                                                                                                                D_0__Eukaryota;D_1__SAR;D_2__Rhizaria;D_3__Cercozoa;D_4__Novel Clade 12;D_5__uncultured marine eukaryote
3                                                                                                 D_0__Eukaryota;D_1__SAR;D_2__Rhizaria;D_3__Retaria;D_4__Polycystinea;D_5__Collodaria;D_6__AT8-54;D_7__marine metagenome
4 D_0__Eukaryota;D_1__Amorphea;D_2__Obazoa;D_3__Opisthokonta;D_4__Nucletmycea;D_5__Fungi;D_6__Dikarya;D_7__Basidiomycota;D_8__Agaricomycotina;D_9__Agaricomycetes;D_10__Agaricales;D_11__Chamaeota;D_12__Chamaeota sinica
5                                                                                           D_0__Eukaryota;D_1__SAR;D_2__Alveolata;D_3__Protalveolata;D_4__Syndiniales;D_5__Syndiniales Group I;D_6__uncultured eukaryote
6                                                                                                                                                                                                              Unassigned
  Consensus
1         1
2         1
3         1
4         1
5         1
6         1
```

We are going to split the taxonomy into different columns and replace NA’s with with a placeholder: 'Unassigned'

``` r
taxonomy_fixed_df <- taxonomy_df %>% separate_wider_delim(Taxon, delim = ";", names_sep = "", too_few = "align_start")
taxonomy_fixed_df[is.na(taxonomy_fixed_df)] <- "Unassigned" # rename NAs into unassigned
head(taxonomy_fixed_df)
```

```
# A tibble: 6 × 25
  Feature.ID       Taxon1 Taxon2 Taxon3 Taxon4 Taxon5 Taxon6 Taxon7 Taxon8 Taxon9 Taxon10 Taxon11
  <chr>            <chr>  <chr>  <chr>  <chr>  <chr>  <chr>  <chr>  <chr>  <chr>  <chr>   <chr>  
1 0005b35778658ed… D_0__… D_1__… D_2__… D_3__… D_4__… D_5__… Unass… Unass… Unass… Unassi… Unassi…
2 0006630fb18ffea… D_0__… D_1__… D_2__… D_3__… D_4__… D_5__… Unass… Unass… Unass… Unassi… Unassi…
3 000fb3ff10896c0… D_0__… D_1__… D_2__… D_3__… D_4__… D_5__… D_6__… D_7__… Unass… Unassi… Unassi…
4 0013dd4bef61148… D_0__… D_1__… D_2__… D_3__… D_4__… D_5__… D_6__… D_7__… D_8__… D_9__A… D_10__…
5 0017c06698f0b92… D_0__… D_1__… D_2__… D_3__… D_4__… D_5__… D_6__… Unass… Unass… Unassi… Unassi…
6 001b7c34dad08db… Unass… Unass… Unass… Unass… Unass… Unass… Unass… Unass… Unass… Unassi… Unassi…
# ℹ 13 more variables: Taxon12 <chr>, Taxon13 <chr>, Taxon14 <chr>, Taxon15 <chr>,
#   Taxon16 <chr>, Taxon17 <chr>, Taxon18 <chr>, Taxon19 <chr>, Taxon20 <chr>, Taxon21 <chr>,
#   Taxon22 <chr>, Taxon23 <chr>, Consensus <dbl>
```

Let's force our taxonomy table back into a matrix datatype. 

``` r
taxonomy_fixed_df <- as.data.frame(taxonomy_fixed_df) # force into a dataframe
row.names(taxonomy_fixed_df) <- taxonomy_fixed_df$Feature.ID # make first column into row names
taxonomy_fixed_df$Feature.ID <- NULL # remove the first column
taxonomy_matrix <- as.matrix(taxonomy_fixed_df) # convert to a matrix 
```

Now we can merge our otu table, taxonomy file, and tree into a phyloseq object
```
physeq_asv <- otu_table(asv_df, taxa_are_rows = T) # convert into phyloseq object
physeq_tax <- tax_table(taxonomy_matrix) # convert into phyloseq object
physeq_meta <- sample_data(metadata) # convert into phyloseq object

phylo_object <- phyloseq(physeq_asv, physeq_tax, physeq_meta) # merge into phyloseq object
phylo_object_tree <- merge_phyloseq(phylo_object, phylo_tree) # add tree into phyloseq object
```

If we type in the phyloseq obect in our console, we can get a summary of our data. 

``` r
phylo_object_tree
```

```
phyloseq-class experiment-level object
otu_table()   OTU Table:         [ 24188 taxa and 208 samples ]
sample_data() Sample Data:       [ 208 samples by 43 sample variables ]
tax_table()   Taxonomy Table:    [ 24188 taxa by 24 taxonomic ranks ]
phy_tree()    Phylogenetic Tree: [ 24188 tips and 24167 internal nodes ]
```
---
#### Removing Contaminant Sequences

For current best practices and guidance regarding contamination in -Omics datasets, the following paper provides a very comprehensive overview of all the things you need to be thinking about:

* Fierer et al. (2025) Guidelines for preventing and reporting contamination in low biomass studies. _Nature Microbiology_, 10: 1570-1580. https://www.nature.com/articles/s41564-025-02035-2

And here is another important early paper raising awareness of the "kit microbiome" and potential contamination in -Omics sequencing:

* Salter et al. (2014) Reagent and laboratory contamination can critically impact sequence-based microbiome analyses, *BMC Biology*, 12:87 - https://bmcbiol.biomedcentral.com/articles/10.1186/s12915-014-0087-z

<img width="892" alt="Screenshot 2025-02-11 at 9 51 19 AM" src="https://github.com/user-attachments/assets/74b2184c-fee3-4f4c-9413-ffb2265af029" />

---
How contaminant removal works - below figure is from Davis et al. (2018) Simple statistical identification and removal of contaminant sequences in marker-gene and metagenomics data, *Microbiome* - https://microbiomejournal.biomedcentral.com/articles/10.1186/s40168-018-0605-2 (this is the software paper you should use to cite decontam)

<img width="762" alt="Screenshot 2025-02-06 at 9 09 16 AM" src="https://github.com/user-attachments/assets/71a216ca-2e93-4a71-aa68-a555bcf2e50a" />

A nice introduction to the decontam software tool is also available here: https://benjjneb.github.io/decontam/vignettes/decontam_intro.html

---****


## Filter out contaminatns using Decontam’s prevelance method. For more information see <https://github.com/benjjneb/decontam>

As we've stressing throughout this course, we need to make sure we have a high-quality dataset. First, we can use our blanks to identify potential contaminations using a package called decontam. Let's set the prevalence theshold to to stricter value (0.5). For more information on parameters see the decontam manual

``` r
sample_data(phylo_object_tree)$is.neg <- sample_data(phylo_object_tree)$Sample_Control == "Control" # create a sample-variable for contaminants
phylo_object_contaminants <- isContaminant(phylo_object_tree, method = "prevalence", neg="is.neg", threshold=0.6, detailed = TRUE, normalize = TRUE) # detect contaminants based on control samples and their ASV prevalance
table(phylo_object_contaminants$contaminant) # check number of ASVs that are contaminents
```
``` 
##  FALSE   TRUE 
## 190197    632
```
Now, we are going to make a presence-absence table of the contaminants in controls and samples. 

``` r
# Make phyloseq object of presence-absence in negative controls and true samples
phylo_object_contaminants.pa <- transform_sample_counts(phylo_object_tree, function(abund) 1 * (abund > 0)) # convert phyloseq table to presence-absence
ps.pa.neg <- subset_samples(phylo_object_contaminants.pa, Sample_Control=="Control")
ps.pa.pos <- subset_samples(phylo_object_contaminants.pa, Sample_Control=="Sample")
df.pa <- data.frame(pa.pos=taxa_sums(ps.pa.pos), pa.neg=taxa_sums(ps.pa.neg), contaminant=phylo_object_contaminants$contaminant) # convert into a dataframe
```

Let's plot our prevalance of ASVs in our blanks and compare then to our real samples. We see a clear split between prevalence in true samples vs controls.

``` r
# Make phyloseq object of presence-absence in negative controls and true samples
ggplot(data=df.pa, aes(x=pa.neg, y=pa.pos, color=contaminant)) + geom_point() + xlab("Prevalence (Negative Controls)") + ylab("Prevalence (True Samples)")
```

Now, we can filter out these contaminants from our dataset.


``` r
phylo_obj_tree_sans_contam <- prune_taxa(!phylo_object_contaminants$contaminant, phylo_object_tree) # remove ASVs identified as decontaminants from the dataset
phylo_obj_tree_sans_contam_sans_controls <- subset_samples(phylo_obj_tree_sans_contam, Sample_Control != "Control") ## Remove blanks and positive controls
phylo_obj_tree_sans_contam_sans_controls
```

```
phyloseq-class experiment-level object
otu_table()   OTU Table:         [ 16568 taxa and 202 samples ]
sample_data() Sample Data:       [ 202 samples by 43 sample variables ]
tax_table()   Taxonomy Table:    [ 16568 taxa by 24 taxonomic ranks ]
phy_tree()    Phylogenetic Tree: [ 16568 tips and 16563 internal nodes ]
```
---

## Generating taxonomy barplots

Taxonomy barplots are a common way to visualize our community composition. We can do this in phyloseq, which wraps ggplot2, to create these visualizations. That means we can use the ggplot2 language to further enchange our graphs. 

We will focus on visualizing the community structure of the marine nematodes since we have well-curated strings. So first, we need to subset our dataframe to only include nematodes and then transform/normalize our counts to relative abundance. 

```
phylo_obj_tree_sans_contam_nematoda <- subset_taxa(phylo_obj_tree_sans_contam, Taxon14=="D_13__Nematoda") # subset for marine nematodes
phylo_obj_tree_sans_contam_nematoda_ra <- transform_sample_counts(phylo_obj_tree_sans_contam_nematoda, function(x) x / sum(x)) # convert to relative abundance
```

Now, we can plot using the `plot_bar` command. I will include other ggplot functions to make sure our figure looks good. This includes `facet_grid` which will separate our samples by Site and the flag `free-scales` will not plot samples in the facet if they do not belong to the Site.  If you want to see how the command reacts without these parameters, remove them :). 

```
plot_bar(phylo_obj_tree_sans_contam_nematoda_ra, "Sample", fill="Taxon17") + # Sample is x-axis
  geom_bar(aes(color=Taxon17), stat="identity") + # specifies the type of graph and color
  facet_grid(~Site, scales = "free", space = "free_x") # lets separate our samples by Site
```

Our titles are too long and unreadable, so lets edit them to make this publishable. First, lets create a dictionary and create "long" form labels.

```
long_form_labels <- c(
  DDT_Barrel_Site_1_North = "DDT Barrels North",
  DDT_Barrel_Site_2_South = "DDT Barrels South",
  Patton_Ridge_South = "Patton Ridge",
  `40Mile_Bank` = "40Mile Bank",
  Lasuen_Knoll = "Lasuen Knoll",
  San_Diego_Trough = "San Diego Trough",
  no.data = "controls")
```

Now, we need to match them to each samples and save them to a new column

```
idx <- match(sample_data(phylo_obj_tree_sans_contam_nematoda_ra)$Site, names(long_form_labels)) # match to each Sample
sample_data(phylo_obj_tree_sans_contam_nematoda_ra)$Site_long <- long_form_labels[idx] # create a new column in our phyloseq metadata
```

Finally, we can plot our data so its readable. The `labeller` flag in the facet command will create line breaks. The `theme()` command allows us to change the font sizes and position. 

```
plot_bar(phylo_obj_tree_sans_contam_nematoda_ra, "Sample", fill="Taxon17") +
  geom_bar(aes(color=Taxon17), stat="identity") +
  facet_grid(~Site_long, scales = "free", space = "free_x", labeller = labeller(Site_long = label_wrap_gen(1))) + 
  theme_classic() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1, size = 4),
        strip.text.x = element_text(size = 8, angle = 90))
```

## Alpha Diversity, Rarefaction, and Normalization

Key Papers:
* Willis AD (2019) Rarefaction, Alpha Diversity, and Statistics, *Frontiers in Microbiology* -https://www.frontiersin.org/journals/microbiology/articles/10.3389/fmicb.2019.02407/full
* McMurdie, P. J., and Holmes, S. (2014). Waste not, want not: why rarefying microbiome data is inadmissible. PLoS Comput. Biol. 10:e1003531 - https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1003531

Alpha diversity metrics summarize the structure of an ecological community with respect to its richness (number of taxonomic groups), evenness (distribution of abundances of the groups), or both. Because many perturbations to a community affect the alpha diversity of a community, summarizing and comparing community structure via alpha diversity is a ubiquitous approach to analyzing community surveys. In microbial ecology, analyzing the alpha diversity of amplicon sequencing data is a common first approach to assessing differences between environments. (Willis 2019 - see paper link above)

**Rarefaction** - Subsampling your overall dataset, by selecting X number of observations (e.g. ASVs) from each sample, regardless of the overall sequencing effort. For example, if you have two sample sites, Beach A (1000 ASVs and 1 million sequence reads), and Beach B (250 ASVs and 200,000 sequencing reads), you could rarify this dataset by randomly subsampling 100 ASVs from each sample site. Rarefaction is a common paremeter you will see in the scripts/code we run in this class. **Note: Rarefaction can be contentious in eDNA studies, but many people still use it - see above literature refs**

**Normalization** - Using proportional abundances of ASVs, instead of the count of sequence reads for that ASV. To normalize a metabarcoding dataset, you typically divide the raw sequence counts for each ASV in a sample by the total number of sequences in that sample. For example, at our sample site Beach A, ASV_347 has an absolute abundance of 50,000 sequence reads. If Beach A has 1 million sequence reads overall, then the normailized abundance of ASV_347 would be 50,000/1,000,000 --> 0.05 or 5% 

Something to think about (annotated image from Willis et al. 2019): 

<img width="1085" alt="Screenshot 2025-02-13 at 10 15 42 AM" src="https://github.com/user-attachments/assets/4230fd18-b62d-4305-9732-0b0bd758b90f" />

---

## Calculating Alpha Diversity

Key Alpha Diversity metrics and visualizations:
* Taxonomy bar charts
* Species Richness
* Evenness
* Chao1, Shannon, Simpson Diversity Indexes

Now that we have created our phyloseq object, we are going to calculate Observed ASVS - a simple measure of alpha diversity. We can do this by using the `phyloseq::estimate_richness` function. We usually do not need to to specify the package, but this might be useful if you are using packages that have functions with the same name. 

```
alpha_div_observed <- phyloseq::estimate_richness(phylo_obj_tree_sans_contam, measures = "Observed") # calculate alpha diversity
head(alpha_div_observed) # view the first few lines of our R object
```

```
         Observed
DDT.1.1       356
DDT.1.2       384
DDT.10.1      352
DDT.10.2      406
DDT.11.1      473
DDT.11.2      480
```

We were able to calculate diverstiy, but we lost our metadata associated with our samples. However, we can access this metadata using the `sample_data` function.

```
metadata_df <- phyloseq::sample_data(phylo_obj_tree_sans_contam) # access metadata from the phyloseq object
head(metadata_df) 
```

We can put this together to calculate the observed diversity (number of ASVs) and create a dataframe that includes our metadata.

```
alpha_div_observed_metadata <- data.frame( # save as a dataframe
  phyloseq::sample_data(phylo_obj_tree_sans_contam), # access metadata 
  "Observed" = phyloseq::estimate_richness(phylo_obj_tree_sans_contam, measures = "Observed")) # calculate Observed diversity and save it to a column names "Observed"
```  

Now we can use ggplot to plot our results

```
alpha_div_observed_plot <- ggplot(alpha_div_observed_metadata, aes(x=Barrel_ID, y=Observed)) + # what are we plotting
  geom_boxplot() + # make it into a boxplot plot
  theme_minimal() # and lets make sure its "pretty"  
```

We can modify the axis labels to make it more readable. We are going to tilt the labels to 45 degrees, change the font size to 8, and adjust the horizontal justification

```
alpha_div_observed_plot <- ggplot(alpha_div_observed_metadata, aes(x=Site_Area, y=Observed)) + # what are we plotting
  geom_boxplot() + # make it into a boxplot plot
  theme_minimal() + # and lets make sure its "pretty"
  theme(axis.text.x = element_text(angle = 45, hjust = 1, size = 8)) # change x-axis labels
```

## Calculating Several Alpha Diversity Metrics
Lets do the same by calculating alpha diversity using commonly used metrics, such as number of reads, shannon diversity, simposon, and eveness.

```
alpha_div_observed_metadata <- data.frame(
  phyloseq::sample_data(phylo_obj_tree_sans_contam), # get metadata
  "Reads" = phyloseq::sample_sums(phylo_obj_tree_sans_contam), # number of reads
  "Observed" = phyloseq::estimate_richness(phylo_obj_tree_sans_contam, measures = "Observed"), # count observed ASVs
  "Shannon" = phyloseq::estimate_richness(phylo_obj_tree_sans_contam, measures = "Shannon"), # Calculate Shannon Diversity
  "InvSimpson" = phyloseq::estimate_richness(phylo_obj_tree_sans_contam, measures = "InvSimpson")) # calculate InvSimpson
```

Now lets calculate Eveness by dividing Shannon by the Log of the Observed ASVs

```
alpha_div_observed_metadata$Evenness <- alpha_div_observed_metadata$Shannon/log(alpha_div_observed_metadata$Observed)
head(alpha_div_observed_metadata)
```

## Beta Diversity - Background and Resources

Key Beta Diversity metrics and visualizations:
* Bray-Curtis - considers ASV presence/absence AND abundance, gives more consideration to abundant ASVs
* Jaccard - only consideres ASV presence/absence
* Canberra - simliar to Bray-Curtis, but gives more consideration to rare taxa (low abundance ASVs) by treating all ASVs equally
* Unifrac (weighted/unweighted) - take phylogenetic distance into account, can be based on presence/absence of ASVs only (unweighted - recommended), or also additionally incorporate normalized abundances (weighted - less commonly used)
* nMDS plots

If you really want to get into the weeds with some of these metrics, here are some recommended resources:
* Roberts DW (2017) Distance, dissimiliarity, and mean-variance ratios in ordination, Methods in Ecology and Evolution, 8:1398-1407 - https://besjournals.onlinelibrary.wiley.com/doi/pdf/10.1111/2041-210X.12739
* QIIME2 online documentation also has some pretty good explanations of different metrics (alpha/beta diversity) - https://docs.qiime2.org/
* See also some of the phyloseq tutorials (with links to key literature) - https://joey711.github.io/phyloseq/

## Ordination Crash Course (class request)

A really good overview of ordination types - https://uw.pressbooks.pub/appliedmultivariatestatistics/chapter/types-of-ordination-methods/ (**NOTE:** This whole book is freely available online, and highly recommended as a deep dive - Baker (2024) Applied Multivariate Statistics in R - https://uw.pressbooks.pub/appliedmultivariatestatistics/)

Ordination is a _**"graphical representation of the similiarity of sampling units and/or attribues in resemblance space"**_ (Wild 2010, pg. 35), quote from Baker textbook

All ordination methods are aimed at **identifying hidden patterns** in a dataset and **reducing the complexity of high-dimensional data**. This is often done by carrying out complex mathematical calculations and comparison of samples, and then "squishing" this data down into a 2D plot, by maximizing the usefulness of the visualization (e.g. choosing the aspect of the data which differentiates sample groupings the most, such as a PCoA axis).  

If you want to work through some ordination examples with R code, here is a great workshop (code + stats explanations): https://r.qcbs.ca/workshop09/book-en/index.html 

There are many types of ordination methods, including (but not limited to):
* **Principal Component Analysis (PCA)** - "a statistical technique used to reduce the dimensionality of a dataset while retaining most of its variability. It is a linear transformation method that converts the original set of variables into a new set of linearly uncorrelated variables, called principal components (PCs), which are sorted in decreasing order of variance." It primarily uses Euclidian distance - https://r.qcbs.ca/workshop09/book-en/principal-component-analysis.html (**NOTE:** in QIIME2 you choose the distance matrix used to build your PCoAs, such as Bray-Curtis similarity or the Unifrac metric which measures phylogenetic distance betwen samples/species). 
* **Correspondence Analysis (CA)** - "preserves Chi2 distances" and may be a better option for data with "long ecological gradients", good for when underlying assumptions of PCA are violated by the dataset characteristics - https://r.qcbs.ca/workshop09/book-en/correspondence-analysis.html
* **Principal Coordinates Analysis (PCoA)** - an "unconstrained ordination" where "points are added to plane space one at a time using Euclidean distance (or whatever distance (dissimilarity) metric you choose)" - https://r.qcbs.ca/workshop09/book-en/principal-coordinates-analysis.html
* **Nonmetric Multidimentional Scaling (NMDS)** - in NMDS, "the priority is not to preserve the exact distances among sites, but rather to represent as accurately as possible the relationships among objects in a small and number of axes (generally two or three) specified by the user", and so "the biplot produced from NMDS is the better 2D graphical representation of between-objects similarity: dissimilar objects are far apart in the ordination space and similar objects close to one another" - https://r.qcbs.ca/workshop09/book-en/nonmetric-multidimensional-scaling.html
* Other ordination types mentioned in class: **Detrended Correspondence Analysis (DCA)**, **ReDundancy Analysis (RDA)**, **Distance-based Redundancy Analysis (db-RDA)** also known as **Canonical Analysis of Principal Coordinates (CAP)** - see Baker texbook (above link) for great explanations of all of these and more!

*Choosing between PCA and PCoA can be tricky, but generally PCA is used to summarize multivariate data into as few dimensions as possible, whereas PCoA can be used to visualize distances between points. PCoA can be particularly suited for datasets that have more columns than rows. For example, if hundreds of species have been observed over a set of quadrats, then a approach based on a PCoA using Bray-Curtis similarity may be best suited.* (Quote from: https://r.qcbs.ca/workshop09/book-en/principal-coordinates-analysis.html) 

Goodrich et al. (2016) Conducting a Microbiome Study, Cell, 158(2):250-262 - [https://pmc.ncbi.nlm.nih.gov/articles/PMC5074386/](https://doi.org/10.1016/j.cell.2014.06.037) 

![1-s2 0-S0092867414008642-gr4_lrg](https://github.com/user-attachments/assets/f5208b11-acc8-4473-9578-1bccd0c8e700)



## Calculating Beta Diversity Metrics

We will calculate the beta-diversity using the bray-curtis metric. 

First, let's normalize our dataset by relative abundance. 

```
phylo_obj_tree_sans_contam_sans_controls_relab <- transform_sample_counts(phylo_obj_tree_sans_contam_sans_controls, function(x) x / sum(x) )
```


```
bray_dist <- phyloseq::distance(phylo_obj_tree_sans_contam_sans_controls_relab, method="bray") # calculate bray-curtis metric
ordination <- ordinate(phylo_obj_tree_sans_contam_sans_controls_relab, method="PCoA", distance=bray_dist) # Perform ordination using bray-curtis metric
plot_ordination(phylo_obj_tree_sans_contam_sans_controls_relab, ordination, color="Site") + theme(aspect.ratio=1) # plot the ordination using ggplot2
```

We can get a list of the ordination methods available in phyloseq by requesting the help menu `??distanceMethodList`. Furthermore, we can test whether the sites differ significantly from each other using the permutational ANOVA (PERMANOVA) analysis:

```
adonis2(bray_dist ~ sample_data(phylo_obj_tree_sans_contam_sans_controls)$Site)
