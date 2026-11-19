---
title: "From FASTQ to Integrated Variant Data: A Reproducible WES/WGS Workflow"
date: 2026-11-18
permalink: /articles/fastq-to-integrated-variant-data/
excerpt: "A step-by-step WES/WGS workflow from raw reads to an analysis-ready variant table, and the checks that make data from different sources safe to combine."
tags:
  - bioinformatics
  - NGS
  - reproducibility
  - Genomics
  - Pipeline
---

## Why reproducibility comes first

Sequencing data rarely arrive from a single source. A typical project mixes whole-exome (WES) and whole-genome (WGS) runs, samples processed at different times, and public reference cohorts aligned years apart. Each source carries its own reference build, file conventions and technical quirks.

A variant table is only as trustworthy as the path that produced it. If that path cannot be rerun, from raw FASTQ to the final table, combining sources becomes guesswork. This article walks through the workflow I use, step by step, and focuses on the part that is easiest to get wrong: integrating data from different origins.

## The workflow, step by step

The pipeline runs in six stages, from raw reads to an analysis-ready table.

| Step | Tools | What it does |
| --- | --- | --- |
| 1. Read quality control | [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/), [MultiQC](https://docs.seqera.io/multiqc) | Checks base quality, adapter content and duplication per sample; MultiQC merges every report into one view |
| 2. Alignment | [DRAGEN](https://help.dragen.illumina.com/) | Maps reads to the reference genome with hardware-accelerated alignment |
| 3. BAM processing | [samtools](https://www.htslib.org/), [GATK](https://gatk.broadinstitute.org/) | Sorts, indexes and computes coverage; marks duplicates and recalibrates base quality scores |
| 4. Variant calling | [GATK](https://gatk.broadinstitute.org/) HaplotypeCaller, Mutect2 or [DRAGEN](https://help.dragen.illumina.com/) | HaplotypeCaller for germline variants, Mutect2 for somatic variants in tumor samples, or the DRAGEN callers end to end |
| 5. Annotation | [ANNOVAR](https://annovar.openbioinformatics.org/) | Adds gene, functional consequence and population-frequency annotations |
| 6. Analysis | R ([Bioconductor](https://bioconductor.org/), [maftools](https://bioconductor.org/packages/maftools/)) and Python ([pandas](https://pandas.pydata.org/)) | Filtering, summaries and visualization |

Two checkpoints matter more than any single tool. After alignment, coverage and duplication metrics confirm that WES and WGS samples are comparable before calling. After annotation, a quick look at variant counts per sample catches outliers before they reach the analysis.

## Integrating data from different sources

Most integration problems are not biological; they are bookkeeping. Four checks prevent the most common ones.

1. **Harmonize the reference build.** Public cohorts may be aligned to GRCh37 while new data use GRCh38. Mixing coordinates silently mislabels variants, so every dataset is either realigned or lifted over to one build, and the build is recorded in the file metadata.
2. **Unify formats.** VCF files from different callers differ in fields and conventions. Converting annotated variants to a common table, such as MAF through maftools, gives R and Python the same columns for every source.
3. **Join metadata deliberately.** Sample identifiers, sequencing type and clinical fields often live in separate spreadsheets. Keeping one curated sample sheet, with a single ID per sample, makes every join explicit and auditable.
4. **Account for WES versus WGS.** Exomes cover only targeted regions and differ in depth from genomes. Restricting comparisons to shared target regions, and recording the assay as a covariate, keeps technical differences from looking like biological ones.

The goal is a single table in which every row can be traced back to its FASTQ, its reference build and its pipeline version.

## Making it reproducible

Three habits turn a set of commands into a workflow someone else can rerun.

- **A workflow manager.** Snakemake or Nextflow encodes each step, its inputs and its outputs, so the pipeline reruns only what changed and documents itself.
- **Pinned environments.** A conda environment file or a container fixes every tool version. When a result changes, the cause is the data or a deliberate update, not a silent upgrade.
- **Version control and configuration.** Code lives in Git; paths, reference files and parameters live in a single configuration file. Rerunning on new data means editing the configuration, not the code.

## Try it yourself

A pipeline is easiest to understand by running it. The companion demo repository applies this workflow to a small, public reference dataset that runs on a laptop, with a step-by-step README and an example of joining the variant table with a sample sheet. It is a technical demonstration only: it draws no biological conclusions.

Demo repository: [github.com/Alma-PC/WES_WGS_Pipeline_demo](https://github.com/Alma-PC/WES_WGS_Pipeline_demo)

## References

Andrews, S. (2010). *FastQC: A quality control tool for high throughput sequence data* [Computer software]. Babraham Bioinformatics. https://www.bioinformatics.babraham.ac.uk/projects/fastqc/

Behera, S., Catreux, S., Rossi, M., et al. (2024). Comprehensive genome analysis and variant detection at scale using DRAGEN. *Nature Biotechnology, 43*(7), 1177–1191. https://doi.org/10.1038/s41587-024-02382-1

Benjamin, D., Sato, T., Cibulskis, K., et al. (2019). Calling somatic SNVs and indels with Mutect2. *bioRxiv*. https://doi.org/10.1101/861054

Danecek, P., Bonfield, J. K., et al. (2021). Twelve years of SAMtools and BCFtools. *GigaScience, 10*(2), giab008. https://doi.org/10.1093/gigascience/giab008

Ewels, P., Magnusson, M., Lundin, S., & Käller, M. (2016). MultiQC: Summarize analysis results for multiple tools and samples in a single report. *Bioinformatics, 32*(19), 3047–3048. https://doi.org/10.1093/bioinformatics/btw354

Huber, W., Carey, V. J., Gentleman, R., et al. (2015). Orchestrating high-throughput genomic analysis with Bioconductor. *Nature Methods, 12*(2), 115–121. https://doi.org/10.1038/nmeth.3252

Mayakonda, A., Lin, D.-C., Assenov, Y., et al. (2018). Maftools: Efficient and comprehensive analysis of somatic variants in cancer. *Genome Research, 28*(11), 1747–1756. https://doi.org/10.1101/gr.239244.118

McKenna, A., Hanna, M., Banks, E., et al. (2010). The Genome Analysis Toolkit: A MapReduce framework for analyzing next-generation DNA sequencing data. *Genome Research, 20*(9), 1297–1303. https://doi.org/10.1101/gr.107524.110

McKinney, W. (2010). Data structures for statistical computing in Python. In *Proceedings of the 9th Python in Science Conference* (pp. 56–61). https://doi.org/10.25080/Majora-92bf1922-00a

Wang, K., Li, M., & Hakonarson, H. (2010). ANNOVAR: Functional annotation of genetic variants from high-throughput sequencing data. *Nucleic Acids Research, 38*(16), e164. https://doi.org/10.1093/nar/gkq603
