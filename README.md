# Nextflow Training: Bioinformatics Pipelines

Pipelines and exercises I built during the **3-day Nextflow training workshop at Northeastern University (November 4–6, 2026)**, run with Seqera and Nextflow community ambassadors.

The work covers writing Nextflow pipelines from scratch, through to two containerized bioinformatics workflows: **GATK germline variant calling** and **bulk RNA-seq preprocessing**.

## Repository contents

| Folder | What it contains |
|---|---|
| `hello-nextflow/` | Core Nextflow development: processes, channels, multi-step workflows, modules, and containers |
| `nf4-science/genomics/` | GATK variant calling pipeline: per-sample calling and joint genotyping of a family trio |
| `nf4-science/rnaseq/` | RNA-seq QC, trimming, and alignment pipeline, in single-end and paired-end versions with MultiQC reporting |

## Pipeline 1: Joint variant calling (GATK)

`nf4-science/genomics/genomics.nf`

```
samplesheet.csv
  → SAMTOOLS_INDEX         (per sample, in parallel)  → BAM + BAI pairs
  → GATK_HAPLOTYPECALLER   (per sample, GVCF mode)    → GVCFs
  → collect()
  → GATK_JOINTGENOTYPING   (GenomicsDBImport + GenotypeGVCFs) → cohort VCF
```

- Reads BAM file paths from a CSV samplesheet.
- Keeps each BAM and its index together as a tuple, so files can't be mismatched during parallel execution.
- Gathers all per-sample GVCFs into a single joint-genotyping step to produce one multi-sample VCF.
- Tested on a family trio (mother, father, son) on a region of human chromosome 20. The joint calls are consistent with Mendelian inheritance.

## Pipeline 2: RNA-seq preprocessing

`nf4-science/rnaseq/rnaseq.nf` (single-end) and `rnaseq_pe.nf` (paired-end)

```
samplesheet.csv
  ├→ FASTQC        (raw read QC)
  └→ TRIM_GALORE   (adapter trimming + post-trim QC)
        → HISAT2_ALIGN  (splice-aware alignment → BAM)
  all QC outputs → mix() → collect() → MULTIQC (one combined report)
```

- Processes 6 samples in parallel from a CSV samplesheet.
- Paired-end version keeps read 1 and read 2 together as tuples and uses `--paired` trimming and `-1`/`-2` alignment.
- Aggregates FastQC, Cutadapt, and HISAT2 metrics from every sample into a single MultiQC report.
- Alignment rates of about 92–94% on the test data.

## Nextflow concepts used

- **Processes and modules:** one tool per process, each in its own reusable module file
- **Channels and parallelism:** each samplesheet row becomes an independent parallel task
- **Operators:** `splitCsv`, `map`, `flatten`, `collect`, `mix`, `view`
- **Tuples:** keeping associated files together (BAM + index, paired reads)
- **Containers:** each process runs in its own Docker container for reproducibility
- **Configuration:** test profiles, parameters, and workflow-level output publishing
- **Resumability:** `-resume` to reuse completed tasks

## Tools

Nextflow · Docker · Samtools · GATK4 (HaplotypeCaller, GenomicsDBImport, GenotypeGVCFs) · FastQC · Trim Galore / Cutadapt · HISAT2 · MultiQC

## Running the pipelines

These pipelines were developed in the official [Nextflow training environment](https://training.nextflow.io) (GitHub Codespaces). The test data is **not included** in this repo, so to run them, use the data from the [nextflow-io/training](https://github.com/nextflow-io/training) repository.

```bash
# Genomics
cd nf4-science/genomics
nextflow run genomics.nf -profile test

# RNA-seq (single-end / paired-end)
cd nf4-science/rnaseq
nextflow run rnaseq.nf -profile test
nextflow run rnaseq_pe.nf -profile test_pe
```

Requires Nextflow and Docker.

## Acknowledgments

Built while following the official Nextflow training materials by Seqera ([training.nextflow.io](https://training.nextflow.io), licensed CC BY-NC-SA 4.0). Thanks to the workshop instructors Geraldine Van der Auwera (Seqera), Rima Zinjuwadia, and Lei Ma (Nextflow Ambassadors), and to the Northeastern MS Bioinformatics program for hosting.
