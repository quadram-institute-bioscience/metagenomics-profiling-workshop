# Kraken2 small exercise

> Try using Kraken2 (and maybe metaphlan or other tools as well) on very
> simple datasets.


This small exercise allows to tinker with Kraken2 in a simplified dataset.

We'll download a reference genome, simulate sequencing reads, and classify them to verify the classification accuracy.

## Overview

We'll use *Bilophila* sp. as our test organism, simulating paired-end sequencing reads and classifying them with Kraken2.

## Prerequisites

- `datasets` (NCBI Datasets command-line tools)
- `seqfu` (sequence manipulation toolkit)
- `kraken2` (taxonomic classifier)
- Access to a Kraken2 database (e.g., PlusPF)

You can create a new environment with the required tools:

```bash
mamba create -n kraken2-demo -c conda-forge -c bioconda \
    ncbi-datasets-cli "seqfu>=1.20" kraken2>=2.0.8 krakentools
```

## Step 1: Download Reference Genome

Download the *Bilophila* sp. 4_1_30 genome from NCBI using the `datasets` tool:

```bash
# Download genome assembly using accession number
datasets download genome accession GCF_000224655.1

# Extract the downloaded archive
unzip ncbi_dataset.zip
```

:bulb: use `ls` and `find` to explore the extracted files. Identify the path to the FASTA file and use `seqfu stats -nt` to check the genome size and number of contigs.

## Step 2: Chop your genome in fragments

Use `seqfu shred` to fragment the genome into simulated paired-end reads:

```bash
# Create output directory
mkdir kraken-test

# Generate synthetic paired-end reads
fu-shred -l 100 -s 3 \
  -o kraken-test/bwa \
  ncbi_dataset/data/GCF_000224655.1/GCF_000224655.1_Bilophila_sp_4_1_30_V1_genomic.fna
```

**Parameters:**
- `-l 100`: Read length of 100 bp
- `-s 3`: Step size of 3 bp (one new fragment every 3 bases)
- `-o kraken-test/bwa`: Output prefix for generated files

**Output:** Creates `bwa_R1.fq` and `bwa_R2.fq` (paired-end FASTQ files)

:bulb: use `gzip` to compress the FASTQ files to save space.

## Step 3: Classify Reads with Kraken2

Run Kraken2 to taxonomically classify the simulated reads:

```bash
# Navigate to output directory
cd kraken-test

# Run Kraken2 classification
kraken2 --db /shared/public/db/kraken2/pluspf_16gb/ \
   --memory-mapping \
   --threads 8 \
   --report bwa.tsv \
   --paired \
   bwa_R1.fq bwa_R2.fq > bwa.raw
```

**Parameters:**

- `--db`: Path to Kraken2 database (PlusPF 16GB version)
- `--memory-mapping`: Load database into memory for faster processing
- `--threads 8`: Use 8 CPU threads
- `--report bwa.tsv`: Generate human-readable classification report
- `--paired`: Process files as paired-end reads
- `> bwa.raw`: Redirect per-read classifications to file

**Outputs:**

- `bwa.raw`: Per-read classification results
- `bwa.tsv`: Summary report with taxonomic distribution

## Exercise

Explore tha "raw" output: this is the *per-read* classification made by Kraken2. 

:bulb: use `less -S` or `vd` to have a first look. Then you can use `cut`, `sort`, `uniq -c` on specific columns.

The report file (`bwa.tsv`) contains the summary of classifications at different taxonomic levels. It aggregates the results from the raw output.

:bulb: Try using Metaphlan, can you check if there is a higher specificity (lower false positive rate) compared to Kraken2?

## Output files

* [Kraken2 report (default parameters)](https://gist.github.com/telatin/068581b1d063b227cbd801ab993ba808)
  * [Kraken2 raw output (first lines)](https://gist.github.com/telatin/d04e83580992491afe1193f44c2627f1)
* [Kraken2 report (confidence=0.1)](https://gist.github.com/telatin/91ee68d4750c39fa55e52ab6437d2ff3)
* [Metaphlan report](https://gist.github.com/telatin/5d943d3be02cf83f3c0cca3a640c5bc0)