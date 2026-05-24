# Methods-Guided Reproduction Plan

This document maps one focused part of the paper's supplementary scRNA-seq
methods onto a Python/Scanpy reanalysis plan for `GSE150903`.

The current notebook is scoped to broad cell identity, telencephalon-reference
vs ChP organoid comparison, mature ChP identification, and mature ChP
subsetting/subclustering. Gene enrichment analysis and comparison to external
human and mouse datasets are reserved for a follow-up notebook.

## What The Paper Reports

The single-cell RNA-seq experiment pooled two organoids per condition:

- H9 telencephalic organoids, day 55
- H1 choroid plexus organoids, day 27
- H1 choroid plexus organoids, day 46
- H1 choroid plexus organoids, day 53

Libraries were generated with 10X Genomics Chromium Single Cell 3 prime v3 and
processed with CellRanger Count 3.1.0 using STAR alignment to the GRCh38 human
reference genome.

The paper reports these analysis parameters:

- merge filtered feature-barcode matrices across samples
- remove cells with mitochondrial percentage greater than 30%
- remove likely doublets using `nCount_RNA`
- final high-quality dataset: 32,464 cells
- normalize, scale, and select variable features using Seurat v3 `SCTransform`
- regress out mitochondrial mapping percentage and cell cycle during
  normalization
- use PCA with ElbowPlot-guided selection of 4 principal components
- run Seurat `FindNeighbors`, `FindClusters`, and UMAP
- identify clusters using the top 10 differentially expressed genes and known
  marker genes
- subcluster the mature choroid plexus cluster using PCs 1-12

## Python Reproduction Strategy

The paper used Seurat/R. This project reproduces the downstream analysis in
Python with Scanpy, using the processed SCT-scaled matrix released on GEO. The
goal here is not to reproduce every analysis in the paper, but to reproduce the
cell-identity workflow leading to mature ChP identification.

| Paper method or parameter | Python/Scanpy implementation |
| --- | --- |
| CellRanger Count v3.1.0 with STAR/GRCh38 | documented as the upstream source of the GEO matrix; not rerun here |
| Seurat v3 object | `scanpy.AnnData` object |
| merged sample matrices | load GEO matrix, transpose to cells x genes, add sample labels |
| telencephalon organoid sample | use as the non-ChP reference/comparison sample |
| ChP organoid D27/D46/D53 samples | analyze as ChP developmental samples |
| mitochondrial percentage >30% removed | documented; exact re-filtering requires raw counts |
| likely doublets removed using `nCount_RNA` | documented as equivalent to `total_counts`; exact re-filtering requires raw counts |
| final dataset of 32,464 cells | checked against the processed GEO matrix dimensions |
| SCTransform normalization/scaling/variable features | use the released SCT-scaled matrix directly; do not re-normalize as raw counts |
| regress mitochondrial percentage and cell cycle | documented as part of the paper's Seurat workflow; exact rerun requires raw counts/Seurat |
| ElbowPlot selected 4 PCs | use `sc.pp.neighbors(..., n_pcs=4)` for main clustering |
| Seurat `FindNeighbors` | `sc.pp.neighbors` |
| Seurat `FindClusters` | `sc.tl.leiden` |
| UMAP | `sc.tl.umap` |
| top 10 differentially expressed genes | `sc.tl.rank_genes_groups(..., method="wilcoxon")` |
| known marker gene interpretation | Scanpy dotplots, UMAP feature plots, and marker-set scoring |
| mature ChP subclustering with PCs 1-12 | optional mature/ChP-like subset with `n_pcs=12` |

## Important Reproduction Boundary

GEO provides `GSE150903_SCT_scaled_count_matrix.tsv.gz` as the processed matrix.
That file is already SCTransform-derived, so the Python reproduction should not
treat it as raw UMI counts.

There are two possible reproduction levels:

1. **Downstream reproduction from the GEO processed SCT matrix**

   This is the practical portfolio path. It can reproduce PCA/UMAP-style
   structure, marker-gene plots, cluster annotation, and cell-type summaries from
   the matrix provided on GEO.

