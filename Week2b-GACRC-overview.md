## GACRC File Management and Organization

Three main file systems that you will work with on the GACRC:
`/project/labname/` - your long term storage, accessible only on the Xfer node (storage limit differs based on PI lab allocation, usually at least 1Tb)
`/work/labname/` - your "permanent" storage where you can store raw data and scripts, accessible on Sapelo 2 and the Xfer node (200Gb storage limit)
`/scratch/myID/` - your temporary storage where your scripts should put all your output files, because scratch has faster read/write speeds (no storage limit)

The raw data and scripts should be in /work since thats "permanent" storage. The outputs should first be output in /scratch (faster read/write speeds) and then the files can be transferred to work. Work has 200Gb storage and scratch doesn't have a user limit
