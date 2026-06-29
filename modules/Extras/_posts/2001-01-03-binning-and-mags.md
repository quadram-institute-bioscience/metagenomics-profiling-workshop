---
title: Binning and MAGs
---
> Most bacteria and archaea in a sample have never been cultured, but their genomes are present in the
> shotgun sequencing data. A **Metagenome-Assembled Genome (MAG)** is a genome reconstructed directly
> from a metagenome, allowing us to retrieve, characterise, and classify the genomes of these otherwise
> unknown organisms.

### The workflow at a glance

Going from raw reads to a usable set of genomes follows a consistent path:

1. **Assembly**: reads are assembled into contigs.
2. **Binning**: contigs are grouped into draft genomes (bins).
3. **Quality assessment**: each bin is scored for completeness and contamination.
4. **Dereplication**: redundant genomes are collapsed into a non-redundant catalogue.
5. **Classification**: each genome is placed taxonomically.

### Binning

Binning is the step that groups assembled contigs into genome bins, ideally one bin per organism. It relies on
signals that are consistent within a genome but differ between genomes, mainly sequence composition
(tetranucleotide frequency, TNF) and coverage across samples.

Older tools such as MetaBAT cluster contigs using TNF and/or coverage. Newer methods such as VAMB and SemiBin2
use deep learning to cluster contigs into bins, often improving recovery of related or low-abundance genomes.

### Assessing MAG quality

A draft genome is only useful if we know how complete and how contaminated it is. Both are estimated using
**single-copy marker genes**: genes expected exactly once per genome, so their presence indicates completeness
and their duplication indicates contamination. Newer tools such as CheckM2 use machine-learning methods to
estimate these metrics (https://doi.org/10.1038/s41592-023-01940-w).

These estimates are then binned into quality tiers:

| Threshold      | Completeness | Contamination |
|----------------|--------------|---------------|
| High quality   | >90%         | <5%           |
| Medium quality | >50%         | <10%          |
| Low quality    | <50%         | <10%          |

### Building a MAG catalogue (dereplication)

When the same organism is recovered from several samples, the same genome appears many times. Dereplication
collapses these into a single representative per genome, producing a non-redundant catalogue.

A common tool is **dRep**, which first compares all genomes with the fast Mash algorithm, then runs a slower,
more accurate secondary comparison (ANIm or gANI) only on genome pairs sharing at least 90% Mash ANI. This
greatly reduces the number of slow comparisons while keeping sensitivity high
(https://drep.readthedocs.io/en/latest/overview.html).

### Classifying MAGs

Classifying genomes from unknown taxa, especially when they are fragmented and incomplete, is handled by tools
such as **GTDB-Tk**, which uses standardised marker gene sets (120 bacterial markers, bac120; 53 archaeal
markers, arc53). These markers are single-copy, near-universal genes that are rarely horizontally transferred,
which makes them reliable even for incomplete genomes. Classification proceeds in three steps:

1. **Marker gene identification**: predict genes in the MAG and search for the bac120 / arc53 markers.
2. **Alignment to GTDB references**: align the identified markers to the GTDB reference alignment and
   concatenate them per genome.
3. **Phylogenetic placement and classification**: insert the genome into the GTDB reference tree and assign
   taxonomy using ANI and relative evolutionary divergence (RED). A genome is assigned to a species at an ANI
   of at least 95% to a reference.

### Caveats and limitations

MAGs are powerful but imperfect, and it helps to keep a few caveats in mind:

* **Binning is imperfect**: no single algorithm is best, and different tools produce different bins. More robust
  pipelines run several binning tools and add a refinement step (https://doi.org/10.1038/s41467-025-57957-6).
* **MAGs are population genomes**: a MAG is a consensus over many cells, not a single isolate. The assembly
  strategy shapes the result. Single-sample assembly preserves strain-level variation and reduces chimeras but
  needs higher per-sample depth and yields more MAGs to dereplicate; co-assembly (pooling samples) boosts
  coverage for low-abundance organisms and gives more complete genomes, but collapses strain variation and is
  more computationally demanding.
