<img width="1221" height="645" alt="Screenshot 2026-03-02 at 10 08 33 PM" src="https://github.com/user-attachments/assets/52d367b7-b700-4f90-a1c2-f21cdb7953ab" />

# Before we get started

### Transferring Data files

All scripts, data, and metadata files for this section are hosted on the teaching cluster in the following file path: `/work/mars8180/instructor_data/metagenomics/` (you can always recopy this directory and start over if you make any mistakes on Sapelo2)

First, we will need to set up our directory structure on Sapelo2 in your `/home/myID/mars8180-course` directory, using the following commands:

1. Change directories `cd /home/myID/mars8180-course`
2. Make a directory for the metagenomics sections `mkdir metagenomics`
3. Change directories `cd metagenomics`
3. Now we will make a new folder for our outputs using the command `mkdir analysis`
4. Next, we will copy three directories (scripts, databases, and raw-data) over from the teaching cluster using the `scp` file transfer command, as following:
5. `scp -r myid@teach.gacrc.uga.edu:/work/mars8180/instructor_data/metagenomics/database .`
6. `scp -r myid@teach.gacrc.uga.edu:/work/mars8180/instructor_data/metagenomics/raw-data .`
7. `scp -r myid@teach.gacrc.uga.edu:/work/mars8180/instructor_data/metagenomics/scripts .`

The `scp` command always needs an origin where the files are (the teaching cluster, in this instance) and a destination location (here, we specify ` . ` to indicate that the files should be copied to the current folder). The `-r` flag specifies that that it should recursively roll down the list of files and directories and copy everything contained within in the origin folder.

After completing the above steps, your filepath for your raw data should be located on Sapelo2 at the following file path `/home/myID/mars8180-course/metagenomics/raw-data`. 

### Visualizing data quality
Now, we can run two scripts `01-fastqc-raw-data.sh` and `02-multiqc-raw-data.sh` to visualize data quality of the metagenomics dataset. Before you run the scripts, change the file paths so that they match the location in your directory. 

```
$ cat scripts/01-fastqc-raw-data.sh

#!/bin/bash

#SBATCH --job-name="fastqc"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem-per-cpu=8G
#SBATCH --time=7-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 01-fastqc.err-%N
#SBATCH -o 01-fastqc.out-%N

# path variables and modules
module load FastQC/0.12.1-Java-11

INPUT=/work/mars8180/instructor_data/metagenomics/raw-data
OUTPUT=/work/mars8180/instructor_data/metagenomics/analysis/01-fastqc

# single assembly
mkdir ${OUTPUT}
fastqc -o ${OUTPUT} -t 8 ${INPUT}/*
```
In this script, the input file are the raw data files located within the `$INPUT` directory: `/work/mars8180/instructor_data/metagenomics/raw-data`. The output will be a folder called 01-fastqc in the analysis subdirectory. The script will generate a quality report for each file. This can be difficult to analyse individualize, so we will use a second script `02-multiqc-raw-data.sh` to collate all the reports onto a single file. 

```
$ cat scripts/02-multiqc-raw-data.sh

#!/bin/bash

#SBATCH --job-name="multiqc"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=2
#SBATCH --mem-per-cpu=12G
#SBATCH --time=7-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 02-multiqc.err-%N
#SBATCH -o 02-multiqc.out-%N

# path variables and modules
module load MultiQC/1.28-foss-2024a

INPUT=/work/mars8180/instructor_data/metagenomics/analysis/01-fastqc
OUTPUT=/work/mars8180/instructor_data/metagenomics/analysis/02-multiqc

# single assembly
mkdir ${OUTPUT}
multiqc -o ${OUTPUT} ${INPUT}
```

The input is the folder where all the fastqc reports are generated. The output is a folder that contains the html file with the multiqc report summary. After you submit this job, you can download the output file to your personal computer using PuTTy or the CommandLine. We will go over the interactive quality plots in class. 

