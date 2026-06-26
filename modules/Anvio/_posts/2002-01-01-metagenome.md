---
title: A metagenomics workflow with Anvi'o
---

## Overview

If we start from the raw reads:

1. Obtain the *contigs FASTA file* (with appropriate sequence names!)
2. Map the raw reads from the available samples against the contigs to obtain *a sorted BAM file* for each sample

In your Anvi'o environment:

1. Import the contigs FASTA file into an Anvi'o contigs database `CONTIGS.db`
2. Generate a profile for each sample BAM file, using the contigs database
3. Merge the profiles into a single profile database

---

## Prepare the contigs FASTA file

:bulb: Having lots of contigs can be a problem and result in unmanageable Anvi'o sessions. A good idea is to filter out short contigs (by length) and Eukaryotic contigs (using Tiara).

Anvi'o requires specific formatting for contig names. 
You can use SeqFu to rename the contigs:

```bash
# Reformat contig names and enforce minimum length (default: 1000 bp)
seqfu cat --anvio contigs.fa > renamed-contigs.fa
```

:bulb: the command will generate a two columns report mapping original names to the new ones. You can redirect it to a file with `rename_report.txt`.



## Create the contigs database

Initialize the Anvi'o contigs database from your assembled contigs:

```bash
# remember to activate your anvi'o environment
conda activate anvio-dev

# Create the contigs database
anvi-gen-contigs-database -f contigs-fixed.fa \
    -o CONTIGS.db \
    -n "My metagenome project"
```

#### Run HMM profiles for gene calling

Search for single-copy core genes to assess completeness:

```bash
# Run bacterial single-copy genes
anvi-run-hmms -c CONTIGS.db \
    --num-threads 4
```

### Annotate taxonomy (optional)

```bash
# Run COG annotations
anvi-run-ncbi-cogs -c CONTIGS.db \
    --num-threads 4

# Run taxonomic classification
anvi-run-scg-taxonomy -c CONTIGS.db \
    --num-threads 4

# [optional] run tRNAscan-SE for tRNA genes
anvi-run-trnascan -c CONTIGS.db \
    --num-threads 4
```

---

## Create and merge the coverage profiles

### Generate individual sample profiles

For each BAM file, create a profile that quantifies coverage:

```bash
# Profile sample 1
anvi-profile -i sample1.bam \
    -c CONTIGS.db \
    -o sample1_profile \
    --num-threads 4

# Profile sample 2
anvi-profile -i sample2.bam \
    -c CONTIGS.db \
    -o sample2_profile \
    --num-threads 4

# Repeat for additional samples...
# or use a "for" loop to automate
```

### Merge profiles

Combine all sample profiles into a single merged database:

```bash
# Merge all profiles
anvi-merge sample*_profile/PROFILE.db \
    -o MERGED_PROFILE \
    -c CONTIGS.db

# Note you might need --enforce-hierarchical-clustering
# if you have many contigs, but if there are too many contigs
# you might want to filter them first.
```

---

## Running Anvi'o

Launch the interactive interface to explore your data:

```bash
# Start the interactive interface
anvi-interactive -p MERGED_PROFILE/PROFILE.db \
    -c CONTIGS.db
```

This opens a web browser where you can:

- Visualize contig coverage across samples
- Manually refine bins
- Assess genome completeness
- Export results for downstream analysis

### Export binning results

```bash
# Summary of bins
anvi-summarize -p MERGED_PROFILE/PROFILE.db \
    -c CONTIGS.db \
    -C default \
    -o SUMMARY
```

## ☕️ Our coffee data

If you want to download a co-assembly of the "Early" samples:

* [Download coffee metagenome data](https://www.dropbox.com/scl/fo/ivkohz4a8hq1q7eyznybr/AIwK-KcBmNRUg-iPOBhwFJA?rlkey=qbmaiy0i0g4eizrg8nndgrm93&dl=0)