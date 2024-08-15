# How to run sharkmer on Hydra using a loop

Link to original sharkmer page
https://github.com/caseywdunn/sharkmer

## Step 1
### Unzip the read files
All reads files must be unzipped for sharkmer to run. The commnand gunzip can be used. The unzipped files will
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
 
 1. For each sample create a unique sample ID (This ID should be at the beginning of the file names for the unzipped read files for each sample). 
 2. Create a text file called "sharkmer_samples.txt" with each of the unique sample IDs (one per sample) from step 1 in a
 column, one sample ID per line.

```
Example_Sample_ID_1
Example_Sample_ID_2
Example_Sample_ID_3
```

 4. The reads files should also end with "_R1_PE_trimmed.fastq" and "_R2_PE_trimmed.fastq" after 
 the unique sample name for the forward and reverse reads (R1 and R2).
 
 Or, the job file below can also be modified to match your reads file names after the sample IDs if desired.

## Step 3
### Run the sharkmer job file on Hydra
Modify the commands in the job file below and run it in same directory as the unzipped reads files and the "sharkmer_samples.txt" file. 
NOTE - Paths in the job file can also be changed to desired directories.

### How to modify the sharkmer commands in the job below
-- max-reads, this is usually set to 1000000 as recommended by the manual. More can be added if desired.


-s, this commnd has the variable ${SAMPLE} which will be each sample ID name listed in the "sharkmer_samples.txt" file. This command can be left alone.

-o, output directories will be created for each sample ID and end with "_sharkmer_output". This can be left alone or modified if desired.

--pcr, for this command enter the forward and reverse primers sequences, the expected length of the PCR sequence (overestimate the length), and the name of the primer set.

NOTE - Several primer pairs can be run at the same time. Simply add another line with the command --pcr and add the appropriate primer information.


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

After running sharmer it is a good idea to delete the very large unzipped reads files from Hydra to save space.
