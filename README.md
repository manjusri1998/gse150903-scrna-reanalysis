# GSE150903 Single-Cell Transcriptomics Reanalysis

This is a computational biology portfolio project reanalyzing one focused part
of the public single-cell RNA-seq dataset
[GSE150903](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE150903) in
Python with Scanpy.

The dataset is associated with the study **"Choroid plexus organoids predict CNS
drug permeability and reveal human CSF proteins produced by specialized cell
types."**

## Important Data Note

The GEO file used here is:

```text
GSE150903_SCT_scaled_count_matrix.tsv.gz
```

This is a processed SCTransform-scaled expression matrix, not raw FASTQ data and
not a raw 10X UMI matrix. The paper reports that the authors already performed
QC, doublet filtering, SCTransform normalization, regression of mitochondrial
percentage and cell cycle, and produced a final dataset of 32,464 high-quality
cells.

Because the downloaded matrix already has 32,464 cells, the recommended analysis
in this repository treats it as the authors' processed dataset and focuses on
downstream reanalysis:

- sample metadata assignment
- PCA and UMAP
- neighborhood graph construction
- Leiden clustering as a Python analogue to Seurat clustering
- marker-gene analysis
- marker-set scoring
- cell-type annotation
- focused subsetting for biological questions

The original exploratory notebook is kept for transparency, but it used extra
QC and normalization steps that are not ideal for an already processed
SCT-scaled matrix.

## Analysis Scope

This repository does not attempt to reproduce the entire paper. The current
notebook focuses on a narrow scRNA-seq component:

- use the telencephalon organoid sample as a non-ChP reference/comparison
  sample
- distinguish telencephalon-like cells from ChP organoid cells
- identify mature ChP-like epithelial cells using marker genes
- prepare and subcluster the mature ChP population
- inspect marker patterns for mature ChP states such as ciliated/light,
  mitochondria-rich/dark, and myoepithelial-like cells

The next notebook will extend this work with gene enrichment analysis and
comparison to external human and mouse datasets.

## Recommended Notebook

Use this notebook for the current methods-aware Python reproduction:

```text
notebooks/02_methods_guided_reanalysis.ipynb
```

This notebook explicitly maps the paper's Seurat/R workflow to Python/Scanpy:

| Paper method or parameter | Python/Scanpy implementation |
| --- | --- |
| Seurat object | `scanpy.AnnData` |
| `nCount_RNA` | `adata.obs["total_counts"]` when raw counts are available |
| `nFeature_RNA` | `adata.obs["n_genes_by_counts"]` when raw counts are available |
| `FindNeighbors` | `sc.pp.neighbors` |
| `FindClusters` | `sc.tl.leiden` |
| UMAP | `sc.tl.umap` |
| 4 PCs for main clustering | `sc.pp.neighbors(..., n_pcs=4)` |
| mature ChP subclustering with PCs 1-12 | subset and rerun neighbors with `n_pcs=12` |

The analysis boundary is documented in
[docs/paper_methods_reproduction_plan.md](docs/paper_methods_reproduction_plan.md).

## Key Outputs From The First Pass

These figures were generated during the first-pass reanalysis and are retained
as portfolio outputs.

### Reviewed Cell-Type Composition

![Reviewed cell-type composition by sample](results/figures/reviewed_cell_type_composition_by_sample.png)

### Marker Expression By Reviewed Cell Type

![Marker dotplot by reviewed cell type](results/figures/marker_dotplot_by_reviewed_cell_type.png)

## Repository Structure

```text
.
├── README.md
├── config/
│   └── paper_parameters.yml
├── data/
│   └── README.md
├── docs/
│   ├── github_quickstart.md
│   └── paper_methods_reproduction_plan.md
├── notebooks/
│   ├── 01_exploratory_first_pass_reanalysis.ipynb
│   ├── 02_methods_guided_reanalysis.ipynb
│   ├── 03_mature_chp_subclustering_stress_mito.ipynb
│   └── 04_reference_comparison_and_enrichment.ipynb
├── results/
│   └── figures/
│       ├── reviewed_cell_type_composition_by_sample.png
│       └── marker_dotplot_by_reviewed_cell_type.png
├── environment.yml
├── requirements.txt
└── .gitignore
```

## How to Reproduce

Create the conda environment:

```bash
conda env create -f environment.yml
conda activate gse150903-scrna
```

Or install the Python requirements:

```bash
pip install -r requirements.txt
```

Download the processed GEO matrix and place it as described in
[data/README.md](data/README.md). Then open:

```text
notebooks/02_methods_guided_reanalysis.ipynb
```

Run the notebook from the repository root so relative paths such as
`data/processed/...` resolve correctly.

The mature ChP follow-up notebook starts from the annotated AnnData file saved by
notebook 02:

```text
notebooks/03_mature_chp_subclustering_stress_mito.ipynb
```

It subclusters mature ChP-like cells and scores stress, mitochondrial, dark ChP,
ciliated ChP, and barrier/transport gene programs.

The enrichment and reference-comparison notebook starts from the mature ChP
object saved by notebook 03:

```text
notebooks/04_reference_comparison_and_enrichment.ipynb
```

It generates pseudobulk summaries, extracts markers, performs functional
theme/enrichment checks, compares dark and light ChP-like programs, and reviews
sample-stage composition.

## Data Source

- GEO: [GSE150903](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE150903)
- Organism: *Homo sapiens*
- Experiment type: expression profiling by high-throughput sequencing
- Samples: telencephalon organoids and choroid plexus organoids at days 27, 46,
  and 53

Large expression matrices and generated AnnData files are intentionally excluded
from GitHub.

## Notes

This is a learning-focused reanalysis. The current recommended workflow uses
the authors' processed SCT-scaled matrix for downstream analysis. A full
from-raw reproduction would require raw 10X/SRA inputs and either rerunning
CellRanger/Seurat or building a separate raw-count Python pipeline.
