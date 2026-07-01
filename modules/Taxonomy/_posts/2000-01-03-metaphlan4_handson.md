---
title: "MetaPhlAn4 hands-on"
---
# MetaPhlAn4: Marker Gene-Based Taxonomic Profiling

---

> The Workshop material is available to download in 
> [Figshare]()

---

## Part 1: Understanding the Database

### Database Organization

MetaPhlAn4 requires a pre-built database of marker genes. For this workshop, the database is located at:

```bash
/shared/team/conda/telatin.cli-giba-2026/mpa/lib/python3.13/site-packages/metaphlan/metaphlan_databases
```

**Key files in the database:**
- `*.bt2l` - Bowtie2 index files (for read mapping)
- `*.pkl` - Marker gene database (pickled Python object)
- `*_species.txt` - List of species in the database

**To explore:**
```bash
ls -lh /shared/team/conda/telatin.cli-giba-2026/mpa/lib/python3.13/site-packages/metaphlan/metaphlan_databases
```

## Part 2: Running MetaPhlAn4

### Step 1: Activate the Environment

```bash
conda activate /shared/team/conda/telatin.cli-giba-2026/mpa
```

### Step 2: Check Installation

```bash
metaphlan --version
```

### Step 3: View Help

```bash
metaphlan --help
```

**Key parameters to understand:**
- Input files (paired-end: comma-separated, no space!)
- `--input_type fastq` - Specifies input format
- `--db_dir` - Path to database directory -- default location if omitted
- `--offline` - Don't check for database updates
- `--nproc` - Number of processors to use
- `-o` or `--output_file` - Path to output file
- `--mapout` - Save Bowtie2 alignment (for re-analysis)
- `--skip_unclassified_estimation` - Turn off unclassified estimation (default: ON)

### Step 4: Construct the Command

For paired-end reads, MetaPhlAn4 requires input files separated by a comma with **NO SPACE**.

**Template:**
```bash
metaphlan [FORWARD],[REVERSE] \
    --input_type fastq \
    --offline \
    --nproc [N] \
    -o [OUTPUT] \
    --mapout [MAPPING_OUTPUT]
```

You can test this command on a small subset of reads:

```bash
metaphlan $WSUSER/small_subset/subset_R1.fq.gz,$WSUSER/small_subset/subset_R2.fq.gz \
    --input_type fastq \
    --offline \
    --nproc 6 \
    -o $WSUSER/test_mpa.txt \
    --mapout $WSUSER/test_mpa.bt2
```
⏱️ **Runtime:** 5-10 minutes for the subset sample --> We will use precomputed results

💡 **Tip:** The `--mapout` file can be reused to re-run MetaPhlAn4 with different parameters without re-mapping reads!

---

## Part 3: Understanding MetaPhlAn4 Output

### Output Format

Open the precomputed folder for Metaphlan outputs you copied in your local personal folder (e.g. T0_ERR2231567.profile.txt).

MetaPhlAn4 produces a tab-delimited file with two columns:

```
clade_name	relative_abundance
k__Bacteria|p__Firmicutes|c__Bacilli|...|s__Leuconostoc_mesenteroides	15.42
k__Bacteria|p__Firmicutes|c__Bacilli|...|s__Lactiplantibacillus_plantarum	8.73
...
```

**Column 1: Taxonomic lineage**
- Uses `|` as separator
- Taxonomic prefixes:
  - `k__` = Kingdom
  - `p__` = Phylum
  - `c__` = Class
  - `o__` = Order
  - `f__` = Family
  - `g__` = Genus
  - `s__` = Species
  - `t__` = Strain (clade)

**Column 2: Relative abundance (%)**
- Values sum to 100% (for classified reads)
- All taxonomic levels are reported

**Optional: UNCLASSIFIED entry**
- If unclassified estimation is enabled, an additional line shows the percentage of unclassified reads (e.g T0_ERR2231567.unclprofile.txt)
- Format: `UNCLASSIFIED	63.4` (63.4% of reads unclassified)

---

## Part 4: Exploring Outputs in RStudio

Open RStudio on your notebook and follow along!

We will need to first install the packages we will use in this session

```r
install.packages("tidyverse")
```

### Load Required Libraries

```r
library(tidyverse)
```

### Read the MetaPhlAn4 Profile

```r
MPA_DIR="/shared/team/datasets/day2/1.metaphlan4_precomputed"

profile_no_uncl <- read_tsv(paste(MPA_DIR, "T0_ERR2231567.profile.txt", sep="/"), 
                    skip = 4,
                    show_col_types = FALSE) %>%
        rename("clade_name"=`#clade_name`)


```

### Compare Profiles With/Without Unclassified Estimation

```r
MPA_DIR="/shared/team/datasets/day2/1.metaphlan4_precomputed"
# Read profile WITH unclassified
profile_uncl <- read_tsv(paste(MPA_DIR, "T0_ERR2231567.unclprofile.txt", sep="/"), 
                         skip = 4,
                    show_col_types = FALSE) %>%
        rename("clade_name"=`#clade_name`)

# Check for UNCLASSIFIED entry
profile_uncl %>%
  filter(clade_name=="UNCLASSIFIED")
  
# Compare first few rows
profile_uncl %>% head(5)
profile_no_uncl %>% head(5)

```

**Question:** What does a high unclassified percentage tell us about the sample?

### Extract and Count Species

```r
# Extract only species-level entries (has "s__" but not "t__")
species <- profile_no_uncl %>%
  filter(str_detect(clade_name, "s__"),
         !str_detect(clade_name, "t__"))

# Count species
n_species <- species %>% nrow()
cat("Number of species detected:", n_species, "\n")
```

**Show top 10 most abundant species:**

```r
# Sort by abundance in descending order and show top 10
species_sorted <- species %>%
  arrange(desc(.[[3]]))  # Sort by rel abundance

cat("\nTop 10 most abundant species:\n")
species_sorted %>% head(10)

# Extract just the species names for easier reading
species_clean <- species_sorted %>%
  mutate(species_name = str_extract(clade_name, "[^|]+$")) %>%  # Extract after last |
  select(species_name, relative_abundance)  # Rename abundance column

# Show top 10
species_clean %>% 
  head(10) %>%
  print()

# group all species with relative abundance below 5% as "Other"
species_grouped <-  species_clean %>%
    mutate(species_name=ifelse(relative_abundance<5, "Other", species_name)) %>%
    group_by(species_name) %>%
    summarize(relative_abundance=sum(relative_abundance))

```

---

**Quick visualization :**

```r
# Bar plot of top species using ggplot2
species_clean %>%
  head(10) %>%
  mutate(species_name = str_trunc(species_name, 40)) %>%  # Truncate long names
  mutate(species_name = fct_reorder(species_name, relative_abundance)) %>%  # Order by abundance
  ggplot(aes(x = species_name, y = relative_abundance)) +
  geom_col(fill = "steelblue") +
  coord_flip() +
  labs(title = "Top 10 Species in ERR2231567",
       x = "Species",
       y = "Relative Abundance (%)") +
  theme_minimal()

species_grouped %>%
  mutate(sample="T0") %>%
  mutate(species_name = str_trunc(species_name, 40)) %>%  # Truncate long names
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

📖 **MetaPhlAn4 Documentation:** https://github.com/biobakery/MetaPhlAn  
💬 **BioBakery Forum:** https://forum.biobakery.org/c/Microbial-community-profiling/MetaPhlAn/  
📊 **MetaPhlAn4 Paper:** Blanco-Míguez et al. (2023) Nature Biotechnology  

---