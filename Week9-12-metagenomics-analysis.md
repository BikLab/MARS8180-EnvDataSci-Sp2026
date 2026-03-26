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

## Metagenome Assemblies

#### Important Definitions:

* **Kmers:** A substring of a sequence of the length K 
	
	Take the sequence ATGCTGCT as an example. This sequence can be broken up into several substrings (or subsequences) of length 4 (tetramer or 4-mer).
	* ATGC
	* TGCT
	* GCTG
	* CTGC
	* TGCT

	This is useful because we can count how many unique and total kmers we find in a sequence and quickly compare them to other sequences. In the example above, there are 5 tetramers (4 unique tetramers). This is useful because alignment algorithms require a lot of resources - the computational time and memory required grow exponentially for each added sequence. Using the same example above, there are only 256 (4^4) unique tetramers. If we increase the size of the kmer, we have more unique kmer combination but fewer overlapping substrings. Even BLAST utilizes Kmers (default size 28; 4^28 unique combinations) to identify exact matches across the entire NCBI database (>3.7 billion sequences). 

* **De Bruijn Graphs:** Graph theory underpin many -omics assembly methods. De brujin graphs are old - they were first developed in 1946 by the Mathmatecian Nicolaas de Brujin. In short they are a directed graph-based method of visualizing and assembling sequence data. You take your kmers and connect them if they overlap. Afterwards, you can follow all your overlapping sequences to identify assembled contigs. **Most modern assemblers are based on De Bruijn Graphs**, although older tools can use other methods such as the Naïve approach, Greedy approach, or Overlap Layout Consensus.

<img width="1294" alt="Screenshot 2025-03-20 at 8 58 40 AM" src="https://github.com/user-attachments/assets/aaa54f12-a53c-4aff-8bc6-75e0bf378c52" />

What a De Bruijn Graph actually looks like (visualization of the underlying math - Image from Wikipedia https://en.wikipedia.org/wiki/De_Bruijn_graph) 

How this actually works in practice: https://www.youtube.com/watch?v=OY9Q_rUCGDw 

## Assembling Short-Read Sequences

Two most popular metagenome assemblers for Illumina short read sequences: 

* **MEGAHIT** (Li et al. 2015, MEGAHIT: an ultra-fast single-node solution for large and complex metagenomics assembly via succinct de Bruijn graph https://academic.oup.com/bioinformatics/article/31/10/1674/177884) 
* **metaSPAdes** (Nurk et al. 2017, metaSPAdes: a new versatile metagenomic assembler - https://genome.cshlp.org/content/27/5/824)

There are several tools you can use to assembly short-read metagenomics, but we are going to implement MegaHit for three main reasons: 1) it is incredibly fast, 2) it requires less computational resources, and 3) assembles metagenomic datasets fairly well. 

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

### Binning Metagenome-assembled Genomes (MAGs)

We are now able to use our abundance information to bin bacterial contigs into metagenome-assembled genomes. We are going to use three tools 1) MetaBat2, 2) comebin, and 3) dastool. Both MetaBat2 and Comebin use tetranucleotide frequencies in conjunction with abundance information for genome reconstruction. However, Comebin uses a contrastive multi-view representation learning to determine the best MAGs and incorporates single-copy gene information and contig length. Finally, Dastool compares the MAGs produced by any binning algorithm and determine the most complete MAGs (with least amount of contamination). 

