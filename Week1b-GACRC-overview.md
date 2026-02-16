## GACRC File Management and Organization

### Initial setup

Please watch this Linux basics playlist before training if you are not already familiar with Linux: https://kaltura.uga.edu/playlist/dedicated/176125031/1_uwkiealj/

Windows Users:
* If you are connecting from a Windows computer, please download and install PuTTY. You can find detailed instructions on how to download and install PuTTY on your Windows computer at https://wiki.gacrc.uga.edu/wiki/How_to_Install_and_Configure_PuTTY

Mac Users
* To login to any GACRC nodes, you will be using the built-in terminal application available on all Mac computers (found under Applications-->Utilities-->Terminal)

### File Systems

Three main file systems that you will work with on the GACRC:

* `/project/labname/` - your long term storage, accessible only on the Xfer node (storage limit differs based on PI lab allocation, usually at least 1Tb)
* `/work/labname/` - your "permanent" storage where you can store raw data and scripts, accessible on Sapelo 2 and the Xfer node (200Gb storage limit)
* `/scratch/myID/` - your temporary storage where your scripts should put all your output files, because scratch has faster read/write speeds (no storage limit)

For example, our Bik Lab project directory is `/project/hmblab/`, work is `/work/hmblab/` and Holly's personal scratch is `/scratch/hmb33427/`

### Key commands for submitting jobs

The GACRC using the Slurm queuing system, and some key commands you will use frequently are:

* `sbatch script.sh` - main command for submitting jobs (as shell script text files)
* `sq --me` - command to check what jobs you have running and have submitted to the queue
* `scancel <JobID>` - command to cancel a Job; replace <JobID> with the numberical job number

If you are seeing jobs with CPU efficiencies <70%, that is an indicator that you're requesting exceessive resources - may need to tweak your script header

### Teaching cluster setup

Please ensure that you can login to the Teaching Cluster before training:

* Instructions for Mac (please use the hostname teach.gacrc.uga.edu): https://kaltura.uga.edu/playlist/dedicated/176125031/1_a6e4voao/1_5vozd2lt
* Instructions for Windows computer (please use the hostname teach.gacrc.uga.edu): https://kaltura.uga.edu/media/t/1_s50lszs5/176125031
