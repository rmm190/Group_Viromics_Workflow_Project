# Group_Viromics_Workflow_Project
BIOL-3615 Group Project

# What is the purpose of this project?
As part of this group viromics workflow, we had seven different viromes sampled from a Swedish bog. Our goals were to repduce the bioinformatic tools used by a previous study to learn about a bioinformatics workflow. We began by learning the importance of notetaking, transparence, and reproducibility in bioinformatics studies. Then, we took our raw data, ran FastQC to get baseline data about our reads, ran it thorugh trimmomatic to clean them, and ran FastQC once again to verify our results. From these cleaned reads, we used megahit to assemble our contigs. To find viruses within our assembled contigs, we used virsorter2. We also clusterd our contigs into vOTUS using vclust and validated our data with checkv. Eventually, we mapped our reads using bowtie2 and analyzed our data with diversity statistics measures and heat maps in R. From our data, we were able to accomplish our goal of getting familiar with bioinformatics techniques as well as learning about viral diversity within a bog environment.

See MasterNotes.md for goals and code. The following readme contains all of our accompanying results graphs and images. 

**TRIMMOMATIC**
**Sample 1 R1_Cleaned_Paired**
<img width="442" height="265" alt="Screenshot 2026-03-27 at 10 06 02 PM" src="https://github.com/user-attachments/assets/c4dcb46e-35f2-4204-a934-b1bb36a9fcaf" />

**Sample 1_R1_Cleaned_Unpaired**
<img width="477" height="283" alt="Screenshot 2026-03-27 at 10 06 43 PM" src="https://github.com/user-attachments/assets/1b442a7b-4d67-46a5-b72b-c599a4295042" />

**Sample 1_R2_Cleaned_Paired**
<img width="452" height="256" alt="Screenshot 2026-03-27 at 10 07 14 PM" src="https://github.com/user-attachments/assets/5e8921c4-a359-4069-8b8f-0bdafc45541a" />

**Sample 1_R2_Cleaned_Unpaired**
<img width="428" height="262" alt="Screenshot 2026-03-27 at 10 07 33 PM" src="https://github.com/user-attachments/assets/3f9d561b-1008-4a47-88e2-82c0b6f1ace5" />

**Megahit Results**
<img width="793" height="46" alt="Screenshot 2026-03-27 at 10 08 11 PM" src="https://github.com/user-attachments/assets/d84d64f8-38f8-497a-a073-fcef014c7a4e" />

**Slurm Script for Bowtie** 
<img width="576" height="275" alt="Screenshot 2026-03-27 at 10 09 15 PM" src="https://github.com/user-attachments/assets/3048b06a-7d18-4727-bb65-d5e75441d86d" />

**vOTU Heatmap of Relative Abundance**
<img width="583" height="401" alt="Screenshot 2026-03-27 at 10 09 41 PM" src="https://github.com/user-attachments/assets/d247c29f-9f7e-49c7-a3aa-e1746178bf04" />

**Per Sample Richness**
<img width="452" height="314" alt="Screenshot 2026-03-27 at 10 10 06 PM" src="https://github.com/user-attachments/assets/6945b925-0ff9-4c0c-83a9-8b21add9afa2" />

**Per Sample Viral Diversity**
<img width="286" height="341" alt="Screenshot 2026-03-27 at 10 10 28 PM" src="https://github.com/user-attachments/assets/5ddb736a-cf8e-4a0d-a74d-1af807c47b25" />
