# DNBseq European Roe Deer Genetic Diversity Analysis

A comprehensive bioinformatics pipeline for processing DNBseq whole-genome sequencing (WGS) data from historical and modern European roe deer (*Capreolus capreolus*) samples to assess genomic and genetic diversity changes. This repository documents the analysis workflow from raw sequencing reads through diversity assessment using ANGSD.

## Project Overview

This study investigates whether European roe deer (*Capreolus capreolus*) populations have experienced loss of genetic and adaptive diversity over the past 150 years. By comparing historical museum specimens with modern populations, we assess changes in nucleotide diversity, allele frequencies, and potential indicators of reduced adaptive potential.

## Research Questions

- How has genomic diversity (π) changed between historical and modern roe deer populations?
- Are there signatures of reduced adaptive potential in modern populations?
- Do allele frequency patterns suggest demographic changes or loss of rare beneficial alleles?

## Pipeline Overview

```
Raw Fastq → Quality Control → Trimming → Mapping → Variant Calling → Diversity Assessment
```

### Step 1: Quality Control (FastQC)
- **Tool:** FastQC v0.12.1
- **Outputs:** 
  - `1_1_fastqc.html`, `1_1_fastqc.zip`
  - `1_2_fastqc.html`, `1_2_fastqc.zip`
- **Key Metrics:** Read quality, adapter contamination, GC content analysis
- **Script:** `fastqccc1.sh`

### Step 2: Adapter Trimming & Quality Filtering (fastp)
- **Tool:** fastp v0.23.1
- **Parameters:** Quality threshold (default), adapter removal, length filtering
- **Outputs:**
  - `1_1_trimmed.fq.gz`, `1_2_trimmed.fq.gz` (cleaned reads)
  - `fastp_report.html`, `fastp_report.json` (detailed QC report)
- **Script:** `fastp1.sh`
- **Log Files:** `fastp_reh1_11643351.out`, `.err`

### Step 3: Reference Genome Indexing (Bowtie2)
- **Tool:** Bowtie2
- **Reference:** *Capreolus capreolus* genome (GCA_951849835.1_ETH_CapCap1)
- **Outputs:** Indexed reference genome
- **Script:** `indexref.sh`
- **Log Files:** `bowtie2_index_11643484.out`, `.err`

### Step 4: Sequence Mapping (Bowtie2)
- **Tool:** Bowtie2 v2.4.2
- **Parameters:** Paired-end mode, quality cutoffs (minQ=20, minMapQ=30)
- **Outputs:**
  - `sample1_sorted.bam` (aligned reads, sorted BAM format)
  - `sample1_sorted.bam.bai` (BAM index)
  - `bowtie2_mapping.log` (mapping statistics)
- **Script:** `mapping.sh`
- **Log Files:** `bowtie2_reh1_11644265.out`, `.err`

### Step 5: Genetic Diversity Assessment (ANGSD)
- **Tool:** ANGSD v0.933 (Analysis of Next Generation Sequencing Data)
- **Parameters:**
  - `-doGL 2` (SAMtools genotype likelihoods)
  - `-doSaf 1` (Site Allele Frequency)
  - `-doThetas 1` (Theta/diversity statistics)
  - `-doMaf 1` (Minor allele frequency)
  - `-minMaf 0.05` (5% minimum allele frequency filter)
  - `-minQ 20` (base quality threshold)
  - `-minMapQ 30` (mapping quality threshold)
