# HG002 Genomic Variant Calling Workflow

## Project Overview
This repository contains an end-to-end pipeline for identifying genetic variants using **PacBio HiFi** sequencing data. The project focuses on the well-characterized **HG002** reference sample (Ashkenazi son) from the Genome in a Bottle (GIAB) consortium. By comparing individual DNA sequences against the **GRCh38** reference genome, the pipeline detects SNPs and INDELs across chromosomes 1-22.

## Technology Stack
The architecture leverages three primary technologies for high-performance computing (HPC):
* **Nextflow (DSL2)**: Serves as the core workflow engine, managing the parallel execution of variant calling tools.
* **Singularity/Apptainer**: Ensures complete reproducibility by executing each bioinformatics tool within its own isolated container.
* **SLURM**: Acts as the cluster's job scheduler, handling resource allocation and job queues.

---

## Workflow Steps
The pipeline is divided into several modular stages:

1. **Alignment**: Mapping HiFi reads to the GRCh38 reference using **Minimap2**.
2. **Post-Processing**: Sorting and indexing the alignment files with **Samtools**.
3. **Variant Discovery**: Concurrent variant calling using two deep-learning engines:
    * **Clair3** (utilizing the hifi_revio model).
    * **DeepVariant (v1.6.1)**.
4. **Filtering**: Using **BCFtools** to isolate high-quality calls for chromosomes 1-22.
5. **Evaluation**: Benchmarking performance against the **NIST v4.2.1** truth set using **hap.py**.

---

## Usage Instructions

### 1. Setup
Clone the repository to your HPC environment:
```bash
git clone [https://github.com/javeria-butt/hg002-variant-calling-pipeline.git]
cd hg002-variant-calling-pipeline
```
### 2. Execute via SLURM
```
sbatch nextflow_pipeline.sh
```
### 3. Monitor Results
```
squeue -u your_username
tail -f logs/nextflow_*.log
```
### OR Run Directly via SLURM
```
sbatch pipeline.sh
```
## Benchmarking Results (Chr1-22)

Performance was evaluated against the **GIAB HG002 NISTv4.2.1** truth set using `hap.py` with the `vcfeval` engine, restricted to chromosomes 1-22.

**Note on recall:** A quarter subset of the full HG002 dataset was used. Low recall is expected as many genomic regions lacked sufficient read coverage to detect variants. At full coverage, both tools typically achieve over 99% recall. The precision values, however, reflect true tool accuracy.

### SNP Performance
| Metric | Clair3 | DeepVariant |
| :--- | :--- | :--- |
| Variants Called | 252,391 | 163,639 |
| True Positives | 167,468 | 101,556 |
| False Positives | 35,508 | 31,719 |
| **Precision** | **82.5%** | **76.2%** |
| **Recall** | **4.98%** | **3.02%** |
| F1 Score | 0.094 | 0.058 |

### INDEL Performance
| Metric | Clair3 | DeepVariant |
| :--- | :--- | :--- |
| Variants Called | 26,778 | 24,264 |
| True Positives | 11,148 | 10,792 |
| False Positives | 5,164 | 5,825 |
| **Precision** | **68.8%** | **64.9%** |
| **Recall** | **2.12%** | **2.05%** |
| F1 Score | 0.041 | 0.040 |

---

## Key Findings
* **Clair3** outperformed DeepVariant at this coverage level with higher recall and precision for SNPs.
* **DeepVariant** is more conservative — it called fewer variants but maintained competitive INDEL precision.
* **High Precision**: Both tools showed high precision, meaning the calls they made were mostly correct.
* **Recall Explanation**: Low recall is expected and fully explained by the quarter-subset input data.

---

## Repository Contents
| File | Description |
| :--- | :--- |
| `variant_calling.nf` | Nextflow workflow — runs Clair3 and DeepVariant in parallel via Singularity |
| `nextflow_pipeline.sh` | SLURM script — submits the Nextflow pipeline to the HPC cluster |
| `pipeline.sh` | SLURM script — runs the full pipeline directly via Apptainer |
| `nextflow.config` | Nextflow config — sets up Singularity container binding and run options |
| `deepvariant.nf` | Standalone Nextflow script for DeepVariant only |

---

## Tools and Containers
| Tool | Version | Container | Purpose |
| :--- | :--- | :--- | :--- |
| **Minimap2** | latest | `docker://staphb/minimap2` | Read alignment |
| **Samtools** | latest | `docker://staphb/samtools` | BAM processing |
| **Clair3** | latest | `docker://hkubal/clair3` | Variant calling via Nextflow |
| **DeepVariant** | 1.6.1 | `docker://google/deepvariant:1.6.1` | Variant calling via Nextflow |
| **BCFtools** | latest | `docker://staphb/bcftools` | VCF filtering (chr1-22) |
| **hap.py** | latest | `docker://pkrusche/hap.py` | Benchmarking vs GIAB |

---

## Data Summary
| Parameter | Value |
| :--- | :--- |
| **Sample** | HG002 (Ashkenazi son, GIAB reference sample) |
| **Sequencing** | PacBio HiFi (Revio chemistry) |
| **Input** | 25% subset (~502MB FASTQ) |
| **Reference** | GRCh38 (hg38) |
| **Truth set** | GIAB NISTv4.2.1 (chr1-22, GRCh38) |


