---
title: "Kraken2 & Bracken"
---
# Kraken2 & Bracken: K-mer Based Taxonomic Classification

---

> The Workshop materia;ls available to download 
> [Figshare](https://figshare.com/articles/presentation/MMB_DTP_workshop_-_Introduction_to_taxonomic_profiling_of_metagenomes/30704492)

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

```bash
conda deactivate

conda activate /shared/team/conda/aliseponsero.mmb-dtp/kraken2
```

- RStudio access on the notebook

---

## Part 1: Understanding Kraken2 Databases

### Database Organization

Kraken2 requires a pre-built database. For this workshop, we're using the **standard_8gb** database located at:

```
/shared/team/2025_training/week5/databases/kraken2/
```

**Key files in the database:**
- `hash.k2d` - K-mer hash table (the main database file)
- `opts.k2d` - Options and parameters used to build the database
- `taxo.k2d` - Taxonomy information
- `database*mers.kmer_distrib` - Bracken k-mer distribution files (for different read lengths)

**To explore:**
```bash
ls -lh /shared/team/2025_training/week5/databases/kraken2/
```

### Available Pre-built Databases

Kraken2 offers many pre-built database options (see https://benlangmead.github.io/aws-indexes/k2):

**Standard databases:**
- `k2_standard` - Bacteria, archaea, viruses, plasmid, human (~137 GB)
- `k2_standard_8gb` - Reduced standard (~8 GB) ← **We're using this**
- `k2_standard_16gb` - Reduced standard (~16 GB)

**PlusPF series:**
- `k2_pluspf` - Standard + protozoa + fungi
- `k2_pluspf_8gb` - Reduced PlusPF

💡 **Database choice matters!** Larger = more sensitivity, but slower and more memory.

---

## Part 2: Running Kraken2

### Step 1: Activate the Environment

```bash
conda deactivate  # If you were in metaphlan environment
conda activate /shared/team/conda/aliseponsero.mmb-dtp/kraken2
```

### Step 2: Check Installation

```bash
kraken2 --version
```

### Step 3: View Help

```bash
kraken2 --help
```

**Key parameters to understand:**
- `--db` - Path to Kraken2 database
- `--threads` - Number of threads to use
- `--quick` - Quick mode (faster, slightly less sensitive)
- `--report` - Output report file (summary - this is what we analyze!)
- `--output` - Output classification file (per-read classifications - large!)
- `--paired` - Input files are paired-end reads

### Step 4: Construct the Command

**Template:**
```bash
kraken2 --db [DATABASE_PATH] \
    --threads [N] \
    --report [REPORT_FILE] \
    --output [OUTPUT_FILE] \
    --paired [FORWARD] [REVERSE]
```

💡 **Tip:** The `--output` file is very large (one line per read). For most analyses, you only need the `--report` file.

**This would normally take 5-15 minutes, but we'll look at precomputed results.**


---

## Part 3: Understanding Kraken2 Output

### Kraken2 Report Format

Open a Kraken precomputed output (e.g. T0_ERR2231567_profiles.txt)

The report file is tab-delimited with 6 columns:

```
%_READS  READS_CLADE  READS_TAXON  RANK  TAXID  NAME
5.43     1087         512          S     1234   Leuconostoc mesenteroides
```

**Columns:**
1. **%_READS** - Percentage of reads in this clade (including children)
2. **READS_CLADE** - Number of reads in this clade (including descendants)
3. **READS_TAXON** - Number of reads assigned directly to this taxon
4. **RANK** - Taxonomic rank:
   - `U` = Unclassified
   - `R` = Root
   - `D` = Domain
   - `P` = Phylum
   - `C` = Class
   - `O` = Order
   - `F` = Family
   - `G` = Genus
   - `S` = Species
5. **TAXID** - NCBI taxonomy ID
6. **NAME** - Scientific name (indented to show hierarchy)

**First line = Unclassified reads**
```
5.21   10420   10420   U   0   unclassified
```

### Key Differences from MetaPhlAn4

| Feature | MetaPhlAn4 | Kraken2 |
|---------|------------|---------|
| Output | Relative abundances (%) | Read counts |
| Unclassified | Usually high (30-70%) | Usually low (5-15%) |
| Species detected | Conservative (~40-50) | Liberal (~100+) |
| False positives | Low | Higher (especially at low abundance) |

---

## Part 4: Running Bracken

### Step 1: Check Bracken Installation

```bash
bracken -v
```

### Step 2: Check Available Read Lengths

Bracken databases are built for specific read lengths. Check what's available:

```bash
ls /shared/team/2025_training/week5/databases/kraken2/database*mers.kmer_distrib
```

You should see files like:
- `database100mers.kmer_distrib` (for 100bp reads)
- `database150mers.kmer_distrib` (for 150bp reads)
- `database200mers.kmer_distrib` (for 200bp reads)

### Step 3: Understand Bracken Parameters

```bash
bracken -h
```

**Key parameters:**
- `-d` - Path to Kraken2 database directory (same as Kraken2 `--db`)
- `-i` - Input Kraken2 report file
- `-o` - Output Bracken abundance file
- `-w` - Output Kraken-style report with redistributed reads
- `-r` - Read length used to build database (e.g., 150)
- `-l` - Taxonomic level (S=species, G=genus, F=family, etc.)
- `-t` - Threshold: minimum reads required (default: 10)

### Step 4: Run Bracken

**Template:**
```bash
bracken -d [DATABASE_PATH] \
    -i [KRAKEN_REPORT] \
    -o [OUTPUT_FILE] \
    -w [BRACKEN_REPORT] \
    -r [READ_LENGTH] \
    -l S \
    -t 10
```

**Example command:**
```bash
mkdir test_bracken

bracken -d /shared/team/2025_training/week5/databases/kraken2 \
    -i Session1_profiling/Kraken2/T0_ERR2231567_profiles.txt \
    -o test_bracken/subset_bracken_species.txt \
    -w test_bracken/subset_bracken_species.report \
    -r 150 \
    -l S \
    -t 10
```

---

## Part 5: Understanding Bracken Output

### Bracken Abundance File Format

The main output file is tab-delimited with 7 columns:

```
name                          taxonomy_id  taxonomy_lvl  kraken_assigned_reads  added_reads  new_est_reads  fraction_total_reads
Leuconostoc mesenteroides     1245         S            450                    125          575            0.0287
```

**Columns:**
1. **name** - Species name
2. **taxonomy_id** - NCBI taxonomy ID
3. **taxonomy_lvl** - Taxonomic level (usually S for species)
4. **kraken_assigned_reads** - Reads Kraken2 assigned directly to this species
5. **added_reads** - Reads redistributed from higher levels ← **KEY!**
6. **new_est_reads** - Total estimated reads (kraken + added)
7. **fraction_total_reads** - Proportion of all classified reads

**The magic is in `added_reads`!** This shows which species gained reads that were originally classified at genus/family level.

---

## Part 6: Exploring Outputs in RStudio

Open RStudio on your notebook and follow along!

### Load Required Libraries

```r
library(tidyverse)
```

### Read Bracken Output

```r
# Read the Bracken output
MYDIR="/shared/team/users/{your_name}/"

bracken <- read_tsv(paste(MYDIR, "Session1_profiling/Bracken/T0_ERR2231567_profiles.txt", sep="/"),
                    show_col_types = FALSE)

```

### Compare Bracken to MetaPhlAn4:

```r
# Subset Bracken species
bracken_rel <- bracken %>% filter(fraction_total_reads>0) %>%
    mutate(relative_abundance=fraction_total_reads*100) %>%
    arrange(desc(relative_abundance))

# Get MetaPhlAn4 species
profile_no_uncl <- read_tsv(paste(MYDIR, "Session1_profiling/Metaphlan/T0_ERR2231567.profile.txt", sep="/"), 
                    skip = 4,
                    show_col_types = FALSE) %>%
        rename("clade_name"=`#clade_name`)
        
metaphlan_species <- profile_no_uncl %>%
  filter(str_detect(clade_name, "s__"),
         !str_detect(clade_name, "t__")) %>%
  mutate(species_name = str_extract(clade_name, "s__[^|]+$")) %>%
  mutate(species_name = str_remove(species_name, "s__")) %>%
  mutate(species_name = str_replace_all(species_name, "_", " ")) %>%
  select(species_name, relative_abundance) %>%
  arrange(desc(relative_abundance))

# Compare species counts
cat("\n=== Method Comparison ===\n")
cat("MetaPhlAn4 species detected:", nrow(metaphlan_species), "\n")
cat("Bracken species detected:", nrow(bracken), "\n")

# Show top 10 from each method side-by-side
cat("\nTop 10 species comparison:\n")
comparison <- tibble(
  Rank = 1:10,
  MetaPhlAn4 = metaphlan_species$species_name[1:10],
  `MetaPhlAn4_%` = round(metaphlan_species$relative_abundance[1:10]),
  Bracken = bracken_rel$name[1:10],
  `Bracken_%` = round(bracken_rel$relative_abundance[1:10])
)

print(comparison)
```


### Visualize Bracken Results

```r
bracken_grouped <- bracken_rel %>%
    mutate(species_name=ifelse(relative_abundance<5, "Other", name)) %>%
    group_by(species_name) %>%
    summarize(relative_abundance=sum(relative_abundance)) %>%
    select(species_name, relative_abundance)


bracken_grouped %>%
  mutate(sample="T0") %>%
  ggplot(aes(x = sample, y = relative_abundance, fill=species_name)) +
  geom_bar(stat="identity") +
  coord_flip() +
  labs(title = "Species composition at T0",
       x = "Species",
       y = "Relative Abundance (%)") +
  theme_minimal()
```


---

## Additional Resources

### Kraken2
📖 **Kraken2 Manual:** https://github.com/DerrickWood/kraken2/wiki/Manual  
📖 **Pre-built Databases:** https://benlangmead.github.io/aws-indexes/k2  
📊 **Kraken2 Paper:** Wood et al. (2019) Genome Biology  

### Bracken
📖 **Bracken GitHub:** https://github.com/jenniferlu717/Bracken  
📊 **Bracken Paper:** Lu et al. (2017) PeerJ Computer Science  

### Database Building
📖 **Custom Kraken2 Databases:** https://github.com/DerrickWood/kraken2/wiki/Manual#custom-databases  
📖 **KrakenTools:** https://github.com/jenniferlu717/KrakenTools (for manipulating Kraken2/Bracken outputs)

---
