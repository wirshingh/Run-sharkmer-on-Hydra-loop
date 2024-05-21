# How to run sharkmer on Hydra using a loop

## Step 1
All reads files should be unzipped. The commnand gunzip can be used. The unzipped files will
 be large. In Hydra, navigate to the directory where the read files are located and run this job. 
 The path to the raw reads in the job can be changed if running from a different directory.
 
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
## Step 2
 ### Preparing your files
 
 1. In each read file include a unique sample ID. 
 2. Create a text file called "sharkmer_samples.txt" with each unique sample name from step 1 in a
 column (i.e., one sample name per line). 
 3. The reads files should end with "_R1_PE_trimmed.fastq" and "_R2_PE_trimmed.fastq" after 
 the unique sample name for the forward and reverse reads (R1 and R2).
 The job file below can also be modified to match your reads file names.
