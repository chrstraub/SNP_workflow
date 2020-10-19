# Snakemake workflow for read mapping and SNP calling

This workflow uses snakemake for quality checking of raw sequences and trimming of raw sequences. It uses unique conda environments for each role of the workflow.

1. Make index of reference for BWA
2. Map reads against reference (*Neisseria gonorrhoeae* FA1090) using bwa. 
If using different ref, update ref in Snakefile and copy fna to `/ref/*`
3. convert sam to bam and sort bam
4. Mark duplicates using Picard
5. Run qualimap for quality control
6. Index bam files
4. SNP calling using freebayes

## How to use:

Make sure you have conda installed.
Create a new conda environment with snakemake and python.

```bash
git clone https://github.com/chrstraub/SNP_workflow

#if conda installed
conda create --env snakemake
conda activate snakemake
```
Create run-links to your raw sequencing files in the `run_links` folder.
Edit the file endings in first couple of lines in the Snakefile, e.g. currently it is set to `_R1_001.fastq.gz`.  
The output automatically goes into the respective folders `output` folder in SNP_workflow. 

**Directory tree**

```bash
SNP_workflow
|
├── config
│   └── cluster.json
├── logs
│   ├── bwa
│   ├── freebayes
│   ├── picard
│   └── slurm
├── output
│   ├── bam
│   ├── dedup
│   ├── multibamqc
│   ├── sam
│   ├── SNPcalls
│   └── sorted
├── ref
│   ├── Ngono_FA1090_genomic.gff
│   └── Ngono_FA1090.fna
├── run_links
└── workflow
    ├── Snakefile
    └── envs
        ├── bowtie.yaml
        ├── bwa.yaml
        ├── freebayes.yaml
        ├── picard.yaml
        └── qualimap.yaml



```

```bash
#Depending on the snakemake version - you have to go into the workflow folder to run snakemake. By default it should work in the overall folder.
#Perform a dry run to make sure all files are handled correctly
snakemake -n --use-conda
snakemake --use-conda 
#the workflow can fail, because the system doesn't write the files fast enough. can include flag to wait 60 s or longer (5s is default):
snakemake --use-conda --latency-wait 60

```

## Using SLURM to submit computationally heavy jobs
You can also SLURM to dispatch the scripts to the HPC. Edit the file `config/cluster.json` and update with specificities for your SLURM configuration.

```bash
#Dry-run of snakemake
snakemake --use-conda -n -R -j 12 -p --local-cores 1 --cluster-config ../config/cluster.json --cluster "sbatch -A {cluster.account} -p {cluster.partition} --ntasks {cluster.ntasks}" --latency-wait 120

#submit real job
snakemake --use-conda -R freebayes -j 12 -p --local-cores 1 --cluster-config ../config/cluster.json --cluster "sbatch -A {cluster.account} -p {cluster.partition} --ntasks {cluster.ntasks}" --latency-wait 300
##local rules are executed with 1 core
```

# To do
sort out log files for freebayes - still empty!