2. **Full raw-data reproduction**

   This would start from SRA FASTQ files or raw 10X matrices, rerun CellRanger,
   then reproduce the Seurat v3 SCTransform workflow. This is much heavier and
   requires more storage, compute time, and likely an R/Seurat environment.

## Python/Scanpy Translation

| Paper / Seurat step | Python / Scanpy equivalent |
| --- | --- |
| Merge sample matrices | read matrix, transpose to cells x genes, assign sample labels |
| Mitochondrial QC | calculate `pct_counts_mt` if raw counts are available |
| Doublet filtering by `nCount_RNA` | `adata.obs["total_counts"]`; approximate with high-total-count thresholds, or use Scrublet if raw counts are available |
| SCTransform | use the provided SCT matrix directly, or use Pearson residuals/Scanpy normalization only if starting from raw counts |
| ElbowPlot | inspect PCA variance ratio |
| 4 PCs | `sc.pp.neighbors(..., n_pcs=4)` |
| FindNeighbors | `sc.pp.neighbors` |
| FindClusters | `sc.tl.leiden` |
| UMAP | `sc.tl.umap` |
| Top 10 DE genes | `sc.tl.rank_genes_groups(..., n_genes=10)` |
| Known marker gene analysis | dotplots, matrixplots, UMAP feature plots |
| Mature ChP subclustering PCs 1-12 | subset mature ChP-like cells, rerun PCA/neighbors/Leiden/UMAP with `n_pcs=12` |

## Seurat Metadata Crosswalk

Some paper terms are Seurat object metadata columns rather than standalone
methods.

| Seurat/R term | Meaning | Scanpy/Python equivalent |
| --- | --- | --- |
| `nCount_RNA` | total UMI/RNA counts per cell | `adata.obs["total_counts"]` from `sc.pp.calculate_qc_metrics` |
| `nFeature_RNA` | number of detected genes per cell | `adata.obs["n_genes_by_counts"]` |
| mitochondrial percentage | percent counts assigned to mitochondrial genes | `adata.obs["pct_counts_mt"]` |
| cell cycle score | score derived from S-phase/G2M marker genes | `sc.tl.score_genes_cell_cycle` |
| Seurat identities / clusters | active cluster or cell labels | `adata.obs["leiden"]` or another annotation column |

For this repository's processed GEO matrix, `nCount_RNA`, `nFeature_RNA`, and
`pct_counts_mt` cannot be reproduced exactly unless raw counts are available.
The methods-guided notebook therefore uses the processed SCT matrix for
downstream PCA, clustering, UMAP, and marker analysis, while documenting where a
raw-count workflow would be required.

## Redo Notebook Outline

The new notebook should be more methods-aware than the first version:

1. Load the processed SCT matrix from GEO.
2. Confirm matrix dimensions match the paper-reported 32,464-cell dataset.
3. Assign sample metadata from cell-barcode prefixes.
4. State clearly that the input matrix is processed SCT data.
5. Run PCA and inspect variance ratio.
6. Build neighbors using 4 PCs to match the paper.
7. Run Leiden clustering and UMAP.
8. Compare sample composition by cluster.
9. Identify top marker genes per cluster.
10. Visualize marker sets from the paper.
11. Assign rule-assisted cell-type labels using marker scores.
12. Make final composition and marker-dotplot figures.
13. Optionally subcluster mature ChP-like cells using PCs 1-12.

## Follow-Up Notebook Scope

The next notebook should start from the mature ChP subset and focus on analyses
that go beyond the current reproduction:

- gene ontology or pathway enrichment for mature ChP subclusters
- marker interpretation for mitochondria-rich/dark, ciliated/light, and
  myoepithelial-like states
- comparison with external human developing brain or ChP datasets
- comparison with mouse ChP datasets after ortholog mapping

## What To Be Careful About

- Cluster IDs may not match the paper exactly because Seurat and Scanpy use
  different implementations and defaults.
- The GEO matrix is already processed, so QC thresholds based on raw counts and
  mitochondrial percentages may not be exactly reproducible from this file alone.
- A faithful full-pipeline reproduction would require raw 10X/SRA inputs and an
  R/Seurat workflow.
