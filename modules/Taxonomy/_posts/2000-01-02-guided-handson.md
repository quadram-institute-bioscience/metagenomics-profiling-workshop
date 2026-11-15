---
title: "Guided hands-on: Kraken2 and Bracken"
---

# Guided hands-on: profiling a mock community with Kraken2 and Bracken

In this guided session we will run Kraken2 and Bracken together on a small **mock community**: a dataset where we know exactly which genomes are present and in which proportions. Because we know the answer, we can check how well the tools recover it, and understand where and why they differ from the truth.

**What you will do:**
- Look at the input reads and the Kraken2 database
- Classify the reads with Kraken2 and read the report
- Re-estimate species abundances with Bracken
- Compare both results with the expected composition
- Visualise the reports with Pavian

---

## The mock community

The mock dataset contains **500 read pairs** simulated with [InSilicoSeq](https://github.com/HadrienG/InSilicoSeq) (HiSeq error model) from 5 genomes of the HumGut [CITATION NEEDED] collection of human gut genomes.

| Genome | Organism (GTDB taxonomy) | Read pairs | Proportion |
|---|---|---|---|
| HumGut_1 | *Phocaeicola vulgatus* (strain 1) | 175 | 35% |
| HumGut_39 | *Bacteroides uniformis* | 125 | 25% |
| HumGut_65 | *Blautia_A wexlerae* | 100 | 20% |
| HumGut_44 | *Phocaeicola vulgatus* (strain 2) | 60 | 12% |
| HumGut_46 | *Blautia_A* sp. (no species name in GTDB) | 40 | 8% |
| **Total** | | **500** | **100%** |

> **Before we start:** look at the table. Which of these genomes do you expect Kraken2 to be able to tell apart? Which ones might be merged?

---

## Setup

Activate the shared environment and define the paths we will use. Replace `{your_name}` with your user name.

```bash
conda activate /shared/team/conda/aliseponsero.mmb-dtp/kraken2

DIR="/shared/team/2026_training/week5/day2_profiling/1.Guided_demo"
DB="${DIR}/k2_standard_08_GB_20261022"
export WSUSER=/shared/team/users/{your_name}/

mkdir -p $WSUSER/guided_demo
cd $WSUSER/guided_demo
```

---

## Step 1: Look at the reads

The reads are in two files: `mock_R1.fastq.gz` (forward reads) and `mock_R2.fastq.gz` (reverse reads). Each read takes 4 lines in a FASTQ file: a header, the sequence, a `+` separator and the quality scores.

```bash
zcat $DIR/mock_R1.fastq.gz | head -n 8
```

Count the number of reads in the R1 file:

```bash
zcat $DIR/mock_R1.fastq.gz | wc -l
```

> **Question:** the command returns a number of lines. How many reads does it correspond to?

---

## Step 2: Look at the database

Kraken2 compares the k-mers of each read to a pre-built database. We use the **Standard-8** database: the standard Kraken2 database (RefSeq archaea, bacteria, viruses, plasmids, human) capped at 8 GB so that it fits in a small computer's memory.

```bash
ls $DB
```

- `hash.k2d`, `opts.k2d`, `taxo.k2d`: the Kraken2 database itself
- `databaseXXXmers.kmer_distrib`: files used by Bracken, one per read length
- `inspect.txt`: the list of taxa in the database

Is each species of our mock community in the database?

```bash
grep "Phocaeicola vulgatus" $DB/inspect.txt
grep "Bacteroides uniformis" $DB/inspect.txt
grep "Blautia wexlerae" $DB/inspect.txt
```

> **Question:** the database uses the NCBI taxonomy, not GTDB. Try `grep "Blautia_A" $DB/inspect.txt`. What do you get, and why?

---

## Step 3: Run Kraken2

```bash
kraken2 --db $DB \
    --threads 6 \
    --report kraken2_report.txt \
    --output kraken2_output.txt \
    --paired $DIR/mock_R1.fastq.gz $DIR/mock_R2.fastq.gz
```

- `--db`: path to the database
- `--threads 6`: number of CPUs to use
- `--report`: summary of the classification per taxon (this is the file we will mostly use)
- `--output`: classification of every single read pair
- `--paired`: the two files are the two reads of each pair, so Kraken2 classifies each pair together

---

## Step 4: Read the Kraken2 report

```bash
cat kraken2_report.txt
```

The report has 6 columns:

| Column | Meaning |
|---|---|
| 1 | Percentage of reads in this taxon and all taxa below it |
| 2 | Number of reads in this taxon and all taxa below it (the "clade") |
| 3 | Number of reads assigned directly to this taxon |
| 4 | Rank code: U (unclassified), R (root), D (domain), K (kingdom), P (phylum), C (class), O (order), F (family), G (genus), S (species), S1 (below species) |
| 5 | NCBI taxonomy ID |
| 6 | Taxon name (the indentation shows the taxonomy tree) |

Answer the questions below using the report.

1. What percentage of reads is unclassified? All our genomes come from the human gut: why are some reads not classified?
2. How many reads are assigned **directly** to *Phocaeicola vulgatus* (column 3)? How many reads stop at the genus *Phocaeicola*?
3. Look at the reads assigned directly to Bacteroidales, Bacteroidaceae, *Phocaeicola* and *Bacteroides*. Why does Kraken2 assign reads to these higher levels instead of a species?
4. How many species are listed in the report (rank S)? How many did you expect? Which ones are false positives, and how many reads do they have?
5. Can Kraken2 separate the two strains of *P. vulgatus*? Where do you think the reads of the *Blautia_A* sp. genome went?

<details markdown="1">
<summary>Answers</summary>

1. 60 read pairs (12%) are unclassified. Possible reasons: the database is capped at 8 GB, so part of the k-mers were removed; the simulated reads contain sequencing errors; the exact strains in our mock may not be in the database.
2. 113 read pairs are assigned to *P. vulgatus*, and 37 stop at the genus *Phocaeicola*.
3. These reads contain k-mers shared by several species (or genera). Kraken2 assigns them to the **lowest common ancestor** (LCA) of all the taxa that share these k-mers.
4. 17 species are listed. We expected 3 or 4 (depending on where the *Blautia_A* sp. reads go). The 14 extra species have between 1 and 4 reads each (22 reads in total): for example *Phocaeicola dorei*, *Bacteroides eggerthii*, *Segatella copri*.
5. No: both strains are reported together as *P. vulgatus*. The *Blautia_A* sp. genome has no species name, and in HumGut it is clustered with *B. wexlerae* at species level, so its reads were most likely classified as *Blautia wexlerae* or stopped at the genus *Blautia*.

</details>

### Optional: the per-read output

Each line of `kraken2_output.txt` is one read pair: classified (C) or unclassified (U), the read name, the taxonomy ID assigned, the read lengths, and the k-mer matches.

```bash
head kraken2_output.txt
```

Count the classified and unclassified read pairs:

```bash
cut -f1 kraken2_output.txt | sort | uniq -c
```

> **Question:** do these numbers match the first lines of the report?

---

## Step 5: Run Bracken

Kraken2 leaves many reads at the genus level or above. **Bracken** uses the database to estimate how these reads should be redistributed among the species detected in the sample.

```bash
bracken -d $DB \
    -i kraken2_report.txt \
    -o bracken_output.txt \
    -w bracken_report.txt \
    -r 150 \
    -l S
```

- `-d`: the same database used for Kraken2
- `-i`: the Kraken2 report
- `-o`: the Bracken abundance table
- `-w`: a new report in the same format as the Kraken2 report
- `-r 150`: the read length; Bracken uses the matching `database150mers.kmer_distrib` file
- `-l S`: estimate abundances at the species level

By default, Bracken ignores species with fewer than 10 reads (the threshold). We keep this default.

Look at the summary printed on the screen:

> **Questions:**
> 1. How many species are above the threshold?
> 2. How many reads were discarded? Which reads are these?
> 3. How many reads were redistributed to species level?

<details markdown="1">
<summary>Answers</summary>

1. 3 species (out of 17).
2. 22 reads: the reads of the 14 species with fewer than 10 reads (the false positives seen in Step 4).
3. 158 reads were redistributed from higher levels to the 3 species.

</details>

---

## Step 6: Read the Bracken report

```bash
cat bracken_report.txt
```

> **Questions:**
> 1. Which species are left? What are their percentages?
> 2. The unclassified reads have disappeared, and the root is now at 100%. On how many reads are the percentages calculated?

<details markdown="1">
<summary>Answers</summary>

1. *Phocaeicola vulgatus* (42.67%), *Bacteroides uniformis* (29.21%), *Blautia wexlerae* (28.13%).
2. On 417 reads: the 500 reads minus the 60 unclassified, the 22 discarded, and 1 read that could not be redistributed. Bracken percentages are relative to the classified reads it kept, not to the whole sample.

</details>

---

## Step 7: Compare with the expected composition

Fill in the table using the Kraken2 report (column 2, rank S) and the Bracken report.

| Species | Expected (read pairs) | Kraken2 (species level) | Bracken |
|---|---|---|---|
| *Phocaeicola vulgatus* (2 strains) | 235 (47%) | | |
| *Bacteroides uniformis* | 125 (25%) | | |
| *Blautia wexlerae* + *Blautia_A* sp. | 140 (28%) | | |

> **Questions:**
> 1. Which tool gives a profile closer to the expected one?
> 2. Which species is the most underestimated? Where could its reads have gone?

<details markdown="1">
<summary>Answers</summary>

| Species | Expected (read pairs) | Kraken2 (species level) | Bracken |
|---|---|---|---|
| *Phocaeicola vulgatus* (2 strains) | 235 (47%) | 113 | 177 (42.67%) |
| *Bacteroides uniformis* | 125 (25%) | 56 | 121 (29.21%) |
| *Blautia wexlerae* + *Blautia_A* sp. | 140 (28%) | 90 | 117 (28.13%) |

1. Bracken is much closer to the expected proportions, because it recovers the reads Kraken2 left at higher levels.
2. *P. vulgatus* is the most underestimated. Its missing reads could be unclassified, or assigned to higher levels that Bracken could not redistribute to it. The report alone does not tell us which: we would need to trace each read back to its genome of origin.

</details>

---

## Step 8: Visualise the reports with Pavian

[Pavian](https://github.com/fbreitwieser/pavian) is an interactive tool to explore Kraken2 and Bracken reports, without writing code.

### Install Pavian

Open RStudio and run:

```r
# Install remotes if not already installed
if (!require(remotes)) {
    install.packages("remotes")
}

# Install Pavian from GitHub
remotes::install_github("fbreitwieser/pavian")
```

Installation takes 2-3 minutes and is only needed once.

### Launch Pavian

```r
# Launch Pavian
pavian::runApp(port=5000)
```

Pavian opens in your web browser at `http://localhost:5000`. Keep the R console open while using Pavian.

> **Tip:** if port 5000 is busy, try `pavian::runApp(port=5001)`

### Download the reports

1. Open the file browser on the notebook (top-left)
2. Navigate to `{your_name}/guided_demo/`
3. Select `kraken2_report.txt` and `bracken_report.txt`
4. Right-click and choose **Download**
5. Save them on your computer

### Explore the reports

In Pavian, click **Browse...**, select both reports and click **Open**. Then:

1. Open the **Sample** view for the Kraken2 report and look at the Sankey diagram. Can you see the reads stopping at genus, family and order level?
2. Do the same for the Bracken report. What changed?
3. In the **Comparison** view, compare the two reports side by side at species level.

---

## Key messages

- Kraken2 is a **classifier**: it classifies each read independently, and assigns reads with shared k-mers to the lowest common ancestor.
- Even on a simple mock community, Kraken2 reports **unclassified reads** and **false positives** with a few reads each.
- Bracken **redistributes** reads to species level and filters out low-count species, giving a profile much closer to the truth.
- Bracken **removes the unclassified fraction** from its percentages: keep this in mind when you compare samples.
- Results depend on the **database** (what it contains and which taxonomy it uses) and the tools cannot separate strains of the same species.
