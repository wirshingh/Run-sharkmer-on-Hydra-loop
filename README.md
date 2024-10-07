# How to run sharkmer on Hydra using a loop

Link to original sharkmer page
https://github.com/caseywdunn/sharkmer

## Step 1
### Unzip the read files
Sharkmer reqpuires unzipped files. The normally-used "fastq.gz" files will need to be unzipped. The gunzip command will be used in this step to uncompress the forward and reverse read files (R1 and R2).

Save the following script as "unzip_reads.job" to the directory where the R1 and R2 "fastq.gz" read files are located and run the job (qsub unzip_reads.job). 
This script will copy the compressed R1 and R2 read files into a new directory named "unzipped_reads", and convert them to uncompressed ".fastq" files. 
#### Note: The uncompressed files will be large.

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
mkdir -p unzipped_reads
cp *.gz ./unzipped_reads
cd ./unzipped_reads
gunzip *.gz
#
echo = `date` job $JOB_NAME done

```
## Step 2
 ### Run Sharkmer on Hydra using a loop
For this script to work, the forward and reverse uncomporessed read files should end with "_R1_PE_trimmed.fastq" and "_R2_PE_trimmed.fastq". Alternatively, the job file below can also be modified to reflect your file names. 
Save the job file below as "sharkmer_loop.job". Run the job in same directory as Step 1 (qsub sharkmer_loop.job).

### How to modify the sharkmer commands
-- max-reads, this is usually set to 1000000 as recommended by the manual. More can be added if desired.

-s, this commnd has the variable ${SAMPLENAME} and is the Sample ID for that sample. This command can be left alone.

-o, output directories will be created for each sample ID and end with "_sharkmer_output".

--pcr, enter the forward and reverse primers sequences, the expected length of the PCR sequence (overestimate the length), and the name of the primer set. There should be no spaces. For example, --pcr "GACTGTTTACCAAAAACATA,AATTCAACATCGAGG,1000,16s" 

NOTE - Several primer pairs can be run at the same time. Simply add another line with the command --pcr and add the appropriate primer information.

#### After running sharkmer it is a good idea to delete the directory with the (very large) unzipped read files from Hydra to save space.

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
  module load bio/sharkmer/6f393f4
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
#
cd ./unzipped_reads
for GETFILENAME in ./*_R1_PE_trimmed.fastq
do 
SAMPLENAME=$(basename "$GETFILENAME" _R1_PE_trimmed.fastq)
 sharkmer \
 --max-reads 1000000 \
 -s ${SAMPLENAME} -o ${SAMPLENAME}_sharkmer_output/ \
 --pcr "forward_primer_sequence,reverse_primer_sequence,expected_length,name_of_primer" \
 ./${SAMPLENAME}_R1_PE_trimmed.fastq ./${SAMPLENAME}_R2_PE_trimmed.fastq 
done 
mkdir -p ../sharkmer_results
mv *_sharkmer_output ../sharkmer_results
#
echo = `date` job $JOB_NAME done
```
