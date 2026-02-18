---
title: Columnar encoding for multimodal single-cell data
status: under investigation
beads: data-a0y.1
---

## Context

omicsio converts AnnData and MuData objects from the scverse ecosystem into columnar table formats (parquet and vortex) for lakehouse storage and analytical querying via DuckDB.
The representation chosen for count and expression matrices is the central design decision of the library because it determines query performance, storage efficiency, downstream consumer compatibility, and the fidelity with which data can be reconstructed into its original in-memory form.

Single-cell expression matrices are large, sparse, and structurally varied.
A typical RNA dataset contains 10k-1M cells with expression values across 20k-60k genes at 5-10% density.
Multimodal experiments add further complexity: splice-state quantification produces parallel count matrices for spliced, unspliced, and ambiguous transcripts; paired assays like 10x Multiome produce RNA and ATAC-seq data with entirely different feature spaces measured from the same cells.
MuData structures formalize this by organizing modalities with independent variable (feature) dimensions under a shared observation (cell) axis.

Two existing implementations provide concrete starting points for this design space.
X-Atlas-Orion uses a denormalized list-column format optimized for DuckDB analytical queries and GPU batch streaming.
pqdata uses a directory-tree format mirroring the AnnData object graph, with sparse matrices decomposed to COO triplet columns.
Each approach makes different tradeoffs, and neither directly addresses multimodal data at the schema level.

This document frames the two coupled questions that omicsio must resolve, surveys what is known from existing implementations and prior analysis, identifies the use cases against which any solution must be evaluated, and catalogs the open questions that remain.

## Two coupled questions

The first question concerns representation: how should multimodal count matrices be encoded in columnar table formats?
The encoding must work in both parquet and vortex, accommodate modalities with different feature dimensions (RNA genes vs ATAC peaks), and support efficient analytical queries without requiring reconstruction into dense matrices.
The X-Atlas-Orion list-column approach provides a strong foundation for single-modality RNA data, but extending it to splice-state layers and cross-modality structures introduces schema design choices that have not been explored in any existing implementation.

The second question concerns rehydration: how should the lakehouse-native format support round-trip conversion back to AnnData and MuData when downstream tools require scverse objects?
The ideal data path for out-of-core analytics avoids materializing full AnnData objects entirely, flowing instead from columnar storage through Arrow record batches to dense or sparse arrays consumed directly by DuckDB, JAX, or PyTorch.
However, many tools in the scverse ecosystem accept only AnnData inputs, making reconstruction a practical necessity.
The rehydration path should be a compatibility layer rather than the primary access pattern, and its design should not constrain the columnar encoding choices.

These questions are coupled because the columnar encoding determines what information is preserved and how efficiently it can be reassembled into scverse objects, while the rehydration requirements constrain which encodings are viable.

## Known approaches

### X-Atlas-Orion: list-column format

X-Atlas-Orion encodes single-cell expression data as a denormalized cell-centric table where each row represents one cell.
Expression values are stored in two parallel list columns: `gene_token_id` (integer identifiers for genes with non-zero expression) and `gene_expression` (corresponding expression values).
A separate `gene_metadata.parquet` table maps integer token IDs to Ensembl IDs and gene names.
Cell-level metadata columns (barcode, sample, QC metrics, perturbation labels) sit alongside the list columns in the same row.

This schema was designed for the Xaira Therapeutics Perturb-seq atlas (8M cells across two cell lines) and published as a HuggingFace dataset.
The original implementation uses `BIGINT[]` for token IDs and `DOUBLE[]` for expression values, though prior analysis in this repository recommends narrowing to `INT32[]` and `FLOAT32[]` respectively.

The list-column format enables several query patterns directly in DuckDB.
Gene presence can be checked with `list_contains(gene_token_id, target_id)`, and expression values extracted with `list_position` for indexed lookup.
The X-Atlas-Orion repository includes a Wnt pathway query demonstrating multi-gene co-expression filtering with a `get_gene_expr` macro that extracts individual gene values from the parallel lists.
The tutorial code shows reconstruction back to AnnData by iterating rows, mapping token IDs to column indices, and assembling a CSR sparse matrix.

