# What is the goal of this project?

The goal of this project is to perform a bioinformatics workflow, including trimming reads, assembling contigs, identifying viruses and analyzing diversity in our sample (accession # = SAMN08784142) using R.
Get familiar with how to use bioinformatics tools and make a workflow relevant to scientific questions. 

# What files will be found in this repository?
1st step: directory named “fastq” for original *.sra files that were made into fastq format 
2nd step: put these *sra files once they are unzipped in a folder named “raw” to organize these raw files before trimmomatic 
Also put these files into the class bucket

# What is our goal for today, 3/12?
Our goal for today is cleaning our reads. We will perform fastqc, then Trimmomatic, then fastqc again on our sample.

***See the README file for all results images****

## Step 1: Downloading our fastq files.
These additional steps are because our files aren't available in the normal ftp format.
```
# Go to home directory and load anaconda module. Install SRA-tools through an environment.

module load anaconda3
conda create -n sra_env -c bioconda sra-tools
```
```
# Initiate and active conda environment. Make sure to say "yes" when prompted by the tool. 

conda init
conda activate sra_env
```
```
#Fetch files and change to fastq format, compress. You might have to be a bit patient with this step.

prefetch SAMN08784142
fasterq-dump *.sra
gzip *.fastq
```
## Step 2: Getting organized to set ourselves up for success throughout our workflow
```
# Make directories for file organization, change into directory for raw files

mkdir fastqfiles 
cd fastqfiles
mkdir SRR6996006
cd SRR6996006
mkdir raw
mkdir cleaned_reads
mkdir assembly
cd raw
```
## Step 3: Running FastQC of your raw data
```
# FastQC of raw data. This will evaluate the baseline state of our DNA so that we can make decisions about how to clean it later in our workflow.
# Enter interactive mode on a compute node (from where you are)
srun --pty bash

#Load FastQC
module load fastqc

#Confirm that FastQC is available and see options
fastqc -h

# Create a directory for the reports (Once per dataset)
mkdir -p fastqc_out

#Run FastQC. The -o line tells the program to put the .html and .zip outputs into your output directory
fastqc -o fastqc_out SRR6996006.sra_1.fastq.gz SRR6996006.sra_2.fastq.gz
```
The files produced should be:
yourfile_fastqc.html
yourfile_fastqc.zip

## Step 4: Run Trimmomatic on the files.
Running trimmomatic is necessary to remove adapters and to trim low quality bases. The quality of reads often tails off at the end of our reads (as evidenced from our initial FastQC analysis. Trimmomatic will remove those while also getting rid of reads that are simply too short for our future needs.

```
# Trimmomatic Script
#!/bin/bash#SBATCH --mail-type=END,FAIL --mail-user=rmm190@georgetown.edu
#SBATCH --job-name="trim_S1"
#SBATCH --output="%x.o%j"
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --time=03:00:00
#SBATCH --mem=10G

#Load Trimmomatic module (“aliases” needed for GU HPC setup here)
shopt -s expand_aliases
module load trimmomatic

#Define paths and variables
R1=/home/rmm190/group_viromics_project/fastq_raw/sample1_R1_uncleaned_fastqc.zip.gz
R2=/home/rmm190/group_viromics_project/fastq_raw/sample1_R2_uncleaned_fastqc.zip.gz
OUTDIR=/home/rmm190//home/rmm190/group_viromics_project/trim_out
mkdir -p $OUTDIR

#Load Trimmomatic module (“aliases” needed for GU HPC setup here)
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
```
After running Trimmomatic, the quality of our reads significantly improved. Our adapter content was reduced to 0, and while we now have a smaller number of total reads, they are of a much higher quality.

## Step 5: FastQC of clean data:

```
Enter interactive mode on a compute node (from where you are)
srun --pty bash
module load fastqc
fastqc -h
mkdir -p fastqc_out
fastqc -o fastqc_out SRR*
```
Additionally, the trimmed files need to be put in the bucket after they're checked with FastQC.

**Result Files in ReadMe**

# What is our goal for today 3/17?
Assemble contigs by running Megahit software. View how many contigs were assembled and the quality with seqkit.  

Overview of project organization to this point:
/home/netID/project
 - Raw/
 - Fastqc
     - Has outputs from fastqc runs
 - Trimmomatic
     - Has outputs from trimmomatic runs
 - Metagit
     - Has outputs from megahit step (running today)
 - Slurmscripts
      - Has all of the slurm scripts for any step requiring one
 - Logs
       - Has error logs and output logs for anything requiring a slurm script.

Before starting the megahit workflow, you'll want to get organized.
- Make sure you have cleaned fastqc files (after FastQC and Trimmomatic)
- Those files are paired-end with clear names
- You know the absolute path to those files
- You have the output directory (megahit folder)

## Step 1: Install Megahit
```
module load mamba/ 
```

## Step 2: Create an environment for megahit 
```
mamba create -y -n megahit-env -c conda-forge -c bioconda megahit
```

## Step 3: Run Megahit
```
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

```

## Step 4: Find your results
No new code required, but you'll want to navigate to your output directory. Make sure you see a file called "final.contigs.fa" which contains your assembled contigs with which you need to proceed. You should also check the log in the log directory and see severarl intermediate files from the assembly process

## Step 5: Checking quality 
```
# Downloading seqkit. Seqkit is a "lightweight" command that you can use on the login node. Seqkit allows you to manipulate and analyze FASTA files.
module load mamba/
mamba activate megahit-env        
mamba install -c bioconda seqkit

# Running onto your assembly 
seqkit stats -a final.contigs.fa
```
### What was the purpose of running megehit?
Megahit is an assembly program. It's when you take your DNA reads and assemble them into longer sequences (contigs, for our purposes). These contigs will make it easier for us to analyze and annotate our reads. As part of its assembly process, megahit uses what's called de Bruijn graphs. They break reads into k-mers of varying lengths, make graphs of the data, and then follow these graphs to produce reads. Using de Bruijn graphs is useful for metagenomics and larger sets of data that will contain DNA from different organisms. 

**See README for results**


# What is our goal for today 3/19?
Identify viral contigs & cluster vOTUs using Virsorter technology

Before beginning, make sure to create new directories for virsorter and votus so that you have places to put your data.

## Some background on virsorter and the process of identifying viruses from a dataset
The main idea is you want to filter out the "virus-like" reads from everything else. The difficulty lies in determining what is "virus-like" and what is not becuase there is no universal 16S rRNA gene like there is in bacteria for viruses. However, that doesn't mean there aren't ways to make this identification.

Reference-based comparison:
 - BLAST-like search of predicted proteins against viral databases (similar sequences to known viral proteins)
 - Number of kits to viral proteins, especially "hallmark" genes (major capsid proteins, terminases, portal proteins, integrases)

Genome patterns:
 - Enriched in viral genes, few typical cellular genes
 - Enriched in genes annotated as "hypothetical proteins"
 - Presence of short, phage-like genomes (circular contigs with matching ends)
Virsorter is a program that incorporates all of this information and makes decisions about whether reads are "virus-like" or not.

## Step 1: Install Virsorter
```
module load mamba
mamba create -y -n vs2-env -c conda-forge -c bioconda virsorter
```

## Step 2: Download databases for virsorter
# Install by creating an environment, installing virsorter
```
mamba activate vs2-env

# Just in case there is a failed attempt before, this step removes the whole directory specified by -d (something that we had a lot of problems with in class.
rm -rf db 					
virsorter setup -d db -j 4		# run setup for the database
```

## Step 3: Run Virsorter2
```
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
```

## Step 4: Find your results: 
After VirSorter2 finishes, navigate to your output directory. You should see several files.

They should include:
 - final-viral-boundary.tsv
 - final-viral-score.tsv
 - final-viral-combined.fa **This is what you need to proceed!

## Step 5: Filtering out contigs that are less thatn 5kb
Anything less than 5kb is not going to be useful for clustering into vOTUS. They simply don't have enough information to accurately be clustered.

```
#You're going to be using seqkit to filter out anything less, so you'll need to load and activate the megahit environment again.
module load mamba/
mamba activate megahit-env        

# Count contigs to see how many you have that have a minimum length of 5kb. Then, make a file just for those and check the number of contigs that are in your new file (called final-viral-combined_min5kb.fa)
seqkit seq -m 5000 final-viral-combined.fa | grep -c “>”
seqkit seq -m 5000 final-viral-combined.fa > final-viral-combined_min5kb.fa

See README for results 
```
## Step 6: Install vclust
This next step begins the clustering process for all of the representative contigs that have made it through the workflow thus far. vOTUS are OTU-like units, grouped by clustering viral contigs/genomes be sequence similarity. They're used because viruses lack a universal gene marker (like 16S) to compare. Additionally, they serve as operational "species-like" units for estimating viral diversity, community structure, and dynamics.

Depending on the tool or script you use, you can get different results
We will use vClust, which relies on 95% sequence identity and 85% total coverage.

```
#Install vclust by creating an environment and activating it
module load mamba
mambe create -n votu-env -c bioconda -c conda-forge vclust
mamba activate votu-env
```
## Step 7: Run vclust

```
#Prefilter similar genome sequence pairs before conducting pairwise alignments.
vclust prefilter -i final-viral-combined_min5kb.fa

#Align similar genome sequence pairs and calculate pairwise ANI measures
vclust align final-viral-combined_min5kb.fa -o ani.tsv --filter fltr.txt

#Cluster genome sequences based on given ANI measure and miniumum threashold (these files were generated in the previous steps)
vclust cluster -i ani.tsv -o clusters.tsv --ids ani.ids.tsv --metric ani --ani 0.95 --out-repr
```
## Step 8: Pulling out the vOTU sequences
```
#Make a list of the vOTU headers
# Prints only the second column. Then takes the stream and sorts the values and removes duplicates
awk '{print$2}' clusters.tsv | sort -u > votu_seeds.txt

#Now, put these vOTU "seed" sequences into a new file.
seqkit grep -f votu_seeds.txt final-viral-combined_min5kb.fa > votus_final.fa

# Sanity check

#Should equal number of clusters
wc -l votu_seeds.txt

# Should match votus_seeds.txt
grep -c ">" votus_final.fa
```

# What is our goal for today, 3/24?
Check quality of clusters with Checkv. Checkv accomplishes completeness, contamination, and provirus trimming. Because viral contigs can be partial, chimeric, or include host DNA, we want a way to evaluate the quality of each viral contig.

Some other things CheckV will be doing:
 - Identify and trim host regions (annotates ORFs and uses gene content, GC content, and other features to detect host-like segments)
 - Estimate completeness (it comeares each contig to a large database of complete viral genomes using amino-acid identity and alignment coverage eg 40% complete v. 90% complete)
 - Identify closed genomes (it looks for terminal repeats)
 - Assign quality tiers: it classifies each sequence into quality categories.

Then, run a bowtie script to create alignment files. Upload alignment files to the larger class data set for analysis. The purpose of running bowtie2 is for mapping our reads to the contigs. Quantifying abundance is a crucial aspect of bowtie2. Seeing how many reads align to each contig is a way to see which viruses are abundant and which ones are relatively rare.

## Step 1: Set up Checkv: 
```
#Make sure you're in the checkv folder when you're doing this! You don't want it to download in the wrong folder.
module load checkv
checkv download_database ./
```


## Step 2: Run Checkv

```
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
```

##  Step 3: Find results: 
After checkv is complete, you'll see a number of files. The one you want to look at is quality_summary_votus.tsv


## Step 4: Grab the pooled vOTUs from the bucket:
This consists of the class's virsorter contigs, filtered for > 10kb, and clustered. Having this data is an important reference point so that we can match reads to more contigs.
```
gcloud storage cp gs://gu-biology-dept-class/ClassProject/votus_10kb_6samples.fna /home/jbs172/group_viromics_project/checkv
```

## Step 5: Loading Bowtie2

```
mkdir bowtie2 
mv votus_10kb_6samples.fna bowtie2
module load bowtie2
bowtie2-build votus_10kb_6samples.fna votu_index
```

## Step 6: Run Bowtie2

```
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

```

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





