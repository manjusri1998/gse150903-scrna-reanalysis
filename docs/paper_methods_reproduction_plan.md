# Methods-Guided Reproduction Plan

This document maps the paper's supplementary methods onto a Python/Scanpy
reanalysis plan for `GSE150903`.

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

## Important Reproduction Boundary

GEO provides `GSE150903_SCT_scaled_count_matrix.txt.gz` as the processed matrix.
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
| Doublet filtering by `nCount_RNA` | approximate with total-count thresholds or use Scrublet if raw counts are available |
| SCTransform | use the provided SCT matrix directly, or use Pearson residuals/Scanpy normalization only if starting from raw counts |
| ElbowPlot | inspect PCA variance ratio |
| 4 PCs | `sc.pp.neighbors(..., n_pcs=4)` |
| FindNeighbors | `sc.pp.neighbors` |
| FindClusters | `sc.tl.leiden` |
| UMAP | `sc.tl.umap` |
| Top 10 DE genes | `sc.tl.rank_genes_groups(..., n_genes=10)` |
| Known marker gene analysis | dotplots, matrixplots, UMAP feature plots |
| Mature ChP subclustering PCs 1-12 | subset mature ChP-like cells, rerun PCA/neighbors/Leiden/UMAP with `n_pcs=12` |

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

## What To Be Careful About

- Cluster IDs may not match the paper exactly because Seurat and Scanpy use
  different implementations and defaults.
- The GEO matrix is already processed, so QC thresholds based on raw counts and
  mitochondrial percentages may not be exactly reproducible from this file alone.
- A faithful full-pipeline reproduction would require raw 10X/SRA inputs and an
  R/Seurat workflow.
