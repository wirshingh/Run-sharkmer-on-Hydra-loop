# How to run sharkmer on Hydra using a loop

## Step 1
All reads files should be unzipped. The commnand gunzip can be used. The unzipped files will
 be large. In Hydra, navigate to the directory where the read files are located and run this job. 
 The path to the raw reads can be changed if running from a different directory.
 
 ```
 # /bin/sh
# ----------------Parameters---------------------- #
#$ -S /bin/sh
#$ -pe mthread 2
#$ -q sThC.q
#$ -l mres=16G,h_data=8G,h_vmem=8G
#$ -cwd
#$ -j y
#$ -N gunzip
#$ -o gunzip.log
#
# ----------------Modules------------------------- #
#
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
#
gunzip *gz
#
echo = `date` job $JOB_NAME done
```