* **MetaBat2** (Kang et al. 2019, MetaBAT 2: an adaptive binning algorithm for robust and efficient genome reconstruction from metagenome assemblies - [https://peerj.com/articles/7359/](https://peerj.com/articles/7359/))
* **Comebin** (Wang et al. 2024, Effective binning of metagenomic contigs using contrastive multi-view representation learning - [https://www.nature.com/articles/s41467-023-44290-z](https://www.nature.com/articles/s41467-023-44290-z))
* **Dastool** (Sieber et al. 2018, Recovery of genomes from metagenomes via a dereplication, aggregation and scoring strategy - [https://www.nature.com/articles/s41564-018-0171-1](https://www.nature.com/articles/s41564-018-0171-1))

## MetaBat2

![peerj-03-1165-g001](https://github.com/user-attachments/assets/d3604d96-0238-42dd-a283-4590f7fae801)

The figure above is from the MetaBat1 software; howwever, MetaBat2 works similarly, except that it is iterative and chooses the best parameters according to your dataset. It does not require to make any decisions about which parameters would be best suited for your dataset. Additionally, MetaBat2 constructs a graph using the tetranucleotide frequency and read abundance to cluster contigs that appear to be similar in structure. 

For MetaBat2, we are first going to summarize our BAM file and then reconstruct the genomes.

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

## DAStool 

![image](https://github.com/user-attachments/assets/d6cd0bbb-ed5c-42df-8625-76b8963998d0)

* Step 1: The DasTool inputs are the bins output by each binning algorith. 
* Step 2: Single-copy genes are predicted and the bins are scored according to completeness and contamination.
* Step 3: Bins that were predicted by multiple binning algorithms are dereplicated.
* Step 4: There is an iterative selection of "best" bins and the partial candidate bins are selected. The output are a non-redundant set of the high-scoring bins from predicted by different binning algorithms.

To run DASTool we first need to make a list of contigs that belong to each bin. Afterwards we can compare the assemblies and choose the best one. We can set the score-threshold to 0 to force it to bin incomplete MAGs (low completion according to single-copy genes). 

```
#!/bin/bash

#SBATCH --job-name="dastool"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=12
#SBATCH --mem-per-cpu=2G
#SBATCH --time=7-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e dastool.err-%N
#SBATCH -o dastool.out-%N

# path variables and modules
module load Miniforge3
source activate /home/ad14556/dastool

CONTIGS=/work/mars8180/instructor_data/metagenomics/analysis/06-assembly/tricoma_059_AQ_202303/final.contigs.fa
MAP=/work/mars8180/instructor_data/metagenomics/analysis/07-read-map
METABAT=/work/mars8180/instructor_data/metagenomics/analysis/08-metagenome-bins
OUTPUT=/work/mars8180/instructor_data/metagenomics/analysis/09-dastool

mkdir -p ${OUTPUT}
Fasta_to_Contig2Bin.sh -i ${METABAT} -e fa > ${OUTPUT}/tricoma_059_AQ_202303-metabat-scaffolds2bin.tsv
DAS_Tool -i ${OUTPUT}/tricoma_059_AQ_202303-metabat-scaffolds2bin.tsv \
  -l metabat \
  -c ${CONTIGS} \
  -o ${OUTPUT}/tricoma_059_AQ_202303 \
  --write_bins \
  --write_bin_evals \
  -t 12 \
  --score_threshold=0.5
```

## Assessing Bin quality using CheckM2

In 2017, two standards were developed by the Genomic Standards Consortium (GSC) for reporting bacterial and archaeal genome sequences. MAGs are classified on different classification levels depending on the completion and contamination determined using single-copy genes.

* **Finished Single Contig MAG**: >90% complete with less than 5% contamination. Genomes in this category should be a single contiguous sequence and also encode the 23S, 16S, and 5S rRNA genes, and tRNAs for at least 18 of the 20 possible amino acids
* **High-quality MAG**: >90% complete with less than 5% contamination. Genomes in this category should also encode the 23S, 16S, and 5S rRNA genes, and tRNAs for at least 18 of the 20 possible amino acids.
* **Medium-quality MAG**: ≥50% complete and less than 10% contamination.
* **Low-quality MAG**: <50% complete with <10% contamination.

It should be noted that there is no minumum assembly size since genomes smaller than 200 kb have been reported. Additionally assembly statistics are usually reported when depositing sequences (N50, L50, largest contig, number of contigs, assembly size, percentage of reads that map back to the assembly, and number of predicted genes per genome).

**Bowers et al. (2017) Minimum information about a single amplified genome (MISAG) and a metagenome-assembled genome (MIMAG) of bacteria and archaea [https://www.nature.com/articles/nbt.3893]()**

We will use the software program CheckM2 to determine the quality of the MAGs:

* Chklovski A, Parks DH, Woodcroft BJ, Tyson GW (2023) CheckM2: a rapid, scalable and accurate tool for assessing microbial genome quality using machine learning. Nature Methods, 20: 1203-1212 - [https://www.nature.com/articles/s41592-023-01940-w]()

It uses two machine learning models to determine the quality depending on wether the genome is novel or if it is closely related to a genome in the training set. These two machine learning methods are:

1. **Gradient boosted decision trees**: Ke, G. et al. Lightgbm: A highly efficient gradient boosting decision tree. Adv. Neural Inf. Process. Syst. 30, 3146–3154 (2017)
2. **Artifical Neural Networks**: Abadi, M. et al. Tensorflow: a system for large-scale machine learning. In Proc. 12th USENIX Symposium on Operating Systems Design and Implementation (OSDI 16) 265–283 (2016)

We can run CheckM2 with the following script:

```
cat scripts/10-checkm2.sh

#!/bin/bash

#SBATCH --job-name="checkm"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=24
#SBATCH --mem-per-cpu=2G
#SBATCH --time=7-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e checkm.err-%N
#SBATCH -o checkm.out-%N

# path variables and modules
module load CheckM2

BINS=/work/mars8180/instructor_data/metagenomics/analysis/09-dastool/tricoma_059_AQ_202303_DASTool_bins
OUTPUT=/work/mars8180/instructor_data/metagenomics/analysis/10-checkm2
DATABASE=/work/mars8180/instructor_data/metagenomics/database/checkm2/uniref100.KO.1.dmnd

mkdir ${OUTPUT}
checkm2 predict --force -x .fa --threads 24 --database_path ${DATABASE} --input ${BINS} --output-directory ${OUTPUT}
```

## Classifying MAGs with GTDB-tk
To identify taxonomically classify the the bacterial bins, we can use the GTDB-tk. This tool will identify single-copy genes and place them on a phylogenetic tree. It will out put a text file with the taxonomic id of each bin

```
cat scripts/11-gtdbtk.sh

#!/bin/bash

#SBATCH --job-name="gtdb-tk"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=30
#SBATCH --mem-per-cpu=2G
#SBATCH --time=7-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e gtdb-tk.err-%N
#SBATCH -o gtdb-tk.out-%N

# path variables and modules
module load GTDB-Tk/2.4.1-foss-2023a

BINS=/work/mars8180/instructor_data/metagenomics/analysis/09-dastool/tricoma_059_AQ_202303_DASTool_bins
OUTPUT=/work/mars8180/instructor_data/metagenomics/analysis/11-gtdbtk

mkdir ${OUTPUT}
gtdbtk classify_wf --genome_dir ${BINS} --out_dir ${OUTPUT} --skip_ani_screen -x fa --cpus 24 --pplacer_cpus 24
```
