# cutTag-pipeline

[![Linux](https://svgshare.com/i/Zhy.svg)](https://svgshare.com/i/Zhy.svg)
![ci/cd status](https://github.com/maxsonBraunLab/cutTag-pipeline/actions/workflows/test.yaml/badge.svg?branch=test)
[![snakemake minimum](https://img.shields.io/badge/snakemake->=5.32-<COLOR>.svg)](https://shields.io/)
![Maintainer](https://img.shields.io/badge/maintainer-gartician-blue)

Snakemake pipeline for Cut&amp;Tag analysis 

# Setup

## 1. Configure the project directory

```bash
# cd into a project directory

# type the following to get a copy of the pipeline
git clone https://github.com/maxsonBraunLab/cutTag-pipeline.git

#create a directory for your fastq files
cd cutTag-pipeline
mkdir -p data/raw

# link fastqs to data/raw
ln -s /path/to/fastq/files/sample1_R1.fastq.gz data/raw
ln -s /path/to/fastq/files/sample1_R2.fastq.gz data/raw
ln -s /path/to/fastq/files/sample2_R1.fastq.gz data/raw
ln -s /path/to/fastq/files/sample2_R2.fastq.gz data/raw
...

# make scripts executable
chmod +x src/*.py src/*.sh *.sh
```

**IMPORTANT**

After creating the symlinks, rename all the symlinks in data/raw to the following format: `{condition}_{replicate}_{mark}_{R1|R2}.fastq.gz`

For example, a file with this original name **LIB200706TB_M6Q3_RBP1_S93_L001_R1_001.fastq.gz** will be renamed to
**M6Q_3_RBP1_R1.fastq.gz**

## 2. Make the sample sheet and deseq2 metadata.

Activate an environment containing snakemake, and then run the script `make_sample_sheet.py` script from the root of the directory.

```bash
$ python src/make_sample_sheet.py data/raw
```

This will make a samplesheet for the experiment called samplesheet.tsv in the root of the directory as well as the file `src/deseq2_metadata.csv`, the contents of the samplesheet will be structured like the following example:

```
sample	R1	R2	mark	condition	igg
HoxE_1_IgG	data/raw/HoxE_1_IgG_R1.fastq.gz	data/raw/HoxE_1_IgG_R2.fastq.gz	IgG	HoxE	HoxE_1_IgG
HoxE_1_Rbp1	data/raw/HoxE_1_Rbp1_R1.fastq.gz	data/raw/HoxE_1_Rbp1_R2.fastq.gz	Rbp1	HoxE	HoxE_1_Rbp1
HoxE_2_Rbp1	data/raw/HoxE_2_Rbp1_R1.fastq.gz	data/raw/HoxE_2_Rbp1_R2.fastq.gz	Rbp1	HoxE	HoxE_2_Rbp1
HoxE_3_Rbp1	data/raw/HoxE_3_Rbp1_R1.fastq.gz	data/raw/HoxE_3_Rbp1_R2.fastq.gz	Rbp1	HoxE	HoxE_3_Rbp1
HoxM_1_IgG	data/raw/HoxM_1_IgG_R1.fastq.gz	data/raw/HoxM_1_IgG_R2.fastq.gz	IgG	HoxM	HoxM_1_IgG
HoxM_1_Rbp1	data/raw/HoxM_1_Rbp1_R1.fastq.gz	data/raw/HoxM_1_Rbp1_R2.fastq.gz	Rbp1	HoxM	HoxM_1_Rbp1
HoxM_2_Rbp1	data/raw/HoxM_2_Rbp1_R1.fastq.gz	data/raw/HoxM_2_Rbp1_R2.fastq.gz	Rbp1	HoxM	HoxM_2_Rbp1
HoxM_3_Rbp1	data/raw/HoxM_3_Rbp1_R1.fastq.gz	data/raw/HoxM_3_Rbp1_R2.fastq.gz	Rbp1	HoxM	HoxM_3_Rbp1
HoxW_1_IgG	data/raw/HoxW_1_IgG_R1.fastq.gz	data/raw/HoxW_1_IgG_R2.fastq.gz	IgG	HoxW	HoxW_1_IgG
HoxW_1_Rbp1	data/raw/HoxW_1_Rbp1_R1.fastq.gz	data/raw/HoxW_1_Rbp1_R2.fastq.gz	Rbp1	HoxW	HoxW_1_Rbp1
HoxW_2_Rbp1	data/raw/HoxW_2_Rbp1_R1.fastq.gz	data/raw/HoxW_2_Rbp1_R2.fastq.gz	Rbp1	HoxW	HoxW_2_Rbp1
HoxW_3_Rbp1	data/raw/HoxW_3_Rbp1_R1.fastq.gz	data/raw/HoxW_3_Rbp1_R2.fastq.gz	Rbp1	HoxW	HoxW_3_Rbp1
```

The script splits the file name on the '_' and uses the first split for the condition, and the second split for the mark. The 'igg' column is the same as the 'sample' column and should be manually replaced with the sample name of the IGG or control you would like to use for that sample. If the sample is an IgG it can be the same as its name, and won't affect peak calling. 

So a fixed version of the table above would look like this:


```
sample	R1	R2	mark	condition	igg
HoxE_1_IgG	data/raw/HoxE_1_IgG_R1.fastq.gz	data/raw/HoxE_1_IgG_R2.fastq.gz	IgG	HoxE	HoxE_1_IgG
HoxE_1_Rbp1	data/raw/HoxE_1_Rbp1_R1.fastq.gz	data/raw/HoxE_1_Rbp1_R2.fastq.gz	Rbp1	HoxE	HoxE_1_IgG
HoxE_2_Rbp1	data/raw/HoxE_2_Rbp1_R1.fastq.gz	data/raw/HoxE_2_Rbp1_R2.fastq.gz	Rbp1	HoxE	HoxE_1_IgG
HoxE_3_Rbp1	data/raw/HoxE_3_Rbp1_R1.fastq.gz	data/raw/HoxE_3_Rbp1_R2.fastq.gz	Rbp1	HoxE	HoxE_1_IgG
HoxM_1_IgG	data/raw/HoxM_1_IgG_R1.fastq.gz	data/raw/HoxM_1_IgG_R2.fastq.gz	IgG	HoxM	HoxM_1_IgG
HoxM_1_Rbp1	data/raw/HoxM_1_Rbp1_R1.fastq.gz	data/raw/HoxM_1_Rbp1_R2.fastq.gz	Rbp1	HoxM	HoxM_1_IgG
HoxM_2_Rbp1	data/raw/HoxM_2_Rbp1_R1.fastq.gz	data/raw/HoxM_2_Rbp1_R2.fastq.gz	Rbp1	HoxM	HoxM_1_IgG
HoxM_3_Rbp1	data/raw/HoxM_3_Rbp1_R1.fastq.gz	data/raw/HoxM_3_Rbp1_R2.fastq.gz	Rbp1	HoxM	HoxM_1_IgG
HoxW_1_IgG	data/raw/HoxW_1_IgG_R1.fastq.gz	data/raw/HoxW_1_IgG_R2.fastq.gz	IgG	HoxW	HoxW_1_IgG
HoxW_1_Rbp1	data/raw/HoxW_1_Rbp1_R1.fastq.gz	data/raw/HoxW_1_Rbp1_R2.fastq.gz	Rbp1	HoxW	HoxW_1_IgG
HoxW_2_Rbp1	data/raw/HoxW_2_Rbp1_R1.fastq.gz	data/raw/HoxW_2_Rbp1_R2.fastq.gz	Rbp1	HoxW	HoxW_1_IgG
HoxW_3_Rbp1	data/raw/HoxW_3_Rbp1_R1.fastq.gz	data/raw/HoxW_3_Rbp1_R2.fastq.gz	Rbp1	HoxW	HoxW_1_IgG
```

For this example there was only one IgG per condition, so the sample name corresponding to that IGG was used for each sample in the condition. In the case that each sample had its own control file, each entry would correspond to the IgG for that sample. If only one IgG was used in the whole experiment, then its sample name could be used for each row. If you are not using IgG set config['USEIGG'] to false, and don't modify the samplesheet.


## 3. Edit configuration files 

1. Edit runtime configuration in the file `config.yml`:

   - Specify whether or not to trim adapters from raw reads to improve downstream alignment rate. You can decide based on adapter content in the sequencing core's FastQC reports.

   - Specify whether or not to use IGG for peak calling.

   - Specify the path to the bowtie2 index for the genome you are aligning to.

   - Specify the path to the fastq screen configuration file.

   - Specify the path to the genome FASTA file.

   - Specify the path to the chromosome sizes. 

2. The pipeline detects samples in the subdirectory data/raw with the following assumptions:

    - Paired end reads

    - Read 1 and 2 are designated using "_R1", and "_R2"

    - Samples are designated in the following convention: `{condition}_{replicate}_{mark}_{R1|R2}.fastq.gz`
      This format affects the output files and ensures the bigwig files from the same marks are merged.  

3. Make sure the `src/deseq2_metadata.csv` looks right. The file is created when you run `make_sample_sheet.py` and should have the following properties:

 - Should have two columns labeled "sample", and "condition"
 - The sample column corresponds to the individual biological replicates (includes all fields around the "_" delimiter). 
 - The condition should be the condition for each sample, which uses the first field with the "_" delimiter.
 - If you have multiple conditions and marks to analyze, you can introduce more columns into this file and adjust the deseq2.R file to account for extra covariates. 

 The file src/deseq2_metadata is populated with the following example data:

```
sample,condition
HoxE_1_IgG,HoxE
HoxE_1_Rbp1,HoxE
HoxE_2_Rbp1,HoxE
HoxE_3_Rbp1,HoxE
HoxM_1_IgG,HoxM
HoxM_1_Rbp1,HoxM
HoxM_2_Rbp1,HoxM
HoxM_3_Rbp1,HoxM
HoxW_1_IgG,HoxW
HoxW_1_Rbp1,HoxW
HoxW_2_Rbp1,HoxW
HoxW_3_Rbp1,HoxW
```

## 4. Set up SLURM integration (for batch jobs)

Do this step if are running the pipeline as a batch job and don't yet have a [SLURM profile](https://github.com/Snakemake-Profiles/slurm) set up.

Download the `slurm` folder from the maxsonBraunLab [repository](https://github.com/maxsonBraunLab/slurm) and copy the entire thing to `~/.config/snakemake`. 

Your file configuration for SLURM should be as follows:
```
~/.config/snakemake/slurm/<files>
```

Change the file permissions for the scripts in the `slurm` folder so that they are executable. To do this, run:
```
chmod +x ~/.config/snakemake/slurm/slurm*
```


# Execution

A "dry-run" can be accomplished to see what and how files will be generated by using the command:

```bash
snakemake -nrp
```

To invoke the pipeline, please use either of the two options below:

```bash
# run in interactive mode. Recommended for running light jobs.
snakemake -j <n cores> --use-conda --conda-prefix $CONDA_PREFIX_1/envs

# run in batch mode. Recommended for running intensive jobs.
sbatch run_pipeline_conda.sh
```

For users running the pipeline in batch mode, `run_pipeline_conda.sh` is a wrapper script that contains the following command:

```bash
snakemake -j <n jobs> --use-conda --conda-prefix $CONDA_PREFIX_1/envs --profile slurm --cluster-config cluster.yaml
```

Additional setup instructions are provided in the wrapper script.

You can standardize further arguments for running the pipeline in batch mode using the following [instructions](https://github.com/Snakemake-Profiles/slurm). The maxsonBraunLab repository [slurm](https://github.com/maxsonBraunLab/slurm) contains further instructions to set up a SnakeMake slurm profile.


# Reproducible results with SnakeMake + Singularity

To ensure the reproducibility of your results, we recommend running a SnakeMake workflow using Singularity containers. These containers standardize the underlying operating system of the workflow (e.g. Ubuntu, centOS, etc.), while conda tracks the installation of bioinformatic software (e.g. bowtie2, samtools, deseq2). To utilize Singularity in your analysis, log in to an interactive node and load the module first like this:

```bash
# request an interactive node
srun -p light --time=36:00:00 --pty bash

# re-activate your environment with snakemake
conda activate <snakemake-env>

# load the singularity program
module load /etc/modulefiles/singularity/current
```

By default, Singularity will create and use a cache directory in your personal user root folder (i.e. in `/home/users/<username>`). This may create problems as there is limited space in a user's root folder on Exacloud. To avoid issues with space in your root folder, you can set the Singularity cache directory path to a folder in your lab group directory like this:

```bash
# make a cache folder inside your lab user folder 
mkdir /home/groups/MaxsonLab/<your_user_folder>/singularity_cache

# make the path to the cache folder accessible to other processes
export SINGULARITY_CACHEDIR=/home/groups/MaxsonLab/<your_user_folder>/singularity_cache
```

If you are an experienced user, you can add the `export SINGULARITY_CACHEDIR=...` line to your `.bashrc` file. Otherwise, run the `export SINGULARITY_CACHEDIR=...` command before doing the steps below.

More Singularity documentation on Exacloud can be found [here](https://wiki.ohsu.edu/display/ACC/Exacloud%3A+Singularity). If it is your first time running the pipeline, and especially when using Singularity, we must install all the conda environments using the following command:

```bash
indices_folder="/home/groups/MaxsonLab/indices"
conda_folder="${CONDA_PREFIX_1}/envs"
fastq_folder="/home/groups/MaxsonLab/input-data2/path/to/FASTQ/folder/"

snakemake -j 1 \
	--verbose \
	--use-conda \
	--conda-prefix $conda_folder \
	--use-singularity \
	--singularity-args "--bind $indices_folder,$conda_folder,$fastq_folder" \
	--conda-create-envs-only

```
The above code snippet will take about an hour or more to set up, but is a one-time installation. After creating the conda environments, symlinks, `samplesheet.tsv`, and the `src/deseq2_metadata.csv`, we can invoke the pipeline in the same shell like this:

```bash
# Singularity + interactive run
snakemake -j <n cores> \
	--use-conda \
	--conda-prefix $conda_folder \
	--use-singularity \
	--singularity-args "--bind $indices_folder,$conda_folder,$fastq_folder"

# Singularity + slurm (batch) run
sbatch run_pipeline_singularity.sh
```

For users running the Singularity version of the pipeline in batch mode, `run_pipeline_singularity.sh` is a wrapper script for the pipeline. You will need to add the appropriate FASTQ folder path to the script prior to running. Additional instructions are provided in the wrapper script.

NOTE: make sure to use double quotes, and insert an integer for the -j flag. 

The above command will install the pipeline's conda environments into the `conda-prefix` directory - this means that conda environments are actually not stored INSIDE the container. The `--bind` argument binds the host (Exacloud) paths to the container to access the genome indices, conda prefix, and the path to the raw sequencing files. The `--profile slurm` in the wrapper script will configure default settings for SnakeMake to interact with SLURM - more information can be found [here](https://github.com/maxsonBraunLab/slurm). Feel free to create another [snakemake profile](https://wiki.ohsu.edu/display/ACC/Exacloud%3A+Singularity) that has its own set of singularity arguments for added convenience.

# Method

Without adapter trimming:

![](rulegraph.svg)


With adapter trimming enabled:

![](rulegraph2.svg)


# Output

Below is an explanation of each output directory:

```
aligned - sorted and aligned sample bam files
callpeaks - the output of callpeaks.py with peak files and normalized bigwigs for each sample.
counts - the raw counts for each mark over a consensus peak list
deseq2 - the results of deseq2 fore each counts table (by mark)
dtools - fingerprint plot data for multiqc to use
fastp - adapter-trimmed FASTQ files (if adapter-trimming option is enabled in `config.yml`)
fastqc - fastqc results
fastq_screen - fastq_screen results
logs - runtime logs for each snakemake rule
markd - duplicate marked bam files
multiqc - contains the file multiqc_report.html with a lot of QC info about the experiment.
plotEnrichment - FRIP statistics for each mark
preseq - library complexity data for multiqc report
```

## Deseq2 outputs

Each mark should have the following output files:

```
"data/deseq2/{mark}/{mark}-rld-pca.png" - PCA of counts after reguarlized log transformation. 
"data/deseq2/{mark}/{mark}-vsd-pca.png" - PCA of counts after variance stabilizing transformation.
"data/deseq2/{mark}/{mark}-normcounts.csv" - normalized count for each sample in each consensus peak.
"data/deseq2/{mark}/{mark}-lognormcounts.csv" - log2 normalized counts for each sample in each consensus peak.
"data/deseq2/{mark}/{mark}-rld.png", - the sdVsMean plot using regularized log transformation.
"data/deseq2/{mark}/{mark}-vsd.png" - the sdVsMean plot using variance stabilizing transformation.
"data/deseq2/{mark}/{mark}-vsd-dist.png" - the sample distance matrix after variance stabilizing transformation.
"data/deseq2/{mark}/{mark}-rld-dist.png" - the sample distance matrix using regularized log transformation.
"data/deseq2/{mark}/{mark}-dds.rds" - the R object with the results of running the DEseq() function.
```

For each contrast, the differentially expressed genes are written to a file ending in `-diffexp.tsv` as well as those with an adjusted p-value less than 0.05 with the extension `-sig05-diffexp.tsv`. A summary of the results using an alpha of 0.05 is also written to a file with the extension `-sig05-diffexp-summary.txt`. Additionally two MA plots are written to the file ending in `plotMA.png` that have highlighted differential peaks with an adjusted p-value less than 0.1.

See the following paper for further explanations of the above plots and data transforms:
https://bioconductor.org/packages/release/bioc/vignettes/DESeq2/inst/doc/DESeq2.html#exporting-results-to-csv-files 
