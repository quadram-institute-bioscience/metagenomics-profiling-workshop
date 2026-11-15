---
title: "Guided hands-on: MAG quality and taxonomy"
---

# Guided hands-on: MAG quality and taxonomy

---

## Overview

The six coffee fermentation samples were each assembled separately, and the contigs of each assembly were grouped into bins with SemiBin2. This gave **249 bins**. But are they good genomes?

In this session, you will:
- Check what kind of sequences each bin contains with **Tiara**
- Estimate the completeness and contamination of bins with **CheckM2**
- Explore the precomputed results for all 249 bins in R: quality, taxonomy (**GTDB-Tk**) and redundancy (**dRep**)

You will run Tiara and CheckM2 on three bins, then explore the results for all bins in an R Markdown document: **you don't need to know R**.

---

## Assessing MAG quality

### Completeness and contamination

**Completeness** estimates how much of the genome is present. **Contamination** estimates how much DNA from other genomes was binned with it. Both are estimated from genes expected in every genome: missing genes lower the completeness, duplicated genes raise the contamination.

These are **estimates**: novel organisms may lack expected genes, and true gene duplications can look like contamination.

### MIMAG quality standards

The community standards for MAG quality (Bowers et al. 2017):

| Quality tier | Completeness | Contamination | Additional requirements |
|---|---|---|---|
| **High** | >90% | <5% | 23S, 16S, 5S rRNA genes and at least 18 tRNAs |
| **Medium** | ≥50% | <10% | |
| **Low** | <50% | <10% | |

CheckM2 does not check the rRNA and tRNA genes: in this session, "High" only uses completeness and contamination.

### The tools

- **CheckM2** uses machine learning to estimate completeness and contamination. It was trained on **bacterial and archaeal** genomes.
- **Tiara** classifies each contig of a bin as bacteria, archaea, eukarya, organelle (mitochondria, plastids) or unknown. The coffee fermentation also contains yeasts and plant material: Tiara tells us which bins are not bacterial, and for which CheckM2 results should not be trusted.

---

## Setup

```bash
export WSUSER=/shared/team/users/{your_name}/
DIR=/shared/team/2026_training/week5/day4_MAGs/Guided_tutorial
CHECKM2DB=/shared/team/2026_training/week5/day4_MAGs/databases/checkm2/CheckM2_database/uniref100.KO.1.dmnd

mkdir -p $WSUSER/MAGs
cd $WSUSER/MAGs

ls $DIR
```

We will look at three bins:

| Bin file | Sample |
|---|---|
| `ERR2231567_SemiBin_9.fa` | T0 |
| `ERR2231569_SemiBin_29.fa` | T16 |
| `ERR2231569_SemiBin_12.fa` | T16 |

---

## Part 1: Run Tiara

```bash
conda activate /shared/team/conda/aliseponsero.mmb-dtp/tiara

tiara -i $DIR/ERR2231567_SemiBin_9.fa -o tiara_bin9.txt --threads 2
```

- `-i`: the bin (a FASTA file of contigs)
- `-o`: the output file
- `--threads`: number of CPUs to use

The output has one line per contig, with its classification. Look at it, and count the contigs in each class:

```bash
head tiara_bin9.txt

cut -f2 tiara_bin9.txt | sort | uniq -c
```

**Your turn:** run Tiara on the two other bins (`ERR2231569_SemiBin_29.fa` and `ERR2231569_SemiBin_12.fa`) and count the classes.

> **Question:** which bin is not bacterial?

---

## Part 2: Run CheckM2

```bash
conda activate /shared/team/conda/aliseponsero.mmb-dtp/checkm2

checkm2 predict --threads 4 \
    --input $DIR/ERR2231567_SemiBin_9.fa $DIR/ERR2231569_SemiBin_29.fa $DIR/ERR2231569_SemiBin_12.fa \
    --output-directory checkm2_out \
    --database_path $CHECKM2DB
```

- `--input`: the bins to assess
- `--output-directory`: where to write the results
- `--database_path`: the database CheckM2 uses to annotate the genes

It takes 1 to 2 minutes. You may see a warning about scikit-learn versions: it can be ignored here.

```bash
column -t checkm2_out/quality_report.tsv | less -S
```

Fill in the table:

| Bin | Tiara: main class | CheckM2 completeness (%) | CheckM2 contamination (%) | MIMAG tier |
|---|---|---|---|---|
| ERR2231567_SemiBin_9 | | | | |
| ERR2231569_SemiBin_29 | | | | |
| ERR2231569_SemiBin_12 | | | | |

> **Questions:**
> - Which bin is a good MAG?
> - Bin 29 is 100% bacterial. Is it a good MAG?
> - What does CheckM2 say about the non-bacterial bin? Should we trust it?

---

## Part 3: Explore all the bins in R

The same tools (plus GTDB-Tk and dRep) were run on all 249 bins. Copy the R Markdown document:

```bash
cp $DIR/Session2_MAGs_guided.Rmd $WSUSER/MAGs/
```

Open RStudio, open `Session2_MAGs_guided.Rmd`, and run the chunks **in order** (green arrow at the top right of each chunk).

### 1. Tiara: what kind of sequences are in each bin?

> - Most bins are bacterial. What else did the binning tool group into bins?
> - Which bin classes could come from yeasts and plant material?

### 2. CheckM2: how complete and contaminated are the bins?

> - How many bins are high or medium quality? Why are most bins low quality?
> - What do all the "contaminated" bins have in common? Look at their Tiara class and genome size.
> - Can we trust CheckM2 for these bins?

### 3. GTDB-Tk: what are these MAGs?

**GTDB-Tk** finds marker genes of bacteria and archaea in each bin, and places the bin in the reference tree of the GTDB taxonomy. GTDB names can differ from NCBI names (for example `Leuconostoc_B`), and new species get placeholder names (for example `Tatumella sp018257575`).

> - Do better quality bins get a more precise classification?
> - Are the main species of the taxonomic profiles recovered as MAGs? Is there a MAG of *Paucilactobacillus vaccinostercus*, the species MetaPhlAn4 and Kraken2 disagreed on?
> - What happens when GTDB-Tk is given a eukaryotic bin?

### 4. dRep: how many different genomes did we recover?

The same organism can be recovered from several samples. **dRep** compares the bins by average nucleotide identity (ANI), groups the bins that are very similar, and keeps the best bin of each group as the representative.

An ANI of 95% is commonly used as the species boundary. Here, dRep used a threshold of 99%: it separates bins that are very close (close to strain level).

> - Is each species recovered from several timepoints?
> - Are all the bins of a species in the same cluster? What could it mean?
> - How many good quality, non-redundant MAGs did we recover?

---

## Resources

- **CheckM2:** https://github.com/chklovski/CheckM2
- **Tiara:** https://github.com/ibe-uw/tiara
- **GTDB-Tk:** https://github.com/Ecogenomics/GTDBTk
- **dRep:** https://github.com/MrOlm/drep
- **MIMAG standards:** Bowers et al. (2017) Nature Biotechnology
