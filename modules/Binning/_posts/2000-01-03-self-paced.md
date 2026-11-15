---
title: "Self-paced: who carries the resistance genes?"
---

# Self-paced exploration: who carries the resistance genes?

On Day 2, KMA found resistance genes in the T64 reads: tet(M), lnu(A), qnrB19 and qnrE1. Reads alone cannot tell us **which organism** carries these genes. Now that we have MAGs, we can try to answer this question.

In this exercise, you will search the MAGs and the assembly of the coffee fermentation for resistance genes, using **ABRicate** and the **ResFinder** database.

Work at your own pace, alone or in pairs. Write down your answers: we will discuss them in the recap.

> **Before you start:** if a T64 MAG carries tet(M), what would that tell you that the KMA result could not?

---

## ABRicate in a nutshell

ABRicate searches **assembled sequences** (contigs, MAGs, genomes) against a database of reference genes. For each gene it finds, it reports:
- on which contig the gene is, and where (start and end positions)
- how much of the reference gene is covered (**%COVERAGE**)
- how similar the sequence is to the reference gene (**%IDENTITY**)

By default, ABRicate only reports genes with at least 80% identity and 80% coverage.

ABRicate compares DNA sequences: it does not use protein similarity.

---

## Setup

```bash
conda activate /shared/team/conda/aliseponsero.mmb-dtp/abricate

DIR=/shared/team/2026_training/week5/day4_MAGs/Self_guided

mkdir -p $WSUSER/abricate
cd $WSUSER/abricate

ls $DIR
```

You have:

| File or folder | Content |
|---|---|
| `T0_bins/` | The bins of the T0 sample (ERR2231567) |
| `T64_bins/` | The bins of the T64 sample (ERR2231572) |
| `T64_contigs.fa` | Contigs of the T64 assembly |

---

## Part 1: Resistance genes in the MAGs

Here is the command for the T0 bins:

```bash
abricate --db resfinder $DIR/T0_bins/*.fa > T0_bins_resfinder.tsv
```

Look at the result:

```bash
column -t -s $'\t' T0_bins_resfinder.tsv | less -S
```

Press `q` to exit `less`.

Each line is a resistance gene found in a bin. If the file only contains the header line, ABRicate found no resistance gene.

**Your turn:** run ABRicate on the T64 bins, and save the result in `T64_bins_resfinder.tsv`.

**Q1.** How many resistance genes did ABRicate find in the T64 bins? Is this what you expected from the KMA results of Day 2?

---

## Part 2: Where did the resistance genes go?

The bins only contain part of the assembly: SemiBin2 does not put every contig in a bin. Let's search the contigs of the T64 assembly.

**Your turn:** run ABRicate on `$DIR/T64_contigs.fa`, and save the result in `T64_contigs_resfinder.tsv`.

**Q2.** How many resistance genes did ABRicate find in the contigs? Are tet(M), lnu(A), qnrB19 and qnrE1 among them?

**Q3.** Look at the columns `%COVERAGE` and `%IDENTITY`. Which genes are complete and nearly identical to the reference? Which ones are only similar to the reference?

Let's get the names of the contigs carrying a resistance gene:

```bash
tail -n +2 T64_contigs_resfinder.tsv | cut -f2 | sort -u > arg_contigs.txt
cat arg_contigs.txt
```

- `tail -n +2` removes the header line
- `cut -f2` keeps the second column (`SEQUENCE`, the contig name)
- `sort -u` sorts the names and removes duplicates

Now look at the header of these contigs in the assembly:

```bash
grep -w -F -f arg_contigs.txt $DIR/T64_contigs.fa
```

The header of each contig was written by the assembler (MEGAHIT). It gives the length of the contig (`len=`, in bp) and how deeply it is covered by reads (`multi=`): the higher the value, the more abundant the sequence in the sample.

**Q4.** How long are the contigs carrying a resistance gene? Compare with the length of a resistance gene (around 0.5 to 3 kb). Do these contigs contain much more than the resistance gene?

Is any of these contigs in a T64 bin?

```bash
grep -w -F -f arg_contigs.txt $DIR/T64_bins/*.fa
```

**Q5.** Are the contigs carrying a resistance gene in a bin? Why do you think the binning left them out? Think about what binning tools use to group contigs.

---

## Part 3: Back to Day 2

Compare the resistance genes found by KMA in the T64 reads on Day 2 (filtered at 90% identity and 90% coverage) with the genes found by ABRicate in the T64 contigs.

**Q6.** Which genes are found by both methods? Which genes are only found in the contigs?

**Q7.** Reads, contigs or MAGs: which approach would you use to answer each of these questions?
- Is there any tet(M) in this food?
- Which organism carries tet(M)?
- Is tet(M) on a plasmid or on the chromosome?

---

## Resources

- ABRicate: https://github.com/tseemann/abricate
- ResFinder: https://genepi.food.dtu.dk/resfinder
