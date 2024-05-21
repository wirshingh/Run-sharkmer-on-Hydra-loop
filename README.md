# How to run sharkmer on Hydra using a loop

## Step 1
### Unzip the read files
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
 
 1. For each read file include a unique sample ID. 
 2. Create a text file called "sharkmer_samples.txt" with each unique sample ID from step 1 in a
 column, one sample ID per line.

```
Example_Sample_ID_1
Example_Sample_ID_2
Example_Sample_ID_3
```

 4. The reads files should end with "_R1_PE_trimmed.fastq" and "_R2_PE_trimmed.fastq" after 
 the unique sample name for the forward and reverse reads (R1 and R2).
 The job file below can also be modified to match your reads file names.

## Step 3
### Run the sharkmer job file in Hydra
Modify the commands in the job file below and run it in same directory as the read files and the sharkmer_samples.txt file. NOTE - THE JOB FILE WILL NOT RUN IF THE COMMANDS ARE NOT MODIFIED FOR YOUR PROJECT.

### How to modify the sharkmer commands in the job below
-- max-reads, this is usually set to 1000000 as recommended by the manual. More can be added if desired.


-s, this commnd has the variable ${SAMPLE} which will be each sample ID name listed in the "sharkmer_samples.txt" file. This command can be left alone.

-o, output directories will be created for each sample ID and end with "sharkmer_output". This can be left alone or modified if desired.

--pcr, for this command enter the forward and reverse primers sequences, the expected length of the PCR sequence (overestimate the length), and the name of the primer set

#### Note
Several primer pairs can be run at the same time. Simple add another --pcr command to the job file with the appropriate primer information

```
# /bin/sh
# ----------------Parameters---------------------- #
#$ -S /bin/sh
#$ -pe mthread 3
#$ -q sThC.q
#$ -l mres=24G,h_data=8G,h_vmem=8G
#$ -cwd
#$ -j y
#$ -N sharkmer
#$ -o sharkmer.log
#
# ----------------Modules------------------------- #
  module load bio/sharkmer/0.2.0_93ee045
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
#
cat ./sharkmer_samples.txt | while read SAMPLE 
do 
 sharkmer \
 --max-reads 1000000 \
 -s ${SAMPLE} -o ${SAMPLE}_sharkmer_output/ \
 --pcr "forward_primer_sequence, reverse_primer_sequence, expected_length, name_of_primer" \
 ./${SAMPLE}_R1_PE_trimmed.fastq ./${SAMPLE}_R2_PE_trimmed.fastq 
done 
#
echo = `date` job $JOB_NAME done
```

