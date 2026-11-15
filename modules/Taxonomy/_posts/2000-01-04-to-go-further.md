---
title: "To go further: running MetaPhlAn4"
---

# To go further: running MetaPhlAn4

In the session, we explored precomputed MetaPhlAn4 profiles. This page shows how to run MetaPhlAn4 yourself on your own data.

---

## What is MetaPhlAn4?

**MetaPhlAn4** (Metagenomic Phylogenetic Analysis) profiles the taxonomic composition of microbial communities from shotgun metagenomic reads.

- It uses a database of **clade-specific marker genes** (ChocoPhlAn): genes that are unique to a species, a genus, a family...
- Reads are mapped to these marker genes with Bowtie2; reads that do not map to any marker are not used
- Relative abundances are calculated from the coverage of the marker genes
- It prioritizes **specificity over sensitivity**: few false positives, but organisms without markers in the database are missed

---

## Setup

```bash
conda activate /shared/team/conda/aliseponsero.mmb-dtp/metaphlan

DB=[PATH TO THE METAPHLAN DATABASE]
READS=[PATH TO THE READS]
```

Check the installation and look at the options:

```bash
metaphlan --version
metaphlan --help
```

The database folder contains:
- `*.bt2l`: Bowtie2 index files, used to map the reads
- `*.pkl`: the marker gene database

```bash
ls -lh $DB
```

---

## Running MetaPhlAn4

**Key parameters:**
- Input files: for paired-end reads, the two files separated by a comma, with **no space**
- `--input_type fastq`: format of the input
- `--db_dir`: path to the database folder
- `--offline`: don't check for database updates
- `--nproc`: number of CPUs to use
- `-o`: output file
- `--mapout`: save the Bowtie2 mapping, to re-run MetaPhlAn4 with other parameters without mapping the reads again
- Unclassified estimation: see `metaphlan --help` for the option in your version

**Example for the T0 sample:**

```bash
metaphlan $READS/ERR2231567_1.fastq.gz,$READS/ERR2231567_2.fastq.gz \
    --input_type fastq \
    --db_dir $DB \
    --offline \
    --nproc 4 \
    -o T0_ERR2231567_profile.txt \
    --mapout T0_ERR2231567_mapout.bz2
```

---

## Understanding the output

The output is a tab-delimited file. After a few header lines starting with `#`, each line is a taxon:

```
#clade_name	NCBI_tax_id	relative_abundance
UNCLASSIFIED	-1	63.40501
k__Bacteria	2	35.440329843496194
k__Eukaryota	2759	1.1546597496384208
...
```

(First lines of the T0 sample of the coffee fermentation; the 4th column, `additional_species`, is not shown.)

**Column 1: taxonomic lineage**, with `|` as separator and a prefix for each rank:

| Prefix | Rank |
|---|---|
| `k__` | Kingdom |
| `p__` | Phylum |
| `c__` | Class |
| `o__` | Order |
| `f__` | Family |
| `g__` | Genus |
| `s__` | Species |
| `t__` | SGB (species-level genome bin) |

**Relative abundance (%)**: all taxonomic levels are reported. Values at each level sum to 100%, or less if the unclassified fraction is estimated.

**UNCLASSIFIED**: when the unclassified fraction is estimated, an extra line gives the percentage of the community that could not be classified. For example, `UNCLASSIFIED	63.4` in the T0 sample of the coffee fermentation.

The output can be explored with Pavian, as in the session.

---

## Resources

- **MetaPhlAn4 documentation:** https://github.com/biobakery/MetaPhlAn
- **bioBakery forum:** https://forum.biobakery.org/c/Microbial-community-profiling/MetaPhlAn/
- **MetaPhlAn4 paper:** Blanco-Míguez et al. (2023) Nature Biotechnology
