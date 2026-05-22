# Data

This project uses the public GEO dataset
[GSE150903](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE150903):
single-cell RNA-seq from human choroid plexus organoids and telencephalon
organoids.

The raw and processed data files are intentionally not committed to GitHub
because transcriptomics matrices and AnnData files can be large.

To reproduce the notebook, download the processed supplementary count matrix
from GEO:

- `GSE150903_SCT_scaled_count_matrix.txt.gz`

Then unzip it and place the resulting file here:

```text
data/processed/GSE150903_SCT_scaled_count_matrix.txt
```

The notebook generates intermediate AnnData files in `data/processed/`, such as:

- `gse150903_raw_counts.h5ad`
- `gse150903_qc_filtered.h5ad`
- `gse150903_normalized.h5ad`
- `gse150903_clustered.h5ad`
- `gse150903_reviewed_annotated.h5ad`

These generated files are also ignored by Git.
