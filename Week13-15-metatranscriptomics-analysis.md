<img width="1182" height="665" alt="Screenshot 2026-03-02 at 10 09 14 PM" src="https://github.com/user-attachments/assets/fe700b94-e796-4617-9719-57db94b6ea2d" />

# Metatranscriptomic data
This week, we will be spending time analyzing a small subset of samples (from 2 stations) that were collected as part of the Tara Oceans Project. There are a total of 12 samples (6 from station TARA_135 and 6 from station TARA_137):

3 surface water (5m) samples near Honolulu
3 water samples at the deep-chlorphyll maximum (30-40m) near Honolulu
3 surface water (5m) samples near San Diego
3 water samples at the deep-chlorphyll maximum (30-40m) near San Diego

Example of the metadata file can be seen here: 


| sampleID | sampleENA | event | station | location | long | lat | depth | collection | notes |
|----------|-----------|-------|---------|----------|------|-----|-------|------------|-------|
|ERR1712149| ERS493517	| TARA-20110928Z-SF | TARA_135 | Honolulu | 21.283 | -157.871 | 5m | SEQ-(100L-or-15min)-W>0.8 | surface water | 
| ERR1711927 | ERS493555	| TARA-20110928Z-CH | TARA_135 | Honolulu |21.283 | -157.871 | 30m | SEQ-(100L-or-15min)-W0.8-5	| deep chlorophyll maximum layer |
| ERR1712163	| ERS493652 | TARA-20111124Z-SF | TARA_137 | San Diego | 32.621 | -117.246 | 5m | SEQ-(500mL-or-15min)-N180-2000	| surface water |
| ERR1719158	| ERS493677 | TARA-20111124Z-CH | TARA_137 | San Diego | 32.621 | -117.246 | 40m | SEQ-(100L-or-15min)-W0.8-5 | deep chlorophyll maximum layer |

The steps of analyzing a metatranscriptomic dataset is nearly identical to metagenomic data. We have to

Quality control our raw data
Assembly our short-read sequences into contigs
Assess quality of our assembly
Identify protein coding sequences
Assign taxonomy classifcation to each contig
Annotate the contigs

