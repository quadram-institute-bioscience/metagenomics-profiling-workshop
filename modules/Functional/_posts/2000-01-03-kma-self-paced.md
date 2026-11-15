---
title: "Self-paced: resistance and virulence genes with KMA"
---

# Self-paced exploration: antimicrobial resistance and virulence genes

HUMAnN3 tells us what the whole community *can do*. Sometimes we have a more targeted question: **are there genes of concern in this food?** For example, antimicrobial resistance genes (ARGs), or virulence genes from pathogens.

In this exercise, you will search the coffee fermentation reads for these genes, using **KMA** and two small, curated databases.

Work at your own pace, alone or in pairs. Write down your answers: we will discuss them in the recap.

---

## KMA in a nutshell

KMA (k-mer alignment) maps raw reads against a database of reference genes, and reports, for each gene it finds:
- how much of the reference gene is covered by reads (**coverage**)
- how similar the reads are to the reference gene (**identity**)
- how many times the gene is covered on average (**depth**), a rough measure of abundance

KMA was designed for databases where many genes are very similar to each other (for example, many variants of the same resistance gene), which is the case for both databases we use today.

## The two databases

| Database | Content |
|---|---|
| **ResFinder** | Acquired antimicrobial resistance genes, grouped by antibiotic class |
| **VFDB (core set A)** | Virulence genes from bacterial pathogens, with experimentally verified virulence factors |

> **Before you start:** what do you expect to find in a coffee fermentation? Many resistance genes, a few, or none? Write down your guess.

---

## Setup

```bash
conda activate /shared/team/conda/aliseponsero.mmb-dtp/kma/

DIR=/shared/team/2026_training/week5/day2_function
DBDIR=$DIR/kma_db

mkdir -p $WSUSER/kma
cd $WSUSER/kma
```

We will compare two timepoints: the **start** (T0, ERR2231567) and the **end** (T64, ERR2231572) of the fermentation.

---

## Part 1: Run KMA

Here is the command for the T0 sample against ResFinder:

```bash
kma -ipe $DIR/ERR2231567_1.fastq.gz $DIR/ERR2231567_2.fastq.gz \
    -o T0_resfinder \
    -t_db $DBDIR/resfinder \
    -t 4
```

- `-ipe`: the two files of paired-end reads
- `-o`: the prefix of the output files
- `-t_db`: the database to search
- `-t 4`: use 4 CPUs

Each run takes about 2 minutes.

**Your turn:** write and run the three other commands:
1. T0 against VFDB (database: `$DBDIR/vfdb_setA`), output `T0_vfdb`
2. T64 against ResFinder, output `T64_resfinder`
3. T64 against VFDB, output `T64_vfdb`

> Tip: while a command runs, go on to Part 2 with the T0 results.

---

## Part 2: Read the results table

KMA creates several files. The one we need is the `.res` file: one line per gene found.

```bash
ls T0_resfinder*

column -t -s $'\t' T0_resfinder.res | less -S
```

(Press `q` to quit `less`.)

The columns we will use are:

| Column | Name | Meaning |
|---|---|---|
| 1 | Template | The reference gene found |
| 5 | Template_Identity | % of the reference gene's bases that are matched by the reads |
| 6 | Template_Coverage | % of the reference gene covered by reads |
| 7 | Query_Identity | % identity between the reads and the reference, only on the covered part |
| 9 | Depth | How many times the gene is covered on average |

**Template_Identity** combines similarity and coverage: if reads match only 3% of a gene, even perfectly, the Template_Identity is around 3%.

How many genes did KMA report? (The first line is the header, starting with `#`.)

```bash
grep -v "^#" T0_resfinder.res | wc -l
```

> **Questions:**
> 1. How many resistance genes are reported in T0?
> 2. Find a gene with a Query_Identity of 100% but a Template_Coverage below 5%. Is this good evidence that the gene is present? Why?
> 3. When T64 is ready: the gene `tet(M)` appears several times (`tet(M)_4`, `tet(M)_5`, `tet(M)_13`). These are variants of the same gene in the database. Which variant fits the reads best?

---

## Part 3: Keep only convincing hits

We keep a gene only if most of it is covered by reads that are very similar to the reference. As ResFinder does, we filter on **Template_Identity** (column 5) and **Template_Coverage** (column 6).

Keep hits with at least 90% for both, and show only the useful columns:

```bash
grep -v "^#" T64_resfinder.res \
    | awk -F'\t' '$5 >= 90 && $6 >= 90' \
    | cut -f1,5,6,9 \
    | sort -t$'\t' -k4,4nr
```

- `awk -F'\t' '$5 >= 90 && $6 >= 90'`: keep lines where column 5 and column 6 are at least 90
- `cut -f1,5,6,9`: keep the gene name, Template_Identity, Template_Coverage and depth
- `sort -t$'\t' -k4,4nr`: sort by depth (4th column after `cut`), highest first

Which antibiotic class does a gene belong to? ResFinder has one file per class, so you can search for the gene name in these files:

```bash
grep -F -l "tet(M)_4_X75073" $DBDIR/resfinder_db/*.fsa | grep -v all.fsa
```

(`all.fsa` contains all the genes, so we remove it from the list with `grep -v`.)

> **Questions:**
> 4. How many resistance genes pass the filter in T0? And in T64?
> 5. Which antibiotic classes are they associated with?
> 6. Try a less strict filter: 80% and 60%. Does it change the result?
> 7. Reads alone do not tell us which bacterium carries these genes. Which approach (seen in the next sessions) could tell us?

---

## Part 4: Virulence genes

Now look at the VFDB results. The VFDB gene names are long: they contain an ID, **the gene name** between brackets, a description, the virulence factor and its category, and, at the end, **the organism the reference gene comes from**:

```
VFG038365(gb|WP_011705044) (hcp1) HcpA-like protein [T6SS (VF0480) - Effector delivery system (VFC0086)] [Aeromonas hydrophila subsp. hydrophila ATCC 7966]
```

Filter the T0 hits as in Part 3, and look at them:

```bash
grep -v "^#" T0_vfdb.res \
    | awk -F'\t' '$5 >= 90 && $6 >= 90' \
    | cut -f1,5,6,9 \
    | sort -t$'\t' -k4,4nr \
    | less -S
```

List only the gene names:

```bash
grep -v "^#" T0_vfdb.res \
    | awk -F'\t' '$5 >= 90 && $6 >= 90' \
    | cut -f1 \
    | cut -d'(' -f3 \
    | cut -d')' -f1
```

Count the organisms the reference genes come from:

```bash
grep -v "^#" T0_vfdb.res \
    | awk -F'\t' '$5 >= 90 && $6 >= 90' \
    | cut -f1 \
    | grep -o '\[[^]]*\]$' \
    | sort | uniq -c | sort -nr
```

Do the same for T64.

> **Questions:**
> 8. How many virulence genes pass the filter in T0 and in T64?
> 9. From which organisms do most reference genes come? Do these organisms, or close relatives (same family), appear in the taxonomic profiles of this morning?
> 10. Look at the Template_Identity of these hits. Are the reads identical to the reference genes? What does this suggest about the organism in the coffee?
> 11. Look at the descriptions: flagella, chemotaxis, secretion systems... Are these functions found only in pathogens?

---

## Before the recap

Think about these questions, we will discuss them together:

> - Does finding a virulence gene mean that the coffee contains a pathogen? What else could explain it?
> - Does finding a resistance gene mean that the bacteria are resistant? What would you need to know more?
> - How did your answers depend on the thresholds you chose?
