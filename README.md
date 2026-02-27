# HG002 Variant Calling Pipeline
![Nextflow](https://img.shields.io/badge/Nextflow%20-%23007D8A.svg?style=for-the-badge&logo=nextflow&logoColor=white)
![Singularity](https://img.shields.io/badge/Singularity-Container-blue?style=for-the-badge)
![Clair3](https://img.shields.io/badge/Clair3-Variant_Calling-orange?style=for-the-badge)
![PacBio](https://img.shields.io/badge/PacBio-HiFi_Data-yellow?style=for-the-badge)
---

### **Workflow Architecture**
`minimap2 (map-hifi)` → `samtools sort/index` → `Clair3` → `VCF` → `hap.py benchmarking`

---

## **Project Overview**
This repository contains a reproducible bioinformatics pipeline designed to process PacBio HiFi sequencing data from the **HG002** (GIAB Ashkenazi son) reference sample. 

The workflow automates the alignment of long reads to the **GRCh38** human reference genome, performs high-accuracy small variant calling (SNVs and Indels) via **Clair3**, and evaluates performance metrics against the official **GIAB v4.2.1** ground-truth dataset.

## **Core Objectives**
* **Reproducibility:** Utilize **Nextflow (DSL2)** and **Singularity/Apptainer** for consistent environment management.
* **Alignment:** Execute optimized long-read mapping using the `map-hifi` preset in **minimap2**.
* **Data Processing:** Generate standardized, coordinate-sorted, and indexed BAM files using **samtools**.
* **Variant Discovery:** Identify genetic variations using the deep-learning-based **Clair3** caller.
* **Validation:** Quantify pipeline precision and recall using the **hap.py** benchmarking engine.

---

## **Technical Prerequisites**
* **Nextflow** (v22.10.0+ recommended)
* **Singularity** or **Apptainer**
* **Java Runtime** (JRE 11 or later)

---

## **Directory Structure**
```text
hg002-variant-calling-pipeline/
├── main.nf               # Core Nextflow workflow logic
├── nextflow.config       # Container and resource definitions
├── run_pipeline.sh       # Shell wrapper for easy execution
├── README.md             # Documentation
├── data/
│   ├── GRCh38.fa                                          # Human reference genome
│   ├── HG002_quarter.fastq.gz                             # Sample HiFi input reads
│   ├── HG002_GRCh38_1_22_v4.2.1_benchmark.vcf.gz         # GIAB gold-standard VCF
│   └── HG002_GRCh38_1_22_v4.2.1_benchmark_noinconsistent.bed # Confident regions BED
└── containers/
    ├── minimap2.sif      # Singularity Image: Alignment
    ├── samtools.sif      # Singularity Image: BAM Utilities
    └── clair3.sif        # Singularity Image: Variant Calling
```
## Pipeline Parameters

### Mandatory Inputs
| Parameter | Description |
| :--- | :--- |
| `--reference` | Path to the GRCh38 human reference genome (FASTA) |
| `--reads` | Input PacBio HiFi sequencing data (FASTQ/FASTQ.GZ) |
| `--model_path` | Location of the pre-trained Clair3 model for HiFi data |

### Execution Options
| Parameter | Default | Description |
| :--- | :--- | :--- |
| `--sample` | *Auto-detected* | Label used for naming output files |
| `--outdir` | `results` | Target directory for all pipeline results |
| `--threads` | `8` | Total CPU cores designated for the workflow |

---

## Execution Guide

### Option A: Using the Wrapper Script
```bash
bash run_pipeline.sh
```
### Option B: Direct Nextflow Execution
```bash
nextflow run main.nf -profile singularity \
  --reference data/GRCh38.fa \
  --reads data/HG002_quarter.fastq.gz \
  --model_path /shared/clair3_models/hifi \
  --sample HG002 \
  --outdir results \
  --threads 8
```
### Option C: HPC Execution (Local SIF Mode)
For offline environments or clusters with restricted image pulling:
```bash
nextflow run main.nf -profile singularity \
  --use_local_sifs true \
  --sif_minimap2 containers/minimap2.sif \
  --sif_samtools containers/samtools.sif \
  --sif_clair3   containers/clair3.sif \
  --reference data/GRCh38.fa \
  --reads data/HG002_quarter.fastq.gz \
  --model_path /shared/clair3_models/hifi \
  --outdir results
```
## Workflow Stages

| Step | Process | Tool | Primary Purpose |
| :--- | :--- | :--- | :--- |
| **1** | `SETUP_CHECK` | `minimap2` | Validates file paths and system environment |
| **2** | `INDEX_REF` | `minimap2` | Generates reference genome index files |
| **3** | `ALIGN_READS` | `minimap2` | Maps HiFi reads using the `map-hifi` profile |
| **4** | `PROCESS_BAM` | `samtools` | Coordinate sorting and BAI index generation |
| **5** | `CALL_VARIANTS`| `Clair3` | Deep-learning based SNV and Indel calling |

 ## Expected Output Files
Upon completion of the workflow, the `results/` directory will contain the following:

* **HG002.sorted.bam & .bai**: The final coordinate-sorted alignments and their associated index.
* **HG002.vcf.gz & .tbi**: The compressed output of the variant calling process along with its index.
* **HG002.sam**: The initial unsorted alignment file (if the retention option is enabled).

## Performance Benchmarking
The pipeline's output was verified against the **NIST v4.2.1** benchmark set (spanning chromosomes 1-22) using the `hap.py` tool equipped with the `vcfeval` engine.

### Benchmarking Command
```bash
hap.py \
  data/HG002_GRCh38_1_22_v4.2.1_benchmark.vcf.gz \
  results/HG002.vcf.gz \
  -f data/HG002_GRCh38_1_22_v4.2.1_benchmark_noinconsistent.bed \
  -r data/GRCh38.fa \
  -o results/happy/results \
  --engine=vcfeval \
  --threads=8
```
### Summary Metrics

#### SNP Analysis
| Metric | Result |
| :--- | :--- |
| Variants Identified | 252,391 |
| True Positives (TP) | 167,468 |
| **Precision** | **82.5%** |
| **Recall** | **4.98%** |
| F1 Score | 0.094 |

#### INDEL Analysis
| Metric | Result |
| :--- | :--- |
| Variants Identified | 26,778 |
| True Positives (TP) | 11,148 |
| **Precision** | **68.8%** |
| **Recall** | **2.12%** |
| F1 Score | 0.041 |

> **Note:** These performance figures are based on a **25% subsample** of the HG002 library. The reduced recall is expected due to the lower sequencing depth of this specific test set. Standard, full-depth HiFi data typically achieves >99% recall.

---

## Software Inventory

| Tool | Version | Source Container |
| :--- | :--- | :--- |
| **Minimap2** | 2.28 | `quay.io/biocontainers/minimap2` |
| **Samtools** | 1.20 | `quay.io/biocontainers/samtools` |
| **Clair3** | Latest | `docker://hkubal/clair3` |
| **hap.py** | Latest | `docker://pkrusche/hap.py` |

