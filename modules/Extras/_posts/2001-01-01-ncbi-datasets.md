---
title: NCBI datasets
---

> "datasets" is a command-line tool to programmatically download genomes and annotations
> from NCBI.

[dataset schematics](https://www.ncbi.nlm.nih.gov/datasets/docs/v2/datasets_schema_taxonomy.svg)

## What is NCBI "datasets"?
 
[NCBI "datasets"](https://www.ncbi.nlm.nih.gov/datasets/docs/v2/) 
is a handy tool to quickly grab genome sequences and annotations from the NCBI database, 
straight to your laptop or HPC system.

It’s great if you want a fully packaged download—FASTA genome files,
GFF annotation, protein FASTA file etc.


### Get the `datasets` command

First, make sure you have the command-line tool installed.
If you’re using conda, you can install datasets like this:

```bash
conda create -n ncbi-datasets -c conda-forge ncbi-datasets-cli
conda activate ncbi-datasets
```

:bulb: You can replace conda with *mamba*, or *micromamba*; depends on your setup

You can also use pre-built binaries, or just run `datasets` via Docker, 
see [datasets install instructions](https://www.ncbi.nlm.nih.gov/datasets/docs/v2/download-and-install/) for other options.



### Downloading a Genome and Annotation

We’ll use _Lactococcus lactis_ as our example (a popular lactic acid bacterium).

#### Step 1: Search with the Web Portal (Optional)

If you don’t know the exact taxonomic name,
you can search using NCBI's web portal: https://www.ncbi.nlm.nih.gov/datasets/genomes

#### Step 2: Download with datasets

To download the latest reference genome and annotation for *Lactococcus lactis*, run:

```bash
datasets download genome taxon "Lactococcus lactis" \
    --reference 
```

- `genome taxon "Lactococcus lactis"`: Specify the organism (replace as you need)
- `--reference`: Download the reference genome (not just any assembly)

This makes a ZIP file: `ncbi_dataset.zip`

#### Step 3: Unpack the Download

Unzip the downloaded file to access all contents:

```bash
unzip  ncbi_dataset.zip  -d lactis/
```

:bulb: use `find` to check what was extracted


 
#### Step 4: Download Multiple Genomes (examples)

Just change the taxon name.
For a *list* of related genomes, you can specify higher-level taxa, e.g.:

```bash
datasets download genome taxon "Bifidobacterium" 
```

:warning: this will download 11k genomes!

Or use NCBI Assembly Accession numbers like
[GCF_003176835.1](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_003176835.1/):

```bash
datasets download genome accession GCF_003176835.1
```

to also download the annotation:

```bash
datasets download genome accession GCF_003176835.1 --include gff3,rna,cds,protein,genome,seq-report
```

For viral, fungal, or archaeal genomes—just swap in the taxon name.



## Inspecting the Data

All files are unpacked in your chosen directory.
You can check their formats using standard tools,
*e.g.* `less genome.gff` or `head genome.fna`.

For the *Lactococcus lactis* experiment you can try:

```bash
seqfu stats -bnt lactis/ncbi_dataset/data/GCF_003176835.1/GCF_003176835.1_ASM317683v1_genomic.fna 
```

## Example Bash Loop (Optional)

For multiple taxa in a bash script:

```bash
for taxon in "Lactococcus lactis" "Bifidobacterium animalis" "Streptococcus thermophilus"; do
    datasets download genome taxon "$taxon" --reference --dehydrated
    unzip ncbi-datasets-genomes-taxonomy-*.zip -d "${taxon// /_}/"
done
```

 
 


