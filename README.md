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
git clone [https://github.com/javeria-butt/hg002-variant-calling-pipeline.git](https://github.com/YOUR_USERNAME/hg002-variant-calling-pipeline.git)
cd hg002-variant-calling-pipeline
git clone [https://github.com/YOUR_USERNAME/hg002-variant-calling-pipeline.git](https://github.com/YOUR_USERNAME/hg002-variant-calling-pipeline.git)
cd hg002-variant-calling-pipeline
