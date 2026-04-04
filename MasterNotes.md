**What is the goal of this project?**

The goal of this project is to perform a bioinformatics workflow, including trimming reads, assembling contigs, identifying viruses and analyzing diversity in our sample (accession # = SAMN08784142) using R.
Get familiar with how to use bioinformatics tools and make a workflow relevant to scientific questions. 

**What files will be found in this repository?**
1st step: directory named “fastq” for original *.sra files that were made into fastq format 
2nd step: put these *sra files once they are unzipped in a folder named “raw” to organize these raw files before trimmomatic 
Also put these files into the class bucket

**What is our goal for today, 3/12? **
Our goal for today is cleaning our reads. We will perform fastqc, then Trimmomatic, then fastqc again on our sample.

***See the README file for all results images****

```$ module load anaconda3
$ conda create -n sra_env -c bioconda sra-tools ```

# Initiate and active conda environment: 

`$ conda init
$ conda activate sra_env`

# Make directories for file organization, change into directory for raw files

`$ mkdir fastqfiles 
$ cd fastqfiles
$ mkdir SRR6996006
$ cd SRR6996006
$ mkdir raw
$ mkdir cleaned_reads
$ mkdir assembly
$ cd raw `

Fetch files and change to fastq format, compress
$ prefetch SAMN08784142
$ fasterq-dump *.sra
$ gzip *.fastq
# FastQC of raw data:
# Enter interactive mode on a compute node (from where you are)
$ srun --pty bash
$ module load fastqc
$ fastqc -h
$ mkdir -p fastqc_out
$ fastqc -o fastqc_out SRR6996006.sra_1.fastq.gz SRR6996006.sra_2.fastq.gz


# Trimmomatic Script
#!/bin/bash#SBATCH --mail-type=END,FAIL --mail-user=rmm190@georgetown.edu
#SBATCH --job-name="trim_S1"
#SBATCH --output="%x.o%j"

#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --time=03:00:00
#SBATCH --mem=10G

# Load Trimmomatic module (“aliases” needed for GU HPC setup here)
shopt -s expand_aliases
module load trimmomatic

# Define paths and variables
R1=/home/rmm190/group_viromics_project/fastq_raw/sample1_R1_uncleaned_fastqc.zip.gz
R2=/home/rmm190/group_viromics_project/fastq_raw/sample1_R2_uncleaned_fastqc.zip.gz
OUTDIR=/home/rmm190//home/rmm190/group_viromics_project/trim_out
mkdir -p $OUTDIR

trimmomatic PE -threads $SLURM_CPUS_PER_TASK \
  $R1 $R2 \
  $OUTDIR/SRR6996006_R1_paired.fq.gz  $OUTDIR/SRR6996006_R1_unpaired.fq.gz \
  $OUTDIR/SRR6996006_R2_paired.fq.gz  $OUTDIR/SRR6996006_R2_unpaired.fq.gz \
  ILLUMINACLIP:/home/rmm190/adapters/TruSeq3-PE.fa:2:30:10 \
  LEADING:3 TRAILING:3 MINLEN:36


# FastQC of clean data:
# Enter interactive mode on a compute node (from where you are)
$ srun --pty bash
$ module load fastqc
$ fastqc -h
$ mkdir -p fastqc_out
$ fastqc -o fastqc_out SRR*

****Result Files in ReadMe**

**What is our goal for today 3/17?**
Assemble contigs by running Megahit software. View how many contigs were assembled and the quality with seqkit.  

# Step 1: Installing megahit by creating an environment → module loading mamba 
$ Module load mamba/ 

# Step 2: Create an environment for megahit 
$ mamba create -y -n megahit-env -c conda-forge -c bioconda megahit

# Script for MegaHit

#!/bin/bash
#SBATCH --job-name=megahit_SRR6996006
   	# how job appears in the queue
#SBATCH --mail-type=END,FAIL --mail-user=jm3448@georgetown.edu
#SBATCH --nodes=1
#SBATCH --cpus-per-task=8                 
#SBATCH --mem=32G                         
#SBATCH --time=03:00:00                   
#SBATCH --output=/home/jm3448/Project1/logs/megahit_test1.%j.out      
#SBATCH --error=/home/jm3448/Project1/logs/megahit_test1.%j.err       

#note %j = job ID

# ==== Load mamba/conda module (students: no need to change) ====
module load mamba
source $(mamba info --base)/etc/profile.d/conda.sh

# Activate the environment where you had MEGAHIT installed
conda activate megahit-env

# ==== Set paths and filenames (students: edit this block!) ====

# Directory where the cleaned reads live
READDIR=/home/jm3448/Project1/trimmed_out

# Input read files (paired-end)
READ1=${READDIR}/sample1_R1_cleaned_paired.fq.gz
READ2=${READDIR}/sample1_R2_cleaned_paired.fq.gz

# Output directory (give it a name, it will be created by MEGAHIT)
OUTDIR=/home/jm3448/Project1/megahit/jm3448_megahit_out

# ==== Run MEGAHIT ====

megahit \
  -1 ${READ1} \
  -2 ${READ2} \
  -t ${SLURM_CPUS_PER_TASK} \
  -o ${OUTDIR}

echo "Done. Contigs should be in: ${OUTDIR}/final.contigs.fa"


# Step 5: Checking quality 
# Downloading seqkit
$ module load mamba/
$ mamba activate megahit-env        
$ mamba install -c bioconda seqkit

# Running onto your assembly 
$ seqkit stats -a final.contigs.fa

**See README for results**

**What is our goal for today 3/19**
Identify viral contigs & cluster vOTUs using Virsorter technology

# Install Virsorter
$ module load mamba
$ mamba create -y -n vs2-env -c conda-forge -c bioconda virsorter

# Download databases for virsorter
# Install by creating an environment, installing virsorter
$ mamba activate vs2-env
$ rm -rf db 					
$ virsorter setup -d db -j 4		# run setup for the database

# Virsorter Slurm script

#!/bin/bash
#SBATCH --job-name=virsorter_SRR6996006 
#SBATCH --nodes=1
#SBATCH --cpus-per-task=8                 
#SBATCH --mem=20G                         
#SBATCH --time=03:00:00     
#SBATCH --mail-type=END,FAIL
#SBATCH --mail-user=jm3448@georgetown.edu             
#SBATCH --output=/home/jm3448/Project1/logs/virsorter_test1.%j.out          
#SBATCH --error=/home/jm3448/Project1/logs/virsorter_test1.%j.err       

# ==== Load mamba (students: no need to change) ====
module load mamba
source $(mamba info --base)/etc/profile.d/conda.sh

# Activate the environment where you had VirSorter2 installed
mamba activate vs2-env

# ==== Set paths and filenames (students: edit this block!) ====
#set up directories
INDIR=/home/jm3448/Project1/megahit/jm3448_megahit_out	 #directory where input will come from
OUTROOT=/home/jm3448/Project1/virsorter/	 #directory output will go
mkdir -p "${OUTROOT}"					 #new directory to be created for output files

SAMPLE_ID=SRR6996006						 #just the basic sample name (sample2 ?)
INPUT="${INDIR}/final.contigs.fa"			 #contig file name/location
OUTDIR="${OUTROOT}/vs2-${SAMPLE_ID}" 			 #where you’ll find the outputs
mkdir -p "${OUTDIR}"


# ==== Run virsorter2 with >5kb cutoff and DNA virus categories first
echo "Running VirSorter2 on ${INPUT}"
virsorter run \
  -w "${OUTDIR}" \
  -i "${INPUT}" \
  --keep-original-seq \
  --include-groups dsDNAphage,NCLDV,ssDNA \
  --min-length 5000

echo "Done."


# Visualizing results: 

$ module load mamba/
$ mamba activate megahit-env        

# Count contigs + make file with those. Check the number 
$ seqkit seq -m 5000 final-viral-combined.fa | grep -c “>”
$ seqkit seq -m 5000 final-viral-combined.fa > final-viral-combined_min5kb.fa

See README for results 

**What is our goal for today 3/24?**
Begin mapping reads to vOTUs and check quality of clusters with Checkv. Checkv accomplishes completeness, contamination, and provirus trimming. Then, run a bowtie script to create alignment files. Upload alignment files to the larger class data set for analysis. 

# Set up Checkv: 
$ Module load checkv
$ Checkv download_database

# Write and send slurm script for checkv

#!/bin/bash
#SBATCH --job-name=checkv
#SBATCH --output=/home/rmm190/group_viromics_project/logs/checkv-%j.out
#SBATCH --error=/home/rmm190/group_viromics_project/logs/checkv-%j.err
#SBATCH --time=03:00:00
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --mem=16G
#SBATCH --mail-type=END,FAIL
#SBATCH --mail-user=rmm190@georgetown.edu


# ==== Load checkv program module (students: no need to change) ====

module load checkv


# ==== Set variables, paths, and filenames (students: edit this block!) ====

CHECKVDB="/home/rmm190/iceland/virusid/checkv/checkv-db-v1.5"

SAMPLE_ID="vOTUs"
INPUT="/home/rmm190/group_viromics_project/votus/votus_10kb_6samples.fna"
OUTDIR="/home/rmm190/group_viromics_project/checkv/${SAMPLE_ID}"

mkdir -p "${OUTDIR}"


# ==== run checkv (students: no need to change. The second line is the command) ====
echo "Running CheckV on ${INPUT}"
checkv end_to_end "${INPUT}" "${OUTDIR}" -d "${CHECKVDB}" -t ${SLURM_CPUS_PER_TASK}
echo "Done."

#  Find results: 
# Look at quality_summary_votus.tsv 
# https://drive.google.com/drive/u/0/folders/1X7hH0vhxEvMmJ2MWs0O7tszbZn3z4swK 
# **Insert notes here** How many complete? High quality? Low quality? 

# Grab the pooled vOTUs from the bucket 
$ gcloud storage cp gs://gu-biology-dept-class/ClassProject/votus_10kb_6samples.fna /home/jbs172/group_viromics_project/checkv

# Loading Bowtie2
$ mkdir bowtie2 
$ mv votus_10kb_6samples.fna bowtie2
$ module load bowtie2
$ bowtie2-build votus_10kb_6samples.fna votu_index

# Write and send slurm script for Bowtie 
#!/bin/bash
#SBATCH --job name=bowtie2_vOTUs                 
#SBATCH --output=/home/jm3448/Project1/logs/bowtie-%j.out
#SBATCH --error=/home/jm3448/Project1/logs/bowtie-%j.err
#SBATCH --mail-type=END,FAIL
#SBATCH --mail-user=jm3448@georgetown.edu
#SBATCH --time=8:00:00                                
#SBATCH --mem=16G                        

# ---------SET UP----------
SAMPLE= ”sample1_jm3448”  	#whatever your sample # is!
INDEX="home/jm3448/Project1/bowtie2/index/votu_index"
OUTPUTDIR="home/jm3448/Project1/bowtie2/${SAMPLE}"

# --------- LOAD MODULES ----------
module purge
module load bowtie2/2.5.4

# --------- RUN BOWTIE2 AND PIPE TO SAMTOOLS ----------

# First make output and log directories; move into OUTPUTDIR
mkdir -p "${OUTPUTDIR}"
cd "${OUTPUTDIR}"
mkdir -p logs

echo "Running bowtie2 on sample ${SAMPLE}"

bowtie2 -p 8 -x "${INDEX}" -1 "/home/jm3448/Project1/trimmed_out/sample1_R1_cleaned_paired.fq.gz" -2 "/home/jm3448/Project1/trimmed_out/sample1_R2_cleaned_paired.fq.gz" \
| samtools view -bS - > "${SAMPLE}.bam"

echo "Finished running bowtie2 and performing compression"

#---------sort and index files
echo "Sorting"
samtools sort "${SAMPLE}.bam" > "${SAMPLE}_sorted.bam"

echo "Indexing"
samtools index "${SAMPLE}_sorted.bam"

echo "Finished ${SAMPLE}"

**See README for graphs/images** 

**What is the goal of today 3/26?**
With all of the class data, use R studio programs to visualize metrics such as abundance, diversity and richness in plots. This helps us better understand the viral community ecology. 

# Visualizing the abundance data across the class with a heat map in R studio. Here is the script we used (next page): 



 
# ==== EDIT THESE THREE LINES ====
filename <- "/Users/janesmith/Downloads/R Studio Analysis/ClassProject_votus_12367_coverm.tsv"  # Excel file name
tpm_threshold <- 10                                  # keep vOTUs with max TPM > this
heatmap_colors <- c("#440154", "#31688e", "#35b779", "#fde725")

# =================================

library(readxl)
library(pheatmap)

# 1. Read the Excel file
cov <- read_tsv(filename)

# 2. Keep Contig and TPM columns only
tpm_cols <- grepl("TPM$", names(cov))
cov_tpm <- cov[ , c("Contig", names(cov)[tpm_cols])]

# Remove S1k141_26921_full
cov_tpm <- subset(cov_tpm, Contig != "S1k141_26921||full")

# 3. Optional filter: drop very low-abundance vOTUs
cov_tpm$max_tpm <- apply(cov_tpm[ , -1], 1, max, na.rm = TRUE)
cov_tpm <- subset(cov_tpm, max_tpm > tpm_threshold)
cov_tpm$max_tpm <- NULL

# If everything got filtered out (threshold too high), warn and stop
if (nrow(cov_tpm) == 0) {
  stop("No vOTUs passed the TPM threshold. Try lowering tpm_threshold.")
}

# 4. Make matrix for heatmap (rows = vOTUs, cols = samples)
mat <- as.matrix(cov_tpm[ , -1])
rownames(mat) <- cov_tpm$Contig

# 5. Log-transform for nicer color scaling
mat_log <- log10(mat + 1)

# 6. Draw heat map
pheatmap(mat_log,
         cluster_rows = TRUE,
         cluster_cols = TRUE,
         scale = "none",
         color = colorRampPalette(heatmap_colors)(100),
         fontsize_row = 8,
         fontsize_col = 10,
         main = "vOTU relative abundance (log10 TPM + 1)")


# Also created a plot of per sample richness and alpha diversity, using this script: 
## Install once if needed:
install.packages(c("vegan", "pheatmap", "RColorBrewer"))

## Each session:
library(vegan)
library(pheatmap)
library(RColorBrewer)

## 1. Read your exact file
abund_raw <- read.csv(
  "~/Downloads/R Studio Analysis/ClassProject_votus_12367_coverm_TPM_3votusremoved.csv",
  header = TRUE,
  check.names = FALSE
)

## 2. First column is vOTU / Contig ID. Make that the rownames.
rownames(abund_raw) <- abund_raw$Contig

## 3. Drop the Contig column so we have a numeric matrix only
abund <- abund_raw[, -1]

## 4. Quick sanity check
dim(abund)
head(rownames(abund))   # vOTU IDs
colnames(abund)         # sample1_jbs172_sorted TPM, etc.

##per sample richness and diversity
## Transpose: now rows = samples, columns = vOTUs
abund_t <- t(abund)

## Optional: drop samples with total abundance = 0
abund_t <- abund_t[rowSums(abund_t) > 0, , drop = FALSE]

## 1. Species richness per sample (number of vOTUs with >0)
richness <- specnumber(abund_t)   # named vector, names are sample IDs

## 2. Shannon diversity per sample
shannon <- diversity(abund_t, index = "shannon")

## 3. Combine into a data.frame
alpha_div <- data.frame(
  Sample   = rownames(abund_t),
  Richness = richness,
  Shannon  = shannon
)

alpha_div

## Barplot of richness PER SAMPLE
barplot(
  alpha_div$Richness,
  names.arg = alpha_div$Sample,   # force sample IDs on x-axis
  las = 2,
  ylab = "vOTU richness (count)",
  main = "Per-sample viral richness"
)

## Barplot of Shannon PER SAMPLE
barplot(
  alpha_div$Shannon,
  names.arg = alpha_div$Sample,
  las = 2,
  ylab = "Shannon diversity",
  main = "Per-sample viral diversity"
)





