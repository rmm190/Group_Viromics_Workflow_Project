# Group_Viromics_Workflow_Project
BIOL-3615 Group Project

Notes 3/12: 

**What is the goal of this project?** 
The goal of this project is to perform a bioinformatics workflow, including trimming reads, assembling contigs, identifying viruses and analyzing diversity in our sample (accession # = SAMN08784142) using R. 

**What files will be found in this repository?**
1st step: directory named “fastq” for original *.sra files that were made into fastq format 
2nd step: put these *sra files once they are unzipped in a folder named “raw” to organize these raw files before trimmomatic 
---Also put these files into the class bucket


**What is our goal for today?**
Our goal for today is cleaning our reads. We will perform fastqc, then Trimmomatic, then fastqc again on our sample. 

**What steps did we take & why?** 
#Load Anaconda module and SRA tools: 
$ module load anaconda3
$ conda create -n sra_env -c bioconda sra-tools
#Initiate and active conda environment: 
$ conda init
$ conda activate sra_env
#Make directories for file organization, change into directory for raw files
$ mkdir fastqfiles 
$ cd fastqfiles
$ mkdir SRR6996006
$ cd SRR6996006
$ mkdir raw
$ mkdir cleaned_reads
$ mkdir assembly
$ cd raw
#Fetch files and change to fastq format, compress
$ prefetch SAMN08784142
$ fasterq-dump *.sra
$ gzip *.fastq

