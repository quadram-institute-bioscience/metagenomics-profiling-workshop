---
title: "Humann3 Hands-on"
---
# Exploring Functional Profiles with HUMAnN3


---

> The Workshop material for this hands-on is available to download in 
> [Figshare]()


## Hands-On: Exploring Coffee Fermentation Functional Profiles

### Prerequisites

Make sure you have:
- Access to the climb shared project
- Data symlinked / copied to your workspace

```bash
export WSUSER=/shared/team/users/{your_name}/
cd $WSUSER

# Link to precomputed HUMAnN3 results
ln -s /shared/team/2025_training/week5/tutorial/Session3_Functional/Single_sample .

ls -lh func_precomputed/humann3/
```

- RStudio access on the notebook

### The Dataset

**Coffee bean fermentation time series from Ecuador:**

- **T0** (0 hours): Initial colonization, diverse community
- **T16** (16 hours): Early fermentation, *Leuconostoc* emergence
- **T24** (24 hours): Active fermentation, lactic acid bacteria dominance
- **T48** (48 hours): Late fermentation, mature community

**Biological question:** How does microbial metabolism shift during fermentation? Which pathways increase or decrease? Do different species contribute at different stages?

---

## Exploring HUMAnN3 Results in R

### Open RStudio and Load the Script

**[Link to R Markdown: HUMAnN3 Pathway Exploration](explore_humann3.Rmd)**

Copy the script to your workspace:

```bash
cd $WSUSER
cp /shared/team/2025_training/week5/tutorial/Session3_Functional/Single_sample/Session3_humann3.Rmd .
```

Open RStudio and open the `Session3_humann3.Rmd` file.

---

## Normalization Best Practices

Different analyses require different normalizations. Here's a guide:

### Gene Families

**Raw output:** RPK (reads per kilobase) - normalized by gene length only

**For comparison across samples:** Convert to CPM
```bash
humann_renorm_table --input genefamilies.tsv --output genefamilies_cpm.tsv --units cpm
```

**For compositional analysis:** Convert to relative abundance
```bash
humann_renorm_table --input genefamilies.tsv --output genefamilies_relab.tsv --units relab
```

### Pathway Abundances

**Raw output:** Already in CPM (copies per million)

**For compositional analysis:** Convert to relative abundance if needed

**For visualization:** Often log-transform for better dynamic range (add pseudocount first)

### General Rules

**Comparing same feature across samples:** CPM normalization

**Comparing different features within a sample:** Relative abundance (sums to 1)

**Statistical testing:** Depends on the tool - DESeq2 uses raw counts, ALDEx2 uses CLR-transformed

**Always document:** State clearly which normalization you used in methods and figure legends!

---

## Functional Database Regrouping

HUMAnN3 outputs use UniRef90 IDs by default, but you can regroup to other functional classification systems.

### Available Regrouping Options

**GO (Gene Ontology)** - Biological process, molecular function, cellular component categories. Useful for broad functional categories.

**EC (Enzyme Commission)** - Enzyme classification numbers. Good for enzyme-centric analyses.

**KEGG (Kyoto Encyclopedia of Genes and Genomes)** - KEGG Orthology groups and KEGG pathways. Widely used, good for integration with other databases.

**Pfam** - Protein family classifications. Useful for domain-level analyses.

**eggNOG** - Evolutionary genealogy classifications. Good for phylogenetic context.

### How to Regroup

```bash
# Regroup to EC numbers
humann_regroup_table --input genefamilies.tsv --output genefamilies_ec.tsv --groups uniref90_ec

# Regroup to Gene Ontology
humann_regroup_table --input genefamilies.tsv --output genefamilies_go.tsv --groups uniref90_go

# Regroup to KEGG
humann_regroup_table --input genefamilies.tsv --output genefamilies_ko.tsv --groups uniref90_ko
```

### When to Use Each

**Use GO** for high-level functional categories and comparing broad functional classes.

**Use EC** for enzyme-focused questions and connecting to metabolomics data.

**Use KEGG** for pathway integration and connecting to other omics data.

**Use multiple** if you want different perspectives on the same data - they're complementary!

---