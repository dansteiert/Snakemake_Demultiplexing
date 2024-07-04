# Demultiplex Files in many Splits
 - Create Symlinks
 - Split FIles (fastqsplitter)
 - demultiplex files (demultiplex)
 - merge files (cat)

# Prerequisites
Conda environment inlcuidng snakemake and apptainer


## Recommended Installation
```
conda create --name snakemake -c bioconda -c conda-forge apptainer snakemake pandas
conda activate snakemake
pip install snakemake-executor-plugin-slurm (for usage with slurm!)
```

# Prepare Pipeline Inputs
## Pipeline Config File
See config/config.yaml

```
wrkdir: Path
logdir: Path
split_count: 10
sample_name: Sample1
index_file: demultiplexing_matrix_file_Sample
indices_name: ["Index_Name1","Index_Name2","Index_Name3",...]
metadata_file: Path
```



## Profile Config
See profile/config.yaml

```
jobs: 60 # number of concurrent jobs (if slurm is used otherwise #Jobs refer    ece to the available cores)
### using slurm as executor engin
#slurm: True  # To be used in snakemake 7
executor: slurm # to be used in snakemake 8 #  install plugin: pip install snakemake-executor-plugin-slurm

### using a containerized version
#use-singularity: True # snakemake7
software-deployment-method: apptainer # snakemake 8
singularity-prefix: "/data/cephfs-1/work/groups/keller/users/steiertd_c/.apptainer/"
singularity-args: "-B /data/cephfs-1/work/projects/scrnaseq-um/:/data/cephfs-1/work/projects/scrnaseq-um/,/data/cephfs-1/scratch/projects/scrnaseq-um/:/data/cephfs-1/scratch/projects/scrnaseq-um/"  # currently not working in snakemake 8

### using conda to manage environments
use-conda: True
# conda-prefix: Path were environments should be stored

### Snakemake behaviour
latency-wait: 60
keep-going: True
printshellcmds: True
rerun-incomplete: True
restart-times: 0
# delete-temp-output: True
```

## Metadata File
Needs to include the following entires (case-insensitive):
```
  1 Sample_Name,Read,Absolute_Path
  2 A4069_3_I1_final_pool_S3_L005,R1,/data/cephfs-1/scratch/projects/scrnaseq-um/Fastq/A4069/A4069_3_I1_final_pool_S3_L005_R1_001.fastq.gz
  3 A4069_3_I1_final_pool_S3_L005,R2,/data/cephfs-1/scratch/projects/scrnaseq-um/Fastq/A4069/A4069_3_I1_final_pool_S3_L005_R2_001.fastq.gz
  4 A4069_3_I1_final_pool_S3_L006,R1,/data/cephfs-1/scratch/projects/scrnaseq-um/Fastq/A4069/A4069_3_I1_final_pool_S3_L006_R1_001.fastq.gz
  5 A4069_3_I1_final_pool_S3_L006,R2,/data/cephfs-1/scratch/projects/scrnaseq-um/Fastq/A4069/A4069_3_I1_final_pool_S3_L006_R2_001.fastq.gz
  6 A4069_4_I1_final_pool_S4_L007,R1,/data/cephfs-1/scratch/projects/scrnaseq-um/Fastq/A4069/A4069_3_I1_final_pool_S4_L007_R1_001.fastq.gz
  7 A4069_4_I1_final_pool_S4_L007,R2,/data/cephfs-1/scratch/projects/scrnaseq-um/Fastq/A4069/A4069_3_I1_final_pool_S4_L007_R2_001.fastq.gz

```
One Metadata File can accomodate metadata for multiple runs!

# Running the pipeline 
## Conda
Pipeline was developed to run with conda environments and slurm

## Singularity /Apptainer

[//]: # (Pipeline was developed to run with apptainer and slurm)

```snakemake --configfile /PATH/config.yaml --profile /PATH/profile --snakefile /PATH/workflow/Snakefile```

### exemplary slurm call
```
#!/bin/bash
#SBATCH --job-name=Demultiplexing_Sample
#SBATCH --ntasks=1
#SBATCH --mem=1G
#SBATCH --cpus-per-task=1
#SBATCH --time=5-00:00:00
#SBATCH --qos=standard
#SBATCH --output=/PATH/%x.log
#SBATCH --error=/PATH/%x.log
#SBATCH --partition=compute

source ~/.bashrc

mkdir PATH
cd PATH # here under .snakemake configuarations are output
conda activate snakemake
snakemake --configfile /PATH/config.yaml --profile /PATH/profile --snakefile /PATH/workflow/Snakefile
```