- **Outputs:**
  - `reh1.saf.idx` (Site Allele Frequency spectrum)
  - `reh1.sfs` (Site Frequency Spectrum)
  - `reh1.theta.out` (Theta statistics: π, θW, Tajima's D per position)
- **Script:** `ANGSD.sh`

## Diversity Metrics

| Metric | Interpretation | Conservation Relevance |
|--------|-----------------|------------------------|
| **π (nucleotide diversity)** | Average genetic diversity across the genome | ↓ π indicates loss of genetic variation |
| **θW (Watterson's theta)** | Number of segregating sites | Sensitive to rare alleles |
| **Tajima's D** | Deviation from neutral evolution | Negative D suggests loss of rare alleles; positive D suggests balancing selection |

## File Organization

```
.
├── README.md                          # This file
├── fastqccc1.sh                       # FastQC quality control script
├── fastp1.sh                          # Adapter trimming script
├── indexref.sh                        # Reference genome indexing
├── mapping.sh                         # Read mapping script
├── ANGSD.sh                           # Genetic diversity analysis
│
├── QC_Output/
│   ├── 1_1_fastqc.html, 1_1_fastqc.zip
│   ├── 1_2_fastqc.html, 1_2_fastqc.zip
│   └── 1.base.png, 1.qual.png         # Quality plots
│
├── Trimmed_Reads/
│   ├── 1_1_trimmed.fq.gz
│   ├── 1_2_trimmed.fq.gz
│   ├── fastp_report.html, fastp_report.json
│   └── fastp_reh1_*.{out,err}
│
├── Mapping/
│   ├── sample1_sorted.bam
│   ├── sample1_sorted.bam.bai
│   ├── bowtie2_mapping.log
│   └── bowtie2_reh1_*.{out,err}
│
└── Diversity_Assessment/
    ├── reh1.saf.idx
    ├── reh1.sfs
    ├── reh1.theta.out
    └── ANGSD_reh1_*.{out,err}
```

## Key Results Summary

| Step | Tool | Input | Output | Status |
|------|------|-------|--------|--------|
| QC | FastQC | Raw FASTQ | HTML reports | ✓ Complete |
| Trimming | fastp | Raw FASTQ | Trimmed FASTQ | ✓ Complete |
| Indexing | Bowtie2 | Reference FASTA | .bt2 indices | ✓ Complete |
| Mapping | Bowtie2 | Trimmed FASTQ | BAM (sorted, indexed) | ✓ Complete |
| Genetic Diversity | ANGSD | BAM + Reference | SAF, SFS, Theta stats | ⏳ In Progress |

## HPC Cluster Setup

- **System:** GWDG (Gesellschaft für Wissenschaftliche Datenverarbeitung)
- **Job Scheduler:** SLURM
- **Environment:** Conda (bioconda, conda-forge channels)
- **Container Runtime:** Singularity/Apptainer (for reproducibility)

## Software Versions

- **FastQC:** v0.12.1
- **fastp:** v0.23.1
- **Bowtie2:** v2.4.2
- **ANGSD:** v0.933 (htslib 1.9)
- **Samtools:** v1.21

## Analysis Outputs

Expected outputs from ANGSD include:
- **π values:** Compared between historical and modern populations to assess diversity loss
- **Tajima's D values:** Negative values may indicate loss of rare alleles and reduced adaptive potential
- **Allele frequency spectra:** Distribution of rare vs. common alleles across populations
- **Genomic regions:** Identification of low-diversity regions potentially related to adaptation

## Next Steps

1. **Historical vs. Modern Comparison:** Quantitative comparison of π and Tajima's D between cohorts
2. **Adaptive Allele Assessment:** Analysis of allele frequency changes at candidate adaptation loci
3. **Demographic Inference:** Site Frequency Spectrum analysis to infer population history
4. **Conservation Implications:** Assessment of genetic erosion and adaptive potential
5. **Manuscript Preparation:** Results integration into conservation genetics publication

## Usage

To run the complete pipeline:

```bash
# Submit QC
sbatch fastqccc1.sh

# Submit trimming
sbatch fastp1.sh

# Submit indexing
sbatch indexref.sh

# Submit mapping
sbatch mapping.sh

# Submit diversity assessment
sbatch ANGSD.sh

# Monitor jobs
squeue -u u12937
```

## Author

Fritz Brockhaus  
PhD Candidate, Wildlife Genetics  
University of Göttingen  
European roe deer (*Capreolus capreolus*) genetic diversity & conservation genetics

## References

- Korneliussen, T. S., Albrechtsen, A., & Nielsen, R. (2014). ANGSD: Analysis of Next Generation Sequencing Data. *BMC Bioinformatics*, 15(1), 356.
- Li, H., Handsaker, B., et al. (2009). The Sequence Alignment/Map format and SAMtools. *Bioinformatics*, 25(16), 2078-2079.
- Langmead, B., & Salzberg, S. L. (2012). Fast gapped-read alignment with Bowtie 2. *Nature Methods*, 9(4), 357-359.
- Frankham, R., Bradshaw, C. J., & Brook, B. W. (2012). Genetics in conservation management: Revised recommendations for the redlist criteria. *Biological Conservation*, 170, 56-63.

---

**Last Updated:** November 5, 2025  
**Pipeline Version:** 1.0