The structural relationship between Arrow ListArray offsets and CSR indptr arrays is the key property enabling efficient conversion between the list-column format and sparse matrix representations.
Within a single non-chunked ListArray, the offsets array is structurally identical to CSR indptr, and the values array maps directly to CSR data and indices arrays.

The format does not currently address multimodal data, splice-state layers, or any AnnData component beyond X and obs-level metadata.

### pqdata: COO directory format

pqdata serializes the full AnnData and MuData object graph as a directory tree of parquet files.
The directory structure mirrors the AnnData hierarchy: `X.parquet` for the expression matrix, `obs.parquet` and `var.parquet` for metadata dataframes, subdirectories for `obsm/`, `varm/`, `obsp/`, `varp/`, `layers/`, and `uns/`.
For MuData, a `mod/` subdirectory contains one nested AnnData directory per modality, with modality ordering preserved in a `pqdata.json` sidecar file.

Sparse matrices are decomposed to COO format with three columns (`row`, `col`, `data`) in a single parquet file.
Matrix shape and original sparse class (CSR, CSC, COO) are stored as JSON in the Arrow schema metadata, enabling reconstruction to the original scipy sparse type on read.
Dense matrices are stored with columns named after variable names (gene names as parquet column names).

pqdata provides a `Group` and `GroupAccessor` class hierarchy that presents an HDF5/Zarr-like interface over the directory tree, supporting `fsspec` for remote filesystem access.
The read path detects COO format by checking for `row`, `col`, `data` column names and reconstructs sparse matrices using `scipy.sparse.coo_matrix`.
MuData is reconstructed via `MuData._init_from_dict_()` from the nested directory structure.

The COO format preserves lossless round-trip fidelity for all AnnData and MuData components including layers, pairwise matrices, unstructured metadata, and raw data.
However, the COO decomposition is not optimized for analytical queries: querying expression values for specific genes requires scanning the full COO table and filtering on the `col` column, with no columnar projection benefit.

### Comparative assessment from prior analysis

The architecture document in this repository (`single-cell-data-architecture-for-queries-and-training.md`) concludes that the X-Atlas-Orion list-column pattern is the superior foundation for queryable expression data across three constraints: array query efficiency, network-scalable queries, and storage efficiency.
pqdata retains advantages for lossless AnnData round-tripping and unstructured metadata components (uns, obsp/varp) that do not benefit from columnar query patterns.

The recommended refinements over raw X-Atlas-Orion include narrower integer and float types, sorted token ID lists as a schema invariant for improved compression, biologically meaningful partitioning, inline fixed-size list columns for embeddings, and separate tables per modality with a shared cell ID key for multimodal data.
The architecture document provides a DDL sketch with `rna_cells`, `rna_layers_spliced`, `rna_layers_unspliced`, `rna_layers_ambiguous`, `atac_cells`, and `gene_cell_index` tables.

## Use cases to evaluate against

Any encoding scheme must support the following data configurations, which represent the range of structures omicsio will encounter in practice.

Transcriptomic data with total RNA is the simplest case: a single expression matrix (X) with one count per cell-gene pair.
This is what X-Atlas-Orion directly addresses and what most scverse tutorials assume.

RNA separated by spliced and unspliced counts appears in RNA velocity workflows.
scvelo and pyrovelocity expect `adata.layers["spliced"]` and `adata.layers["unspliced"]` as sparse matrices with the same shape as X.
The spliced and unspliced matrices share the same feature space (genes) but have different sparsity patterns and different values.

RNA separated by spliced, unspliced, and ambiguous counts is the three-layer variant produced by STARsolo, kallisto-bustools, and the binseq pipeline described in the companion processing document.
The ambiguous layer captures reads that cannot be definitively assigned to a splice state.

Multimodal data with ATAC-seq from 10x Multiome format pairs RNA expression with chromatin accessibility measurements from the same cells.
The RNA and ATAC modalities have different feature dimensions (genes vs peaks), different sparsity characteristics, and different value semantics (UMI counts vs binary accessibility or fragment counts).
In AnnData terms, these are stored as separate AnnData objects within a MuData container, sharing the same observation axis but with independent variable axes.

