<img width="1182" height="665" alt="Screenshot 2026-03-02 at 10 09 14 PM" src="https://github.com/user-attachments/assets/fe700b94-e796-4617-9719-57db94b6ea2d" />

https://private-user-images.githubusercontent.com/27451655/431037610-4042a01f-a783-4b37-8c98-97de36bbc751.jpg?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzUxNTIyMDcsIm5iZiI6MTc3NTE1MTkwNywicGF0aCI6Ii8yNzQ1MTY1NS80MzEwMzc2MTAtNDA0MmEwMWYtYTc4My00YjM3LThjOTgtOTdkZTM2YmJjNzUxLmpwZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjA0MDIlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwNDAyVDE3NDUwN1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWZlMWZlNGI5MDQwN2Q4NWYzZjExYWExNjMzMWJmNmM0ZDI0ZGMwNjdlOTQ2NzY1MTBhMjdmZGU3ODM0Y2IzNzAmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.mGHx8J7Mkb-kg32rl02Klrjc0IMHpfn-0XA-00aKNlw

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
