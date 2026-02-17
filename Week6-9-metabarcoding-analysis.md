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

After you submit this job, you can download the output file to your personal computer using PuTTy or the CommandLine. 