MuData structures with modalities of different feature dimensions generalize the multiome case to arbitrary combinations of assays.
CITE-seq pairs RNA with protein measurements (typically 100-300 surface proteins).
Spatial transcriptomics may pair gene expression with spatial coordinates and morphological features.
The key structural property is that modalities share cells but have independently sized feature spaces.

## Downstream consumer constraints

Three downstream projects in the ecosystem consume single-cell data, and their format expectations constrain what omicsio must support.

pyrovelocity implements probabilistic RNA velocity models using PyTorch and Pyro.
It loads data exclusively as AnnData objects from h5ad files, using `scanpy.read()` with sparse matrix support.
The preprocessing pipeline accesses `adata.layers["spliced"]` and `adata.layers["unspliced"]` (and copies to `raw_spliced` / `raw_unspliced`), computes library sizes from these layers, and expects scipy sparse matrices.
There is no support for parquet, vortex, or DuckDB-based data loading.
pyrovelocity requires full AnnData rehydration for any data delivered via omicsio.

stormi implements stochastic differential equation models for gene regulatory network inference using JAX and NumPyro.
It operates on AnnData objects for preprocessing, expecting separate RNA and ATAC AnnData objects as inputs.
The preprocessing pipeline converts expression matrices to CSR format (`csr_matrix(adata.X)`), performs metacell aggregation, and produces dense numpy arrays as model inputs.
The training loop consumes dense numpy arrays sliced by a cell minibatch loader, not AnnData objects directly.
stormi's data path is AnnData for preprocessing, then dense arrays for training, suggesting that a DuckDB-to-Arrow-to-dense path could bypass AnnData for the training phase if the preprocessing steps were adapted.

hodosome is a placeholder project for stochastic dynamical modeling of gene regulatory networks.
It depends on pyrovelocity and stormi for modeling algorithms and outputs DuckDB data for visualization.
No data loading code exists yet, so its format constraints are inherited from its dependencies and from the DuckDB output requirement.
hodosome represents an opportunity to design data loading against the lakehouse-native format from the start rather than inheriting AnnData assumptions.

Across all three consumers, the common pattern is that preprocessing requires AnnData with layers (particularly spliced/unspliced for velocity), while training consumes dense or sparse numpy/JAX arrays.
The rehydration requirement is driven primarily by preprocessing compatibility with scverse tools (scanpy, scvelo, scvi-tools) rather than by the modeling frameworks themselves.

## Open questions

Several questions remain unresolved and would benefit from experimental validation or further analysis.

How should splice-state layers relate to the primary expression table?
The DDL sketch in the architecture document uses separate tables per layer (`rna_layers_spliced`, `rna_layers_unspliced`, `rna_layers_ambiguous`), each with their own `cell_id`, `gene_token_id`, and `gene_expression` columns.
An alternative is to add additional list columns to the primary cell table (`spliced_token_id`, `spliced_expression`, `unspliced_token_id`, `unspliced_expression`), keeping all per-cell data in a single row.
The separate-table approach avoids wide rows and allows layers to be queried independently, but requires joins for velocity workflows that need spliced and unspliced counts together.
The inline approach keeps related data co-located for joint queries but produces wider rows with more list columns.
The tradeoff depends on typical query patterns and on how DuckDB handles joins versus wide list-column rows at scale.

How should multimodal data be organized across tables?
The architecture document recommends separate tables per modality with a shared `cell_id` key.
This aligns with the MuData model where modalities have independent feature spaces.
However, it raises questions about how cross-modality queries perform, how metadata shared across modalities (cell-level annotations) should be stored (duplicated per table or in a shared obs table), and whether the DuckLake catalog should represent each modality as a separate table or group them under a dataset-level namespace.

