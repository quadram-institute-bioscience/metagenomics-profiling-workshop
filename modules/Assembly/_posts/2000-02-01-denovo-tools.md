---
title: Assembly tools for metagenomics
---


## Tools: overview

There are several (but not many) tools available for metagenomic assembly. The choice of tool often depends on the type of sequencing data (short reads vs. long reads), computational resources, and specific project requirements. 

For a benchmark, see [Goussarov et al.](https://www.microbiologyresearch.org/content/journal/micro/10.1099/mic.0.001469), or [Zhang et al.](https://academic.oup.com/bib/article/24/2/bbad087/7077274#398990342).

> 💡 **TL;DR** For Illumina dataset we usually recommend [MegaHit](https://github.com/voutcn/megahit) for its realiability. An interesting tool, especially for training, is [Minia](https://github.com/GATB/minia), that focuses on low memory usage.

Below are some commonly used assemblers for metagenomic data:

### Illumina (Short Reads)

- **MEGAHIT**  
  [MEGAHIT](https://github.com/voutcn/megahit) is a fast, memory-efficient assembler designed for large and complex metagenomic datasets. Its multi-k-mer strategy enables accurate assembly from Illumina short reads, making it a popular default choice in many workflows.
- **metaSPAdes**  
  A variant of the well-known [SPAdes assembler](https://github.com/ablab/spades), metaSPAdes incorporates advanced error correction and assembly graph refinement for high-quality metagenome assemblies. It performs particularly well with datasets that exhibit highly variable coverage. Requires more memory, and is prone to failing due to resource limits on large datasets.
- **Minia**
  [Minia](https://github.com/GATB/minia) is a lightweight, memory-efficient assembler that uses a succinct de Bruijn graph representation. It is suitable for assembling large metagenomic datasets on machines with limited RAM, though it may produce more fragmented assemblies compared to other tools.

### Long Reads (Oxford Nanopore, PacBio)

- **metaFlye**  
  metaFlye is specifically optimized for metagenomic long-read data (Nanopore, PacBio), showing strong performance in assembling contiguous genomes from complex communities.
- **metaMDBG**  
  [metaMDBG](https://github.com/GaetanBenoitDev/metaMDBG) stands out for memory efficiency and fast runtime on long-read datasets, offering near-comparable quality to hifiasm-meta in recent studies. Since publication, it has been improved for ONT reads too.


## Installing the tools

Most of these tools can be installed via [Bioconda](https://bioconda.github.io/). For example, to install MEGAHIT using Conda, you can run:

```bash
mamba create -n metadenovo -c conda-forge -c bioconda \
   megahit minia metamdbg flye
```

## Trying de novo assembly of Illumina reads

We can use a *subsampled* dataset to test Minia.
To make our command a bit more generic, we will write them using Bash variable `R1` and `R2` to point to the input files.

For example, you can set the variables like this:

```bash
# Set the absolute paths to your subsampled FASTQ files
export R1=/path/to/ERR2231569_1_subsample.fastq.gz
export R2=/path/to/ERR2231569_2_subsample.fastq.gz
```

### Using Minia

```bash
export R1=
minia \
  -in "$R1" \
  -out minia-assembly/T16 \
  -kmer-size 41 \
  -abundance-min 3 \
  -max-memory 24000 \
  -nb-cores 8
```

* `-in`: Input file (for paired-end data, provide both files separated by a comma)
* `-out`: Output "basename" (directory and prefix for output files)
* `-kmer-size`: Size of k-mers to use for assembly (common values are 21, 31, 41)
* `-abundance-min`: Minimum k-mer abundance to consider (helps filter out errors)
* `-max-memory`: Maximum memory to use (in MB)
* `-nb-cores`: Number of CPU cores to use   


You will find two fasta files in the output directory: `minia-assembly/T16.contigs.fa` and `minia-assembly/T16.unitigs.fa`.

1. **Unitigs** are the smallest, unambiguous paths in the de Bruijn graph where the sequence is forced with no branching.

2. **Contigs** are longer sequences formed by merging unitigs after cleaning the graph to remove bubbles, tips, and ambiguities.

#### Minia contig names

Fields:

* `>0`, `>1`, `>2` - Contig identifier
* `LN:i:487` - Length in base pairs (integer)
* `KC:i:2566` - K-mer Count - total number of k-mers in this contig (integer)
* `km:f:5.741` - K-mer mean abundance - average coverage/depth (float)
* `L:+:6051:+` - Links to other contigs in the assembly graph, format: "L:orientation_this:target_contig:orientation_target"
+ or - indicates forward/reverse strand orientation


### Using MEGAHIT

Megahit syntax is similar, but it supports paired end reads via separate arguments. The output directory will contain all the files produced.

```bash
megahit -1 $R1 -2 $R2 -o megahit-assembly/T16/ -t 8
```

* Output contigs will be in `megahit-assembly/T16/final.contigs.fa`
* The name will contain information about the coverage as `multi=....` and the connectivity of a contig in the assembly graph. `flag=1` means the contig is standalone, `flag=2` a looped path and `flag=0` for other contigs.

### Comparing the assemblies metrics

We can use [QUAST](http://quast.sourceforge.net/quast) to compare the assemblies.

```bash
# Generic syntax
quast -o $OUTDIR FASTA_1 FASTA_2 ...
```

The bare metrics can be obtained with SeqFu:

```bash
seqfu stats -ntb minia-dir/*.fa
```

## Renaming contigs

Each assembler use a different naming scheme for the contigs they produce. Some downstream tools (especially Anvi'o),
might be picky on which characters are allowed.

:warning: Check the output file to see what information the assembler encoded in the contig names.

It's safe to rename them, and using SeqFu we can also keep a conversion table:

```bash
# This will produce `rename-report.txt`: a two columns table 
seqfu cat --anvio $ORIGINAL_CONTIGS > renamed_contigs.fa
```

The `--anvio` is a shortcut for 
```bash
seqfu cat -p c_ -s -z --zero-pad 12 --report rename_report.txt  $ORIGINAL_CONTIGS > renamed_contigs.fa
```
Where:
* `-p "c_"` means use the string `c_` as the prefix of all names
* `-s` (or `--strip-comments`) means remove any comment
* `-z` (or `--strip-name`) means remove the original name
* `--zero-pad 12` means use leading zeroes against the progressive number (12 digits) 