**We will largely be following recommendations set forth by Krinos et al (2023) [https://bmcbioinformatics.biomedcentral.com/articles/10.1186/s12859-022-05121-y]()**

## Before we get started

Before, we get started let's go ahead and downlaod the necessary metatranscriptomics data files including the raw-data and scripts. 

All scripts, data, and metadata files for this section are hosted on the teaching cluster in the following file path: `/work/mars8180/instructor_data/metatranscriptomics/` (you can always recopy this directory and start over if you make any mistakes on Sapelo2)

First, we will need to set up our directory structure on Sapelo2 in your `/home/myID/mars8180-course` directory, using the following commands:

1. Change directories `cd /home/myID/mars8180-course`
2. Make a directory for the metagenomics sections `mkdir metatranscriptomics`
3. Change directories `cd metatranscriptomics `
3. Now we will make a new folder for our outputs using the command `mkdir analysis`
4. Next, we will copy three directories (scripts and raw-data) over from the teaching cluster using the `scp` file transfer command, as following:

 `scp -r myid@teach.gacrc.uga.edu:/work/mars8180/instructor_data/metatranscriptomics/raw-data .`
 `scp -r myid@teach.gacrc.uga.edu:/work/mars8180/instructor_data/metatranscriptomics/scripts .`
 `scp -r myid@teach.gacrc.uga.edu:/work/mars8180/instructor_data/metatranscriptomics/metadata .`

The `scp` command always needs an origin where the files are locted (the teaching cluster, in this instance) and a destination location (here, we specify ` . ` to indicate that the files should be copied to the current folder). The `-r` flag specifies that that it should recursively roll down the list of files and directories and copy everything contained within in the origin folder.

After completing the above steps, your filepath for your raw data should be located on Sapelo2 at the following file path `/home/myID/mars8180-course/metatranscriptomics/raw-data`. 

## Visualizing data quality
Similar to the metagenomics workflow, we can run two scripts `01-fastqc-raw-data.sh` and `02-multiqc-raw-data.sh` to visualize data quality of the metagenomics dataset. Before you run the scripts, change the file paths so that they match the location in your directory. 

```
cat 01-fastqc.sh

#!/bin/sh
#SBATCH --job-name="fastqc"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem-per-cpu=1G
#SBATCH --time=1-00:00:00
#SBATCH --mail-user=userid@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 01-fastqc.err-%N
#SBATCH -o 01-fastqc.out-%N

module load FastQC/0.12.1-Java-11

INPUT=/work/mars8180/instructor_data/metatranscriptomics/raw-data
OUTPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/01-fastq

mkdir -p ${OUTPUT}
fastqc ${INPUT}/* -o ${OUTPUT} -t 8
```

We will use a second script 02-multiqc-raw-data.sh to collate all the reports onto a single file.


```
cat 02-multiqc.sh

#!/bin/sh
#SBATCH --job-name="multiqc"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1G
#SBATCH --time=1-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 01-multiqc.err-%N
#SBATCH -o 01-multiqc.out-%N

module load MultiQC/1.28-foss-2024a

INPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/01-fastq
OUTPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/02-multiqc

multiqc --outdir ${OUTPUT} ${INPUT}
```

The input is the folder where all the fastqc reports are generated. The output is a folder that contains the html file with the multiqc report summary. After you submit this job, you can download the output file to your personal computer using PuTTy or the CommandLine. We will go over the interactive quality plots in class.

## Quality and Adapter trimming

Now, we can use a tool, Trimmomatic, to quality trim our reads and remove adapters sequences. However, this time, we are analyzing multiple samples, so we will use a for loop to iterate the same command over all our data files.

```
$ cat scripts/03-trimmomatic.sh

#!/bin/sh
#SBATCH --job-name="trimmomatic"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=24
#SBATCH --mem-per-cpu=5G
#SBATCH --time=1-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 03-trimmomatic.err-%N
#SBATCH -o 03-trimmomatic.out-%N

module load Trimmomatic/0.39-Java-17

INPUT=/work/mars8180/instructor_data/metatranscriptomics/raw-data
OUTPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/03-trimmomatic
ADAPTER=/work/mars8180/instructor_data/metagenomics/database/trimmomatic/NexteraPE-PE.fa

mkdir -p ${OUTPUT}
for file in ${INPUT}/*_1.fastq.gz; do
  base=$(basename ${file} _1.fastq.gz)
  java -jar $EBROOTTRIMMOMATIC/trimmomatic-0.39.jar PE -phred33 ${INPUT}/${base}_1.fastq.gz ${INPUT}/${base}_2.fastq.gz \
      ${OUTPUT}/${base}_R1_paired.fastq.gz ${OUTPUT}/${base}_R1_unpaired.fastq.gz \
      ${OUTPUT}/${base}_R2_paired.fastq.gz ${OUTPUT}/${base}_R2_unpaired.fastq.gz \
      SLIDINGWINDOW:4:20 MINLEN:25 ILLUMINACLIP:${ADAPTER}:2:40:15 -threads 24
done
```

First, we specify for each sample in the directory `${INPUT}` do the following: 

1. **Use the `basename` command to create a variable with the sample name.** This remove the path information and any patterns. In this case we want to remove `_1.fastq.gz` and only keep the sample names.
2. **Run `Trimmomatic`**. We will use the input path and the new variable `${base}` to specify the sample names. 
3. Done 

This will iterate the same command for each sample. 


## Visualizing data quality

Now run the scripts `04-fastqc.sh` and `05-multiqc.sh` to assess the quality of the quality trimmed data. 

## Assembling data

Simliar to the metagenomics dataset, we are going to use MegaHit to assembled our transcriptomic data. However, this time we have multiple Fastq files, so we will have to write this using a loop.

```
$ cat 06-megahit.sh

#!/bin/sh
#SBATCH --job-name="megahit"
#SBATCH --partition=highmem
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=24
#SBATCH --mem-per-cpu=15G
#SBATCH --time=7-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 06-megahit.err-%N
#SBATCH -o 06-megahit.out-%N

module load MEGAHIT/1.2.9-GCCcore-13.3.0

INPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/03-trimmomatic
OUTPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/06-assembly

mkdir -p ${OUTPUT}

for file in ${INPUT}/*_R1_paired.fastq.gz; do
  base=$(basename ${file} _R1_paired.fastq.gz)
  megahit --k-min 21 --k-max 111 -m 0.9 -t 24 -1 ${INPUT}/${base}_R1_paired.fastq.gz -2 ${INPUT}/${base}_R2_paired.fastq.gz \
    -f -o ${OUTPUT}/${base}
done
```

We will following recommendations by Krinnos et al and maintain a kmer size range from 21-111. 

1. **Use the `basename` command to create a variable with the sample name.** This remove the path information and any patterns. In this case we want to remove `_1.fastq.gz` and only keep the sample names.
2. **Run `Megahit`**. We will use the input path and the new variable `${base}` to specify the sample names. 
3. Done 

## Assess Assembly Quality

We are going to use a new tool, QUAST, to assess how well the data assembled. QUAST is agnostic to datatype, so you can also use this for metagenomic datasets. 

```
$ cat 07-quast.sh

#!/bin/sh
#SBATCH --job-name="quast"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --mem-per-cpu=5G
#SBATCH --time=7-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 07-quast.err-%N
#SBATCH -o 07-quast.out-%N

module load QUAST/5.2.0-gfbf-2023b

ASSEMBLY=/work/mars8180/instructor_data/metatranscriptomics/analysis/06-assembly
OUTPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/07-quast

mkdir -p ${OUTPUT}

for folder in ${ASSEMBLY}/*; do
  base=$(basename ${folder})
  quast ${ASSEMBLY}/${base}/final.contigs.fa -o ${OUTPUT}/${base} --threads 16
done
```

Quant is an easy-to-use software that estimates assembly statistics, such as:

* Number of Contigs
* Length of the Assembly
* N50
* N90
* L50
* L90
* GC%

## Merge transcriptomes
First, we will merge all of our assemblies together.

```
$ cd /work/mars8180/instructor_data/metatranscriptomics/scripts
$ cat 08-merge-assemblies.sh

#!/bin/sh
#SBATCH --job-name="merge"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=50G
#SBATCH --time=7-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 09-merge.err-%N
#SBATCH -o 09-merge.out-%N

module load MMseqs2/18-8cc5c-gompi-2025a

INPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/06-assembly
OUTPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/08-merged-assembly

mkdir -p ${OUTPUT}
cat ${INPUT}/*/*.fa > ${OUTPUT}/merged-assembly.fa

mmseqs createdb ${OUTPUT}/merged-assembly.fa ${OUTPUT}/merged-assembly
mmseqs linclust ${OUTPUT}/merged-assembly ${OUTPUT}/merged-assembly-2 tmp --min-seq-id 0.98 --cov-mode 1 --split-memory-limit 120G --remove-tmp-files
mmseqs createsubdb ${OUTPUT}/merged-assembly-2 ${OUTPUT}/merged-assembly ${OUTPUT}/merged-assembly-3
mmseqs convert2fasta ${OUTPUT}/merged-assembly-3 ${OUTPUT}/merged-assembly-98.fa
```

And then, we will simplify the contig names,which is a requirement for some bioinformatics tools.

```
$ cat 09-rename-contigs.sh

#!/bin/sh
#SBATCH --job-name="rename"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=20G
#SBATCH --time=7-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 09-rename.err-%N
#SBATCH -o 09-rename.out-%N

module load seqtk/1.4-GCC-13.3.0

ASSEMBLY=/work/mars8180/instructor_data/metatranscriptomics/analysis/08-merged-assembly

mkdir -p ${ASSEMBLY}

sed 's/ /_/g' ${ASSEMBLY}/merged-assembly-98.fa > ${ASSEMBLY}/merged-assembly-98-sans-space.fa #remove white spaces in contig names
seqtk rename ${ASSEMBLY}/merged-assembly-98-sans-space.fa contig_ > ${ASSEMBLY}/merged-assembly-98-simplify-namese.fa # simplify the contig names

```

## Filtering short transcripts
Now, we can filter our dataset to remove short contigs that will be 1) noninformative for functional annotation, or 2) represent short regulatory RNA. Here, we will set a minimum length of 300bp, but this will vary according to your own project.