### Quality and Adapter trimming 
Now, we can use a tool, Trimmomatic, to quality trim our reads and remove adapters sequences. 


```
$ cat scripts/03-trimmomatic.sh

#!/bin/sh
#SBATCH --job-name="trimmomatic"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=12
#SBATCH --mem-per-cpu=5G
#SBATCH --time=1-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 03-trimmomatic.err-%N
#SBATCH -o 03-trimmomatics.out-%N

module load Trimmomatic

INPUT=/work/mars8180/instructor_data/metagenomics/raw-data
OUTPUT=/work/mars8180/instructor_data/metagenomics/analysis/03-trimmomatic
ADAPTER=/work/mars8180/instructor_data/metagenomics/database/trimmomatic/NexteraPE-PE.fa

mkdir ${OUTPUT}
java -jar $EBROOTTRIMMOMATIC/trimmomatic-0.39.jar PE -phred33 ${INPUT}/tricoma_059_AQ_202303_R1.fastq.gz ${INPUT}/tricoma_059_AQ_202303_R2.fastq.gz \
    ${OUTPUT}/tricoma_059_AQ_202303_R1_paired.fastq.gz ${OUTPUT}/tricoma_059_AQ_202303_R1_unpaired.fastq.gz \
    ${OUTPUT}/tricoma_059_AQ_202303_R2_paired.fastq.gz ${OUTPUT}/tricoma_059_AQ_202303_R2_unpaired.fastq.gz \
    SLIDINGWINDOW:4:20 MINLEN:100 ILLUMINACLIP:${ADAPTER}:2:40:15 -threads 12
```

With trimmotatic the position of the arguments is important. First, we specify the data type (`PE`) and quality encoding method (`phred33`). Then we specify the input data in the following order: R1 and R2. 

After, we specify the output files in the following order: 

1. R1 paired
2. R1 unpaired
3. R2 paired
4. R2 unpaired

In this script, we specify a sliding window of 4:20. This means that if the quality of any 4 consecutive bases falls below a quality score of 20, trim. We will also specify to only keep reads with a minimum length of 100bp. After we specify the adapters we want to remove from our data followed by 3 numbers that indicate how we should deal with adapter matching: 

* seedMismatches: specifies the maximum mismatch count which will still allow a full match to be performed
* palindromeClipThreshold: specifies how accurate the match between the two 'adapter ligated' reads must be for PE palindrome read alignment.
* simpleClipThreshold: specifies how accurate the match between any adapter etc. sequence must be against a read.

In this script we indicate 2 seedmismatches, 40 palindromeClipThreshold, and 15 simpleClipThreshold. 

### Assessing quality after trimming and adapter removal 

Now, we can rerun FASTQC and MultiQC to assess whether our trimming parameters improved our data quality. 

The files are called `04-fastqc-trimmed.sh` and `05-multiqc-trimmed.sh`

### Assembling contigs 
Now, we can assemble our quality-controlled data using Megahit.  


```
$ cat scripts/06-assembly-megahit.sh

#!/bin/sh
#SBATCH --job-name="megahit"
#SBATCH --partition=highmem
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=24
#SBATCH --mem-per-cpu=30G
#SBATCH --time=7-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 07-megahit.err-%N
#SBATCH -o 07-megahit.out-%N

module load MEGAHIT

INPUT=/work/mars8180/instructor_data/metagenomics/analysis/03-trimmomatic
OUTPUT=//work/mars8180/instructor_data/metagenomics/analysis/06-assembly

mkdir -p ${OUTPUT}
megahit -1 ${INPUT}/tricoma_059_AQ_202303_R1_paired.fastq.gz \
    -2 ${INPUT}/tricoma_059_AQ_202303_R2_paired.fastq.gz \
    -o ${OUTPUT}/tricoma_059_AQ_202303 -t 24 --presets meta-sensitive --min-contig-len 1000
```

Here, we will use a present called `meta-sensitive` with predefined set of kmers that work best for assembling low diversity samples. We will also specify a minimum contig length of 1,000 (anything less than 1,000bp is not useful for binning or functional annotation). 

