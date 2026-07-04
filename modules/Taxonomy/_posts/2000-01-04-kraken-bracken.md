---
title: "Kraken2 & Bracken setup"
---
# Kraken2 & Bracken: K-mer Based Taxonomic Classification

---

> The Workshop materials are available to download in
> [Figshare](https://doi.org/10.6084/m9.figshare.32900546)

## Theory Overview

### What is Kraken2?

**Kraken2** is an ultrafast taxonomic classification system that uses exact k-mer matching to assign taxonomic labels to metagenomic DNA sequences.

**Key Features:**
- **K-mer based** classification (exact matching of short DNA sequences)
- **Extremely fast** - can process millions of reads per minute
- **High sensitivity** - detects more taxa than marker-based methods
- Reports **read counts** at all taxonomic levels
- Uses **LCA (Lowest Common Ancestor)** algorithm for ambiguous k-mers

### How Does Kraken2 Work?

1. **Database Construction**
   - K-mers (default: 35bp) extracted from reference genomes
   - Each k-mer mapped to the lowest taxonomic level where it appears
   - Database contains bacteria, archaea, viruses, plasmids, and optionally fungi, protozoa, plants

2. **Classification Step**
   - Each read broken into k-mers
   - K-mers queried against database
   - LCA algorithm determines the most specific taxonomic assignment
   - Confidence score calculated based on k-mer agreement

3. **Output**
   - Per-read classifications (which taxon each read belongs to)
   - Summary report with read counts at each taxonomic level

### Kraken2 Modes

**Standard mode:** Examines all k-mers in a read  
**Quick mode (`--quick`):** Stops after first k-mer hit - much faster, slightly less accurate

### Advantages

✅ **Very fast** - Processes samples in minutes  
✅ **High sensitivity** - Detects more taxa, including rare ones  
✅ **Low unclassified rate** - Classifies more reads than MetaPhlAn4  
✅ **Comprehensive** - Reports all taxonomic levels  
✅ **Flexible databases** - Many pre-built options available  

### Limitations

⚠️ **Higher false positive rate** - Especially for low-abundance taxa  
⚠️ **Reports read counts** - Not normalized for genome size  
⚠️ **High memory usage** - Full databases require a good amount RAM (to run on the HPC)
⚠️ **Many reads at higher levels** - Reads classified to genus/family rather than species  

---

## What is Bracken?

### The Problem Kraken2 Doesn't Solve

Kraken2 is very good at **classifying reads**, but has a problem:
- Many reads are classified at **higher taxonomic levels** (genus, family, order)
- This happens when k-mers are shared across multiple species
- We want **species-level abundances** for most analyses

### Bracken's Solution

**Bracken** (Bayesian Reestimation of Abundance with KrakEN) uses a probabilistic approach to:
- **Redistribute** reads from higher levels down to species
- Use expected k-mer distributions to estimate the true species of origin
- Produce more accurate **species-level abundance estimates**

### How Bracken Works

1. **Pre-computation** (already done for workshop databases)
   - For each species in database, simulate reads of length X (e.g., 150bp)
   - Classify simulated reads with Kraken2
   - Calculate: "What % of this species' reads get classified to genus vs species?"

2. **Re-estimation** (what you run)
   - Takes Kraken2 report as input
   - Uses Bayesian inference to redistribute higher-level reads
   - Filters out low-confidence species (below threshold)
   - Outputs species-level abundance estimates

### Bracken Advantages

✅ **Better species-level estimates** than raw Kraken2  
✅ **Reduces false positives** through thresholding  
✅ **Fast** - runs in seconds  
✅ **Conservative** - filters out poorly supported assignments  

### Important Note

⚠️ Bracken still reports **READ COUNTS**, not genome-size-normalized abundances  
⚠️ For relative abundances, you need to convert: `(reads / total_reads) × 100`

---

## When to Use Kraken2 + Bracken?

- When you need **comprehensive detection** of all taxa
- For **novel or diverse environments** where sensitivity matters
- When you want to **minimize unclassified reads**
- For **rapid screening** of many samples
- When combined with Bracken, provides **balanced specificity and sensitivity**

---

## Hands-On: Running Kraken2 and Bracken

### Prerequisites

Make sure you have:
- Access to the workshop notebook
- Data symlinked/copied to your workspace
- Kraken2 conda environment available
- RStudio access on the notebook


```bash
conda deactivate

conda activate /shared/team/conda/aliseponsero.cli-giba-2026/kraken2/
```