```
$ cat 10-seqkit-filter.sh

#!/bin/sh
#SBATCH --job-name="filter"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=12
#SBATCH --mem-per-cpu=20G
#SBATCH --time=1-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 10-filter.err-%N
#SBATCH -o 10-filter.out-%N

module load SeqKit/2.8.2

INPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/08-merged-assembly/merged-assembly-98-simplify-names.fa
OUTPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/10-merged-assembly-98-simplify-names-min-length.fa

seqkit seq -m 300 -g -w 0 ${INPUT} > ${OUTPUT}l
```

## Qualtifying transcripts using SALMON
Here, we will use a tool SALMON to quantiy the abundance of our transcripts. 

There are two main steps:

1. index your assembly
2. quantify

These can be run consequetively:

```
salmon index -t final.contigs.fa -i sample-salmon-index -k 31 --threads 12
salmon quant -i sample-salmon-index -l A -1 sample_R1_paired.fastq.gz -2 sample_R2_paired.fastq.gz --validateMappings -o output-sample-dir
```

We will index our data and quantify are transcripts for each sample using a single scripts: 

```
$ cat 11-salmon-quantity.sh

#!/bin/sh
#SBATCH --job-name="salmon"
#SBATCH --partition=gpu
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=12
#SBATCH --mem-per-cpu=5G
#SBATCH --time=7-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 11-salmon-quant.err-%N
#SBATCH -o 11-salmon-quant.out-%N

module load Salmon/1.10.3-GCC-12.3.0

ASSEMBLY=/work/mars8180/instructor_data/metatranscriptomics/analysis/10-merged-assembly-98-simplify-names-min-length.fa
READS=/work/mars8180/instructor_data/metatranscriptomics/analysis/03-trimmomatic
OUTPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/11-salmon-quant

salmon index -t ${ASSEMBLY} -i ${OUTPUT}/merged-salmon-index -k 31 --threads 12

for file in ${READS}/*_R1_paired.fastq.gz; do
  base=$(basename ${file} _R1_paired.fastq.gz)
  salmon quant -i ${OUTPUT}/merged-salmon-index -l A -1 ${READS}/${base}_R1_paired.fastq.gz -2 ${READS}/${base}_R2_paired.fastq.gz --validateMappings -o ${OUTPUT}/${base} --threads 12
done
```