### Read map our samples

After assembling our contigs, we can read map our quality-controlled data to our contigs to get abundance information for each contig. This will help us bin our samples into metagenome-assembled genomes. We will use two tools for this: BWA and SAMtools. 

```
$ cat scripts/07-read-map-reads.sh

#!/bin/bash

#SBATCH --job-name="read-map"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=32
#SBATCH --mem-per-cpu=2G
#SBATCH --time=7-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e read-map.err-%N
#SBATCH -o read-map.out-%N

# path variables and modules
module load BWA
module load SAMtools

CONTIGS=/work/mars8180/instructor_data/metagenomics/analysis/06-assembly/tricoma_059_AQ_202303/final.contigs.fa
READS=/work/mars8180/instructor_data/metagenomics/analysis/03-trimmomatic
OUTPUT=/work/mars8180/instructor_data/metagenomics/analysis/07-read-map

mkdir -p ${OUTPUT}

bwa index ${CONTIGS}
bwa mem -t 32 ${CONTIGS} ${READS}/tricoma_059_AQ_202303_R1_paired.fastq.gz ${READS}/tricoma_059_AQ_202303_R2_paired.fastq.gz | samtools sort -o ${OUTPUT}/tricoma_059_AQ_202303-alignment.bam --threads 32
samtools index -@ 32 ${OUTPUT}/tricoma_059_AQ_202303-alignment.bam
```

When we assemble our genome we lose quantitative information. We can map our reads back to the assembly to determine the relative abundance of each contig in the sample. This is important because we can use this information to bin our sequences into metagenome-assembled genomes (MAGs) - sequences that come from the same organisms should be present in roughly equal proportions.

We will use two tools - BWA and SAMTools - to read map our samples. BWA will map short-reads to the assembly. After we will sort the BAM file and index it for rapid random access.

BWA (Li et al. 2009, Fast and accurate short read alignment with Burrows–Wheeler transform - [https://academic.oup.com/bioinformatics/article/25/14/1754/225615?login=false]())

SamTools (Danecek et al. 2021, Twelve years of SAMtools and BCFtools - [https://academic.oup.com/gigascience/article/10/2/giab008/6137722?login=false]())

To read map our samples, we 1) index our assembly, 2) read map using BWA, 3) sort to SAM file and convert to BAM, and 4) index the sorted BAM file.

### Binning MAGs

```
$ cat scripts/08-bin-mags.sh

#!/bin/bash

#SBATCH --job-name="metabat"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=24
#SBATCH --mem-per-cpu=2G
#SBATCH --time=7-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e metabat.err-%N
#SBATCH -o metabat.out-%N

# path variables and modules
module load MetaBAT

CONTIGS=/work/mars8180/instructor_data/metagenomics/analysis/06-assembly/tricoma_059_AQ_202303/final.contigs.fa
MAP=/work/mars8180/instructor_data/metagenomics/analysis/07-read-map/tricoma_059_AQ_202303-alignment.bam
OUTPUT=/work/mars8180/instructor_data/metagenomics/analysis/08-metagenome-bins

mkdir ${OUTPUT}
jgi_summarize_bam_contig_depths --outputDepth ${OUTPUT}/tricoma_059_AQ_202303-depth.txt ${MAP}
metabat2 -i ${CONTIGS} -a ${OUTPUT}/tricoma_059_AQ_202303-depth.txt -o ${OUTPUT}/tricoma_059_AQ_202303 -t 24
```


We are able to use our abundance information to bin bacterial contigs into metagenome-assembled genomes. We are going to use a single tool to bin our datasets: MetaBat2. MetaBat2 uses tetranucleotide frequencies in conjunction with abundance information for genome reconstruction. 

MetaBat2 (Kang et al. 2019, MetaBAT 2: an adaptive binning algorithm for robust and efficient genome reconstruction from metagenome assemblies - [https://peerj.com/articles/7359/]())

