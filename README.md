
# Single-Cell RNA-Seq Pipeline — Colon Adenocarcinoma (N-of-1)

Overview
========

This repository provides a reproducible single-cell RNA-seq (scRNA-seq) analysis pipeline for target discovery in a single human colon adenocarcinoma sample (N-of-1). The notebook implements a complete workflow from data acquisition through clustering, annotation, and focused differential expression to nominate candidate therapeutic targets within intra-tumor subpopulations.

Why this pipeline
------------------

- Designed for deep intra-tumor heterogeneity analysis (single-sample focus).
- Reproducible, documented notebook with clearly separated preprocessing, clustering, and DE steps.

Table of contents
-----------------

- [Quick start](#quick-start)
- [Data](#data)
- [Pipeline overview](#pipeline-overview)
  - [Quality control](#quality-control)
  - [Normalization & feature selection](#normalization--feature-selection)
  - [Dimensionality reduction & clustering](#dimensionality-reduction--clustering)
  - [Annotation & target discovery](#annotation--target-discovery)
- [Outputs & examples](#outputs--examples)
- [Reproducibility notes](#reproducibility-notes)
- [Contributing & contact](#contributing--contact)

Quick start
===========

Prerequisites
-------------

- Python 3.10+
- Conda (recommended) or pip

Install (recommended via conda)

```bash
conda create -n sc-env python=3.10 -y
conda activate sc-env
conda install -c conda-forge scanpy leidenalg python-igraph -y
pip install cellxgene-census
```

Run the analysis
----------------

1. Open `cancer_pipeline.ipynb` in Jupyter or VS Code.
2. Execute cells from top to bottom. Key configurable cells include the CELLxGENE data query, QC thresholds, and HVG count.

Data
====

- Source: CELLxGENE / tiledb-soma (data is streamed in the notebook).
- Default subset: first 5,000 matching cells (configurable).
- Important fields: `adata.obs` (metadata) and `adata.var['feature_name']` (gene symbols).

Pipeline overview
=================

Quality control
---------------

- Identify mitochondrial genes (`MT-` prefix) and compute QC metrics via `sc.pp.calculate_qc_metrics`.
- Recommended filters (editable): mitochondrial percent threshold (e.g. 15–20%), minimum/maximum gene counts per cell.

Normalization & feature selection
---------------------------------

- Normalize counts to a target sum (default 1e4) and log-transform using `sc.pp.normalize_total` and `sc.pp.log1p`.
- Identify Highly Variable Genes (HVGs) — default 2,000 — with `sc.pp.highly_variable_genes` and subset the matrix to these features.

Dimensionality reduction & clustering
-------------------------------------

- Run PCA (`sc.tl.pca`) and compute a neighborhood graph (`sc.pp.neighbors`).
- Compute UMAP (`sc.tl.umap`) for visualization.
- Cluster using Leiden (`sc.tl.leiden`) to populate `adata.obs['leiden']`.

Annotation & target discovery
-----------------------------

- Annotate clusters with canonical marker genes (mapping provided in the notebook).
- Subset malignant epithelial clusters and run `sc.tl.rank_genes_groups` (Wilcoxon) for focused differential-expression testing.

Outputs & examples
==================

- UMAP visualizations colored by cluster and annotated labels.
- Per-cluster marker ranking plots (`sc.pl.rank_genes_groups`).
- Volcano/MA plots for targeted DE results.
- Optionally save processed AnnData: `adata.write('colon_cancer_adata_symbols.h5ad')`.

Reproducibility notes
=====================

- The notebook saves a backup of original feature IDs in `adata.var['orig_id']` before replacing `adata.var_names` with gene symbols; you can revert if needed.
- If you perform downstream filtering or slicing of `adata`, re-run the PCA → neighbors → leiden sequence before plotting or recomputing markers.
- The notebook resolves duplicate gene names using a consistent delimiter (default `|`) to avoid collisions and make programmatic parsing safer.

Contributing & contact
======================

Contributions and issues are welcome. Please open an issue or submit a pull request. For quick questions, contact the repository owner or maintainer.

License
=======

Add a license file or include license text here (e.g., MIT, BSD).