After, we can use a second scripts to merge our results for all our samples. 

```
$ cat 12-salmon-merge.sh

#!/bin/sh
#SBATCH --job-name="salmon"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=20
#SBATCH --mem-per-cpu=5G
#SBATCH --time=7-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 12-salmon-merge.err-%N
#SBATCH -o 12-salmon-merge.out-%N

module load Salmon/1.10.3-GCC-12.3.0

INPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/11-salmon-quant
OUTPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/12-salmon-all-sample-quant-counts.txt

salmon quantmerge --quants ${INPUT}/ERR* -o ${OUTPUT} -c numreads
```

## Identify protein coding sequences

To identify protein coding regions within transcripts we will use the tool TransDecoder. TransDecoder identifies likely coding sequences based on the following criteria:

1. a minimum length open reading frame (ORF) is found in a transcript sequence
2. a log-likelihood score similar to what is computed by the GeneID software is > 0.
3. the above coding score is greatest when the ORF is scored in the 1st reading frame as compared to scores in the other 2 forward reading frames.
4. if a candidate ORF is found fully encapsulated by the coordinates of another candidate ORF, the longer one is reported. However, a single transcript can report multiple ORFs (allowing for operons, chimeras, etc).

TransDecoder is run in two steps. First, sequences with long open reading frames are extracted from the assembled sequences. By default, only sequences that are 100 amino acids long are kept. Second, you predict the likely coding region.

In practice, it looks like the following:

```
TransDecoder.LongOrfs -t final.contigs.fa -m 100 --output_dir output/dir/sample-basename
TransDecoder.Predict -t final.contigs.fa --output_dir output/dir/sample-basename
```

You will have four final outputs:

* **transcripts.fasta.transdecoder.pep**: peptide sequences for the final candidate ORFs; all shorter candidates within longer ORFs were removed. This is the most important file you'll need.
* **transcripts.fasta.transdecoder.cds**: nucleotide sequences for coding regions of the final candidate ORFs
* **transcripts.fasta.transdecoder.gff3**: positions within the target transcripts of the final selected ORFs
* **transcripts.fasta.transdecoder.bed**: bed-formatted file describing ORF positions, best for viewing using GenomeView or IGV.

Now, let's run this script on our own data: 

```
$ cat 13-transdecoder.sh

#!/bin/sh
#SBATCH --job-name="transdecoder"
#SBATCH --partition=highmem
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=150G
#SBATCH --time=7-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 13-transdecoder.err-%N
#SBATCH -o 13-transdecoder.out-%N

module load TransDecoder

ASSEMBLY=/work/mars8180/instructor_data/metatranscriptomics/analysis/10-merged-assembly-98-simplify-names-min-length.fa
OUTPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/13-transdecoder

mkdir -p ${OUTPUT}

TransDecoder.LongOrfs -t ${ASSEMBLY} -m 100 --output_dir ${OUTPUT}
TransDecoder.Predict -t ${ASSEMBLY} --output_dir ${OUTPUT} --no_refine_starts
```

## Assign taxonomy to each contig
We will be using Eukulele and the PhyloDB database [https://github.com/allenlab/PhyloDB]() to assign taxonomy to our contig.

I have already installed the phyloDB database in the instructory directory `/work/mars8180/instructor_data/metatranscriptomics/databases/phylodb` using the following command:

```
EUKulele download --database phylodb
```

Now we are able to run Eukulele on our metatranscriptomes

```
$ cat 14-eukulele.sh

#SBATCH --job-name="eukulele"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=12
#SBATCH --mem-per-cpu=4G
#SBATCH --time=7-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 14-eukulele.err-%N
#SBATCH -o 14-eukulele.out-%N

module load EUKulele/2.0.9_2-foss-2022a
module load DIAMOND/2.1.23-GCC-13.3.0

INPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/13-transdecoder
OUTPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/14-eukulele
DATABASE=/work/mars8180/instructor_data/metatranscriptomics/databases/

mkdir -p ${OUTPUT}

EUKulele --mets_or_mags mets -s ${INPUT} --p_ext ".pep" -d phylodb --reference_dir ${DATABASE} -o ${OUTPUT} --CPUs 12
```

## Annotating contigs
To annotate our peptide file we are going to use a program called eggnog-mapper.

> EggNOG-mapper is a tool for fast functional annotation of novel sequences. It uses precomputed orthologous groups (OGs) and phylogenies from the eggNOG database (http://eggnogdb.embl.de/) to transfer functional information from fine-grained orthologs only.
> 
> Common uses of eggNOG-mapper include the annotation of novel genomes, transcriptomes or even metagenomic gene catalogs.
> 
> The use of orthology predictions for functional annotation permits a higher precision than traditional homology searches (i.e. BLAST searches), as it avoids transferring annotations from close paralogs (duplicate genes with a higher chance of being involved in functional divergence).

First, we need to download the eggnog database. I have already done this for you. The database is located in `/work/mars8180/instructor_data/metatranscriptomics/databases/eggnog`

Now we can run eggnog-mapper on our own data:

```
$ cat 15-eggnog-mapper.sh

#!/bin/sh
#SBATCH --job-name="eggnog"
#SBATCH --partition=batch
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=24
#SBATCH --mem-per-cpu=4G
#SBATCH --time=7-00:00:00
#SBATCH --mail-user=ad14556@uga.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH -e 15-eggnog.err-%N
#SBATCH -o 15-eggnog.out-%N

module load eggnog-mapper/2.1.12-foss-2023a

INPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/13-transdecoder/10-merged-assembly-98-simplify-names-min-length.fa.transdecoder.pep
OUTPUT=/work/mars8180/instructor_data/metatranscriptomics/analysis/15-eggnog-redo
DATABASE=/work/mars8180/instructor_data/metatranscriptomics/databases/eggnog

mkdir -p ${OUTPUT}
#mkdir -p ${DATABASE}

emapper.py --override -i ${INPUT} --itype proteins -m diamond --data_dir ${DATABASE} -o tara --output_dir ${OUTPUT} --cpu 24
```