What is the performance difference between COO and list-column formats for reconstruction?
The list-column format maps efficiently to CSR via Arrow ListArray offsets, but reconstruction requires a token-to-column-index mapping step.
COO format maps directly to `scipy.sparse.coo_matrix` but then requires conversion to CSR for most downstream operations.
Benchmarking both reconstruction paths at realistic scale (100k-1M cells, 20k-60k genes) would quantify whether the format choice materially affects rehydration performance.

How does vortex encoding interact with list-column schemas?
Sorted integer token ID lists should trigger FastLanes delta encoding in vortex, and float expression lists should get ALP encoding.
However, the interaction between vortex's encoding selection and nested list types has not been empirically validated.
Benchmarks comparing vortex vs parquet storage size and query latency for list-column expression data would inform whether vortex's theoretical advantages materialize for this specific data shape.

Can rehydration be lazy rather than eager?
Rather than reconstructing a full AnnData object in memory, a lazy proxy could back AnnData's `.X`, `.layers`, `.obs`, and `.var` properties with DuckDB queries that materialize data on access.
This would enable scverse tool compatibility without full materialization, potentially supporting out-of-core workflows where the full dataset exceeds host RAM.
The feasibility depends on which AnnData operations downstream tools actually invoke and whether the anndata library's internal assumptions permit lazy backing stores.

What token ID assignment strategy should omicsio use?
X-Atlas-Orion assigns integer token IDs derived from its specific gene reference (GRCh38 2024-A).
For a general-purpose tool, token IDs could be assigned per-dataset (simple sequential), per-organism-reference (stable across datasets using the same reference), or via a global registry.
The choice affects whether token IDs are meaningful across datasets and whether the gene metadata table needs versioning.

How should embeddings and dimensionality reductions be handled?
The architecture document recommends inline fixed-size list columns for embeddings (e.g., `umap FLOAT[2]`, `pca FLOAT[50]`).
This works for standardized reductions but becomes unwieldy if datasets carry many different embeddings or if embedding dimensions vary.
An alternative is a separate `obsm` table with embedding name as a key column.

## References

### Source implementations

- X-Atlas-Orion dataset and schema: `~/projects/omicslake-workspace/X-Atlas-Orion/`
- X-Atlas-Orion query examples: `~/projects/omicslake-workspace/X-Atlas-Orion/queries/wnt-pathway-query.sql`
- X-Atlas-Orion AnnData reconstruction tutorial: `~/projects/omicslake-workspace/X-Atlas-Orion/tutorials/filter_convert_to_anndata.py`
- pqdata library: `~/projects/omicslake-workspace/pqdata/`
- pqdata COO write implementation: `~/projects/omicslake-workspace/pqdata/src/pqdata/io/write.py`
- pqdata COO read implementation: `~/projects/omicslake-workspace/pqdata/src/pqdata/io/read.py`

### Design documents in this repository

- Single-cell data architecture for queries and training: `docs/notes/omicsio/single-cell-data-architecture-for-queries-and-training.md`
- Single-cell processing pipeline (binseq to count matrices): `docs/notes/omicsio/single-cell-processing-pipeline-binseq-to-count-matrices.md`

### Context documents

- omicsio architecture context: `~/projects/sciexp/planning/contexts/omicsio.md`
- sciexp data platform context: `CLAUDE.md` (repository root)

### Downstream consumers

- pyrovelocity data loading: `~/projects/pyrovelocity-workspace/pyrovelocity/src/pyrovelocity/io/datasets.py`
- pyrovelocity preprocessing (splice-state layer access): `~/projects/pyrovelocity-workspace/pyrovelocity/src/pyrovelocity/tasks/preprocess.py`
- stormi preprocessing pipeline: `~/projects/pyrovelocity-workspace/stormi-review/stormi/packages/stormi/src/stormi/preprocessing/_pipeline.py`
- stormi multimodal embedding: `~/projects/pyrovelocity-workspace/stormi-review/stormi/packages/stormi/src/stormi/preprocessing/_embedding.py`
- stormi training data loader: `~/projects/pyrovelocity-workspace/stormi-review/stormi/packages/stormi/src/stormi/train.py`
- hodosome (placeholder): `~/projects/hodosome-workspace/hodosome/`
