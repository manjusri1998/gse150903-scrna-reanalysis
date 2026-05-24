# Data

This project uses the public GEO dataset
[GSE150903](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE150903):
single-cell RNA-seq from human choroid plexus organoids and telencephalon
organoids.

The raw and processed data files are intentionally not committed to GitHub
because transcriptomics matrices and AnnData files can be large.

To reproduce the recommended notebook, download the processed supplementary
matrix from GEO:

- `GSE150903_SCT_scaled_count_matrix.txt.gz`

Then unzip it and place the resulting file here:

```text
data/processed/GSE150903_SCT_scaled_count_matrix.txt
```

This matrix is already SCTransform-scaled and appears to correspond to the
paper's final post-QC dataset of 32,464 cells. It should be treated as a
processed analysis matrix, not as raw counts.

The methods-guided notebook may generate intermediate AnnData files in
`data/processed/`, such as:

- `methods_guided_reanalysis.h5ad`
- `methods_guided_leiden_marker_genes.csv`
- `methods_guided_auto_cluster_annotation.csv`

These generated files are also ignored by Git.
