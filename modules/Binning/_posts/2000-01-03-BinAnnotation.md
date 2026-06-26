---
title: "Bin Annotation and Dereplication"
---
# Part 2: Bin Annotation and Dereplication

---

## Overview

In this section, you'll learn to:
- **Dereplicate bins** to remove redundancy and select best representatives
- **Assign taxonomy** using GTDB-Tk phylogenetic placement
- **Understand classification confidence** and its relationship to MAG quality
- **Compare assembly strategies** for MAG recovery

**The Workshop material** is available to download in [Figshare](https://figshare.com/articles/presentation/MMB_DTP_workshop_-_Introduction_to_binning_of_metagenomes/30724289)

---

## dRep: Dereplication and Representative Selection

### Why Dereplicate?

When using multiple assemblies (individual samples + co-assembly), the same organism is often recovered multiple times. Different samples, same species. Different assembly strategies, same organism. Different binning parameters, same genome.

Without dereplication: wasteful redundancy, inflated diversity estimates, unclear which bin to use for analysis.

With dereplication: one high-quality representative per species, clean non-redundant MAG set, ready for comparative genomics.

### How dRep Works

**Step 1: Quality Assessment**
- Runs CheckM on all bins (we already have this!)
- Calculates quality scores for representative selection

**Step 2: Primary Clustering (Mash - fast)**
- Uses k-mer sketches for approximate comparison
- Groups genomes at ~90% ANI threshold
- Avoids expensive all-vs-all ANI calculations

**Step 3: Secondary Clustering (FastANI - precise)**
- Accurate ANI within primary clusters
- Groups at 95% ANI (species level)
- Creates final species-level clusters

**Step 4: Representative Selection**
- Chooses best genome from each cluster
- Score formula: `completeness - 5Ã—contamination + 0.5Ã—log(N50)`
- Higher completeness good, contamination heavily penalized, better assembly quality as tiebreaker

### Understanding ANI Thresholds

**Average Nucleotide Identity (ANI)** measures genome-wide sequence similarity:
- **>95% ANI:** Same species
- **>99% ANI:** Very close strains
- **<95% ANI:** Different species

The 95% threshold roughly equals the traditional 70% DNA-DNA hybridization species definition.

---
## GTDB-Tk: Taxonomy Assignment

### What is GTDB-Tk?

GTDB-Tk assigns taxonomy to MAGs through phylogenetic placement in the GTDB reference tree. It uses 120 bacterial or 53 archaeal conserved marker genes.

### The Process

1. **Identify markers** - Finds expected marker genes in your MAG
2. **Align markers** - Aligns to reference sequences
3. **Build tree** - Places MAG in reference phylogenetic tree
4. **Assign name** - Names based on tree position and closest references

### Classification Confidence

**ANI to closest reference:**
- **>95% ANI:** High confidence, likely same species
- **85-95% ANI:** Medium confidence, genus-level accurate
- **<85% ANI or N/A:** Low confidence, potentially novel organism

**Completeness matters!**
- High completeness → more markers found → confident placement
- Low completeness → few markers → uncertain placement

### GTDB Taxonomy Format

GTDB uses standardized rank prefixes:
```
d__Bacteria;p__Bacillota;c__Bacilli;o__Lactobacillales;f__Lactobacillaceae;g__Lactiplantibacillus;s__Lactiplantibacillus_plantarum
```

- `d__` = Domain
- `p__` = Phylum  
- `c__` = Class
- `o__` = Order
- `f__` = Family
- `g__` = Genus
- `s__` = Species

---

## Exploring dRep and GTDB-TK Results in R

### Open RStudio

**[Link to R Markdown: dRep and GTDB-Tk Exploration](https://figshare.com/articles/presentation/MMB_DTP_workshop_-_Introduction_to_binning_of_metagenomes/30724289)**

In this R session, you'll:
1. Load dRep clustering results
2. Explore how many species-level clusters were found
3. Examine which bins grouped together
4. Identify the final representative MAGs
5. Load GTDB-Tk taxonomy assignments
6. Merge all results (Tiara, CheckM2, dRep, GTDB-Tk)
7. Investigate the relationship between quality and taxonomy confidence

### Key Questions to Explore

**Q1:** How much redundancy did dRep remove?
- Input bins vs final representatives

**Q2:** Which clusters have multiple bins?
- Could indicate real strain variation across timepoints
- Or technical redundancy from multiple assemblies

**Q3:** How many representatives have taxonomy assigned?
- Should be most of the high quality bins (if they're bacterial)
- Eukaryotic bins won't get GTDB-Tk classification

---

## Resources

– **dRep:** https://drep.readthedocs.io/  
– **GTDB-Tk:** https://ecogenomics.github.io/GTDBTk/  
– **GTDB:** https://gtdb.ecogenomic.org/  
- **ANI Species Definition:** Jain et al. (2018) Nature Communications  

---
