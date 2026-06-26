---
title: About this course
---

![Introduction]({{ site.baseurl }}{% link img/workflow_small.png %})

This workshops introduces some core concepts and basic skills in the
analysis of **whole metagenome shotgun** experiments.

The goal is to present:

1. What information we can extract from the gDNA sequencing of a microbial community (whole metagenome shotgun)
2. The difference between read based approaches (profiling) and *de novo* assembly (MAGs)
3. How to evaluate the quality of the sequencing and how to remove unwanted reads (QC, host removal,...)


### What we will learn to do 

In terms of tools we will:

1. Remove host reads using **Hostile** (and other methods)
2. Perform a taxonomic profiling (who is there?) of the prokaryome using **MetaPhlAn 4** and the overall profiling using **Kraken2**, comparing the advantages and disadvantages of each tool
3. Perform a functional profiling (what are they doing?) using **HUMAnN**
4. Assemble the reads with **MegaHit**, to produce a set of contigs. We will see what a co-assembly is and when it's useful to do it
5. See how to perform the *backmapping* (sometimes called read recruitment), and what information we can extract from the *coverage tracks*, using **CoverM**
6. Try to group contigs belonging to the same genome (*Binning*), using **SemiBin2**
7. Evaluate the completeness and contamination of bins (**CheckM2**, **BUSCO**), and see how to dereplicate them with **dRep**, and how to assign a taxonomy using **GTDBtk**
8. Finally, we will see how to perform manual binning and explore metagenomics assemblies using **Anvi'o**

