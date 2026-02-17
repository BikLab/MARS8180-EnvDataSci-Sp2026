# Before we get started

### Demultiplex Data 
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

Lets use the following command to get the first 8 lines of one of our ddt-samples. 

```
zcat ddt-raw-fastq/DDT.10.1_S1258_L001_R1_001.fastq.gz | head -n 8
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

# 16S Metabarcoding Analsysis 
Before we get started, let's copy our data over to a new directory `mars8180-class-data` directory.'

Login to Sapelo2 using PuTTy and create your directory using the following command: 

```
$ mkdir mars8180-class-data
$ cd mars8180-class-data
```

now use `scp` to copy the data files, metadata, and scripts from the teaching cluster to Sapelo2. 

`$ scp -r userid@txfer.gacrc.uga.edu:/work/mars8180/instructor_data/metabarcoding-16S/scripts`

`$ scp -r userid@txfer.gacrc.uga.edu:/work/mars8180/instructor_data/metabarcoding-16S/metadata`

`$ scp -r userid@txfer.gacrc.uga.edu:/work/mars8180/instructor_data/metabarcoding-16S/raw-data`

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

