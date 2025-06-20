# ![nf-core/resolvinf](docs/images/nf-core-resolvinf_logo.png)

**A Nextflow implementation of ResolVI for spatial transcriptomics data analysis**.

[![GitHub Actions CI Status](https://github.com/nf-core/resolvinf/workflows/nf-core%20CI/badge.svg)](https://github.com/nf-core/resolvinf/actions)
[![GitHub Actions Linting Status](https://github.com/nf-core/resolvinf/workflows/nf-core%20linting/badge.svg)](https://github.com/nf-core/resolvinf/actions)
[![Nextflow](https://img.shields.io/badge/nextflow-%E2%89%A523.04.0-brightgreen.svg)](https://www.nextflow.io/)

[![install with bioconda](https://img.shields.io/badge/install%20with-bioconda-brightgreen.svg)](http://bioconda.github.io/)
[![Docker](https://img.shields.io/docker/automated/nfcore/resolvinf.svg)](https://hub.docker.com/r/nfcore/resolvinf)

## Introduction

**nf-core/resolvinf** is a bioinformatics pipeline that implements ResolVI (Resolution of Variational Inference) for spatial transcriptomics data analysis. ResolVI is a deep generative model designed to correct for technical noise in spatial transcriptomics data, particularly from platforms like 10x Xenium, while preserving biological signal and spatial context.

The pipeline takes spatialdata zarr stores (typically generated by the SOPA processing pipeline) as input and performs:

- **Data preprocessing** and quality control
- **ResolVI model training** with GPU acceleration support
- **Noise correction** and cell type prediction
- **Differential expression analysis** between cell types
- **Differential niche abundance analysis** for spatial context
- **Comprehensive visualization** including spatial plots, UMAP embeddings, and gene expression comparisons

The pipeline is built using [Nextflow](https://www.nextflow.io), a workflow tool to run tasks across multiple compute infrastructures in a very portable manner. It uses Docker/Singularity containers making installation trivial and results highly reproducible.

## Pipeline Summary

The pipeline consists of four main steps:

1. **RESOLVI_PREPROCESS**: Converts spatialdata zarr stores to AnnData format, sets up spatial coordinates, and prepares data for ResolVI
2. **RESOLVI_TRAIN**: Trains the ResolVI model, generates corrected counts, calculates noise components, and produces cell type predictions
3. **RESOLVI_ANALYZE**: Performs differential expression and differential niche abundance analyses
4. **RESOLVI_VISUALIZE**: Creates comprehensive visualizations including spatial plots, UMAP embeddings, and gene expression comparisons

## Quick Start

1. Install [`Nextflow`](https://nf-co.re/usage/installation) (`>=23.04.0`)

2. Install any of [`Docker`](https://docs.docker.com/engine/installation/), [`Singularity`](https://www.sylabs.io/guides/3.0/user-guide/) (you can follow [this tutorial](https://singularity-tutorial.github.io/01-installation/)), [`Podman`](https://podman.io/), [`Shifter`](https://nersc.gitlab.io/development/shifter/how-to-use/) or [`Charliecloud`](https://hpc.github.io/charliecloud/) for full pipeline reproducibility _(you can use [`Conda`](https://conda.io/miniconda.html) both to install Nextflow itself and also to manage software within pipelines. Please only use it within pipelines as a last resort; see [docs](https://nf-co.re/usage/configuration#basic-configuration-profiles))_.

3. Download the pipeline and test it on a minimal dataset with a single command:

   ```bash
   nextflow run nf-core/resolvinf -profile test,docker --outdir <OUTDIR>
   ```

   Note that some form of configuration will be needed so that Nextflow knows how to fetch the required software. This is usually done in the form of a config profile (`YOURPROFILE` in the example command above). You can chain multiple config profiles in a comma-separated string.

   > - The pipeline comes with config profiles called `docker`, `singularity`, `podman`, `shifter`, `charliecloud` and `conda` which instruct the pipeline to use the named tool for software management. For example, `-profile test,docker`.
   > - Please check [nf-core/configs](https://github.com/nf-core/configs#documentation) to see if a custom config file to run nf-core pipelines already exists for your Institute.
   > - If you are using `singularity`, please use the [`nf-core download`](https://nf-co.re/tools/#downloading-pipelines-for-offline-use) command to download images first, before running the pipeline. Set the [`NXF_SINGULARITY_CACHEDIR` or `singularity.cacheDir`](https://www.nextflow.io/docs/latest/singularity.html?#singularity-docker-hub) Nextflow options to be able to store and re-use the images from a central location for future pipeline runs.
   > - If you are using `conda`, it is highly recommended to use the [`NXF_CONDA_CACHEDIR` or `conda.cacheDir`](https://www.nextflow.io/docs/latest/conda.html) settings to store the environments in a central location for future pipeline runs.

4. Start running your own analysis!

   ```bash
   nextflow run nf-core/resolvinf --input samplesheet.csv --outdir <OUTDIR> -profile <docker/singularity/podman/shifter/charliecloud/conda/institute>
   ```

## Input Requirements

### Samplesheet

You will need to create a samplesheet with information about the samples you would like to analyse before running the pipeline. Use this parameter to specify its location. It has to be a comma-separated file with 2 columns, and a header row as shown in the examples below.

```csv
sample_id,zarr_path
sample1,/path/to/sample1_spatialdata.zarr
sample2,/path/to/sample2_spatialdata.zarr
sample3,/path/to/sample3_spatialdata.zarr
```

| Column      | Description                                                                                                                                                                            |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `sample_id` | Custom sample name. This entry will be identical for multiple sequencing libraries/runs from the same sample. Spaces in sample names are automatically converted to underscores (`_`). |
| `zarr_path` | Full path to spatialdata zarr store file. File should be generated by SOPA processing pipeline or compatible spatial transcriptomics preprocessing tool.                               |

An [example samplesheet](assets/samplesheet.csv) has been provided with the pipeline.

### Marker Genes

You can optionally provide a list of marker genes for focused analysis and visualization. This can be done in two ways:

1. **As a text file** (one gene per line):

   ```bash
   --marker_genes /path/to/marker_genes.txt
   ```

2. **As a comma-separated list in your config file**:
   ```nextflow
   params {
       marker_genes = ['EPCAM', 'CD3D', 'CD68', 'COL1A1', 'PECAM1']
   }
   ```

If no marker genes are specified, the pipeline will analyze all genes present in the dataset.

## Key Parameters

### Core ResolVI Parameters

- `--max_epochs`: Maximum number of training epochs for ResolVI model (default: 100)
- `--num_samples`: Number of posterior samples for uncertainty quantification (default: 20)
- `--num_gpus`: Number of GPUs to use for training (default: auto-detect, 0 for CPU-only, -1 for all available)

### Resource Parameters

- `--max_memory`: Maximum memory allocation (default: '128.GB')
- `--max_cpus`: Maximum CPU cores (default: 16)
- `--max_time`: Maximum runtime per job (default: '240.h')

## Output

The pipeline generates several types of output organized in the following directory structure:

```
results/
├── preprocessing/           # Preprocessed AnnData files
├── training/               # Trained ResolVI models and processed data
├── analysis/               # Final analyzed AnnData with all results
├── differential_expression/ # DE analysis results and plots
├── differential_abundance/  # Spatial niche analysis results
├── plots/                  # All visualization outputs
├── multiqc/               # MultiQC report
└── pipeline_info/         # Pipeline execution information
```

### Key Output Files

- **Trained Models**: `training/resolvi_model/` - Saved ResolVI models for each sample
- **Final Data**: `analysis/*_analyzed.h5ad` - AnnData files with corrected counts, predictions, and noise components
- **Spatial Plots**: `plots/spatial_*.png` - Spatial distribution plots for cell types and noise components
- **UMAP Plots**: `plots/umap_*.png` - UMAP embeddings showing cell type predictions vs. annotations
- **Gene Expression**: `plots/gene_*_comparison.png` - Before/after correction comparisons for marker genes
- **DE Results**: `differential_expression/de_*.csv` - Differential expression analysis results
- **DA Results**: `differential_abundance/da_*.csv` - Differential niche abundance analysis results

## GPU Support

The pipeline supports GPU acceleration for ResolVI model training, which can significantly reduce training time:

- **Auto-detection**: Leave `--num_gpus` unspecified for automatic GPU detection
- **Specific GPU count**: Use `--num_gpus N` to use N GPUs
- **All available GPUs**: Use `--num_gpus -1`
- **CPU-only**: Use `--num_gpus 0`

Ensure your execution environment has appropriate GPU drivers and CUDA support when using GPU acceleration.

## Documentation

The nf-core/resolvinf pipeline comes with documentation about the pipeline:

1. [Installation](https://nf-co.re/usage/installation)
2. Pipeline configuration
   - [Local installation](https://nf-co.re/usage/local_installation)
   - [Adding your own system config](https://nf-co.re/usage/adding_own_config)
3. [Running the pipeline](docs/usage.md)
4. [Output and how to interpret the results](docs/output.md)
5. [Troubleshooting](https://nf-co.re/usage/troubleshooting)

## ResolVI Method

ResolVI (Resolution of Variational Inference) is a deep generative model specifically designed for spatial transcriptomics data that:

1. **Corrects technical noise** while preserving biological signal
2. **Decomposes noise sources** into true signal, diffusion, and background components
3. **Predicts cell types** using semi-supervised learning
4. **Maintains spatial context** for downstream spatial analysis

The method is particularly effective for high-resolution spatial transcriptomics platforms like 10x Xenium, where technical noise can significantly impact downstream analysis.

### Key Features

- **Noise decomposition**: Separates true biological signal from technical artifacts
- **Semi-supervised learning**: Leverages both labeled and unlabeled cells for improved predictions
- **Spatial awareness**: Incorporates spatial information in the modeling process
- **Uncertainty quantification**: Provides confidence estimates for predictions and corrections

## Credits

nf-core/resolvinf was originally written by Christopher Tastad.

We thank the following people for their extensive assistance in the development of this pipeline:

- The ResolVI development team for the original method implementation
- The nf-core community for framework and best practices
- The SOPA development team for spatialdata preprocessing tools

## Contributions and Support

If you would like to contribute to this pipeline, please see the [contributing guidelines](.github/CONTRIBUTING.md).

For further information or help, don't hesitate to get in touch on the [Slack `#resolvinf` channel](https://nfcore.slack.com/channels/resolvinf) (you can join with [this invite](https://nf-co.re/join/slack)).

## Citations

If you use nf-core/resolvinf for your analysis, please cite:

### Pipeline

<!-- TODO: Add DOI after first release -->
<!-- > **nf-core/resolvinf: A Nextflow implementation of ResolVI for spatial transcriptomics data analysis**
>
> Christopher Tastad et al.
>
> _bioRxiv_ 2024. doi: [10.1101/XXXXXX](https://doi.org/10.1101/XXXXXX) -->

### ResolVI Method

> **ResolVI: Resolution of Variational Inference for spatial transcriptomics**
>
> [Please cite the original ResolVI paper when available]

### nf-core Framework

> **The nf-core framework for community-curated bioinformatics pipelines.**
>
> Philip Ewels, Alexander Peltzer, Sven Fillinger, Harshil Patel, Johannes Alneberg, Andreas Wilm, Maxime Ulysse Garcia, Paolo Di Tommaso & Sven Nahnsen.
>
> _Nat Biotechnol._ 2020 Feb 13. doi: [10.1038/s41587-020-0439-x](https://dx.doi.org/10.1038/s41587-020-0439-x).

### Software Dependencies

An extensive list of references for the tools used by the pipeline can be found in the [`CITATIONS.md`](CITATIONS.md) file.

You can cite the `nf-core` publication as follows:

> **The nf-core framework for community-curated bioinformatics pipelines.**
>
> Philip Ewels, Alexander Peltzer, Sven Fillinger, Harshil Patel, Johannes Alneberg, Andreas Wilm, Maxime Ulysse Garcia, Paolo Di Tommaso & Sven Nahnsen.
>
> _Nat Biotechnol._ 2020 Feb 13. doi: [10.1038/s41587-020-0439-x](https://dx.doi.org/10.1038/s41587-020-0439-x).
