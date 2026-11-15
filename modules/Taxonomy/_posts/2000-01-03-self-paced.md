---
title: "Self-paced exploration: coffee fermentation"
---

# Self-paced exploration: profiling the coffee fermentation

In the guided hands-on, we profiled a mock community where we knew the answer. Now we look at **real samples**, where we don't.

We have profiled six samples of a coffee bean fermentation with three tools: **MetaPhlAn4**, **Kraken2** and **Bracken**. Your task is to explore these profiles with Pavian, follow how the community changes during fermentation, and find where the tools agree and where they disagree.

Work at your own pace, alone or in pairs. Parts 1 to 3 are the core of the exercise; the stretch questions are for those who finish early. Write down your answers: we will discuss them together in the debrief.

---

## The samples

| Sample | Timepoint | Fermentation stage |
|---|---|---|
| ERR2231567 | T0 | Start: diverse environmental microbiota |
| ERR2231568 | T8 | Early fermentation: pH dropping |
| ERR2231569 | T16 | End of standard fermentation |
| ERR2231570 | T24 | Extended fermentation begins |
| ERR2231571 | T36 | Mid-extended fermentation |
| ERR2231572 | T64 | Final sample point |

## The files

You have 18 files: one per sample and per tool.

| Tool | Files | What the percentages mean |
|---|---|---|
| MetaPhlAn4 | `T*_profiles_metaphlan.txt` | Relative abundance, **including** an estimate of the unclassified fraction |
| Kraken2 | `T*_profiles_kraken2.txt` | Percentage of **all reads**, including unclassified reads |
| Bracken | `T*_profiles_bracken.txt` | Percentage of the **classified reads kept by Bracken** (no unclassified) |

The Kraken2 database used is **Standard-8**, the same as in the guided hands-on.

### Download the files

1. Open the file browser on the notebook (top-left)
2. Navigate to `[PATH TO THE SELF-PACED FILES]`
3. Select the files, right-click and choose **Download**
4. Save them on your computer

### Load the files in Pavian

Launch Pavian in RStudio as in the guided hands-on:

```r
pavian::runApp(port=5000)
```

Upload the files **of one tool at a time** (for example the 6 MetaPhlAn4 files), so that you compare the samples within a tool.

**Useful views in Pavian:**
- **Sample**: one sample at a time, with the Sankey diagram of the taxonomy
- **Comparison**: all samples side by side in a table; you can choose the taxonomic rank (family, genus, species) and sort the columns

---

## Part 1: Follow the fermentation with MetaPhlAn4

Upload the 6 MetaPhlAn4 files.

**Q1. Unclassified reads.** What percentage of the community is unclassified at T0 and at T64? Does it go up or down during the fermentation? What could explain this?

> Hint: look for the `UNCLASSIFIED` line in the Comparison view, or open the files in a text editor.

**Q2. Dominant families.** Which is the most abundant family at T0? And at T64?

> Hint: in the Comparison view, choose the family rank and sort by the T0 column, then by the T64 column.

**Q3. Lactobacillaceae.** Fill in the MetaPhlAn4 line of the table below. How does Lactobacillaceae change during the fermentation?

| Lactobacillaceae (%) | T0 | T8 | T16 | T24 | T36 | T64 |
|---|---|---|---|---|---|---|
| MetaPhlAn4 | | | | | | |
| Kraken2 | | | | | | |
| Bracken | | | | | | |

**Q4. Top species.** Which species is the most abundant from T0 to T36? At which timepoint does it reach its maximum? Which species is the most abundant at T64?

> Hint: switch to the species rank in the Comparison view.

---

## Part 2: The same questions with Kraken2 and Bracken

**Q5. Unclassified reads with Kraken2.** Upload the 6 Kraken2 files. What percentage of reads is unclassified at T0 and at T64? Compare with MetaPhlAn4 (Q1).

> Hint: the first line of each Kraken2 report is `unclassified`.

**Q6. Lactobacillaceae with Kraken2 and Bracken.** Fill in the Kraken2 line of the table in Q3. Then upload the 6 Bracken files and fill in the Bracken line.

**Q7.** For the same samples, the three tools give different percentages for Lactobacillaceae. Do they agree on the **trend**? Why are the **values** different?

> Hint: go back to the table "What the percentages mean" at the top of this page.

**Q8.** Answer Q2 and Q4 again with the Bracken profiles. Do you get the same answers as with MetaPhlAn4?

---

## Part 3: Where do the tools disagree?

**Q9. Human reads.** Search for *Homo sapiens* in the Bracken profiles. What is its abundance at T0? Is it in the MetaPhlAn4 profiles? Why would there be human DNA in a coffee fermentation, and why does only one tool report it?

> Hint: think about what each database contains. Which step of the analysis should have removed these reads?

**Q10. Yeasts.** Look at the Eukaryota in the MetaPhlAn4 profiles. Which organism is detected? Is it in the Bracken profiles? Why?

> Hint: the Standard-8 database contains RefSeq archaea, bacteria, viruses, plasmids and human.

**Q11. The final community.** Which genus is the most abundant at T64 according to MetaPhlAn4? And according to Bracken? Look for *Paucilactobacillus vaccinostercus* in the Kraken2 and Bracken profiles: what do you find?

**Q12. How many species?** How many species are detected in the T0 sample by MetaPhlAn4? And by Bracken? How many species have at least 0.1% abundance in Bracken? What does this tell you about the low-abundance species reported by Kraken2 and Bracken?

> Hint: in the guided hands-on, how many false positives did we find in a community of 5 genomes?

---

## Stretch questions

**S1.** When does *Lactiplantibacillus* first reach more than 1% abundance with each tool?

**S2.** Compare the *Tatumella* and *Gluconobacter* species reported by MetaPhlAn4 and by Bracken. Do the tools agree at genus level? And at species level?

**S3.** Some MetaPhlAn4 species have names like `Frateuria_SGB93747`. What does "SGB" mean, and why don't these species have a proper name?

**S4.** MetaPhlAn4 percentages include the unclassified fraction. Recalculate the Lactobacillaceae abundance at T64 as a percentage of the **classified** part only. Is it closer to the Bracken value?

> Hint: divide the Lactobacillaceae percentage by (100 - unclassified percentage), then multiply by 100.

---

## Before the debrief

Think about this question, we will discuss it together:

> **If you had to describe the microbial community at the end of the fermentation (T64), which profile would you trust, and why? What would you need to be more confident?**
