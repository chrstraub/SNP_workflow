# Snakemake workflow for read mapping and SNP calling

This workflow uses snakemake for quality checking of raw sequences and trimming of raw sequences. It uses unique conda environments for each role of the workflow.

1. Make index for bowtie2
2. Map reads against reference (FA1090) using bowtie2 / bwa
3. convert sam to bam and sort
4. SNP calling

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
The output automatically goes into the output folder in SNP_workflow. Log files will be automatically moved to the logs folder after it finished.

```bash
#Perform a dry run to make sure all files are handled correctly
snakemake -n --use-conda
snakemake --use-conda 
#the workflow can fail at the multiqc step, because the system doesn't write the files fast enough. can include flag to wait 60 s (5s is default):
snakemake --use-conda --latency-wait 60

```


## Using SLURM to submit computationally heavy jobs
You can also SLURM to dispatch the scripts to the HPC. Edit the file `config/cluster.json` and update with specificities for your SLURM configuration.

```bash
#Dry-run of snakemake
snakemake --use-conda -n -R -j 4 --cores 12 -p --cluster-config ../config/cluster.json --cluster "sbatch -A {cluster.account} -p {cluster.partition} --ntasks {cluster.ntasks}" --latency-wait 120

#submit real job
snakemake --rerun-incomplete --use-conda -R all -j 4 --cores 12 -p --cluster-config ../config/cluster.json --cluster "sbatch -A {cluster.account} -p {cluster.partition} --ntasks {cluster.ntasks}" --latency-wait 300

```

for some reason it skips pre-fastqc and pre-multiqc - perhaps because I am using same input files for prefastqc and trim??


also need to find a way to tidy up .out from slurm
...work in progress