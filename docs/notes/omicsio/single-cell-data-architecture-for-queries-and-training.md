---
title: Single-cell data architecture for network queries and JAX GPU training
---

## Context

This document synthesizes conclusions from comparative schema analysis (pqdata vs X-Atlas-Orion list-column format), JAX GPU data loading landscape research (Grain, ArrayRecord, DLPack, DuckDB streaming), and architectural design for the omicsio pipeline.
The analysis covers two distinct data regimes: full-transcriptome (60k genes, low density) and focused-gene (2k genes, moderate density).

The upstream processing pipeline (binseq through splice-state count matrices) is documented in the companion document `single-cell-processing-pipeline-binseq-to-count-matrices.md`.
This document begins at the Arrow in-memory boundary where that pipeline ends.

Source conversation transcripts are archived alongside this document:
- `pqdata-vs-xatlas-orion-schema-comparison.md`
- `jax-gpu-data-loading-parquet-vortex-sparse-schema-analysis.md`
- `binseq-minimap2-splice-pipeline-design.md` (pending archival)

## Schema comparison: pqdata vs X-Atlas-Orion

pqdata mirrors the AnnData object graph as a directory tree of parquet files (X.parquet stores expression as COO triplets or dense gene-named columns, with separate obs.parquet, var.parquet, and subdirectories for obsm, layers, obsp, uns).
X-Atlas-Orion uses a denormalized cell-centric table where each row carries inline parallel list columns `gene_token_id BIGINT[]` and `gene_expression DOUBLE[]` encoding only non-zero entries, with a separate `gene_metadata.parquet` lookup table mapping integer tokens to ensembl IDs and gene names.

The X-Atlas-Orion list-column pattern is the superior foundation for queryable expression data across three constraints: array query efficiency, network-scalable queries, and storage efficiency.
pqdata retains advantages for lossless AnnData round-tripping and unstructured metadata components (uns, obsp/varp) that do not benefit from columnar query patterns.

### Recommended refinements over raw X-Atlas-Orion

The following refinements improve on X-Atlas-Orion's original schema:

- Use INT32 rather than BIGINT for token IDs (40k genes fit in int32, halves storage and improves compression via narrower bitpacking).
- Use FLOAT32 rather than DOUBLE for expression values (pre-normalization counts can be INT32 or INT16).
- Guarantee sorted token ID lists as a schema invariant (enables binary search semantics and dramatically improves delta-encoding compression).
- Partition by biologically meaningful units (dataset, tissue, organism) rather than arbitrary batches, enabling partition pruning for cross-study queries.
- Store embeddings as fixed-size list columns inline (e.g., `umap FLOAT[2]`, `pca FLOAT[50]`).
- Use separate tables per modality with a shared cell_id key for multimodal (MuData-like) data, preserving modality-specific feature spaces while enabling cross-modality joins.
- Target vortex as the storage format where supported (sorted INT32 token ID lists trigger FastLanes delta compression automatically, expression FLOAT32 lists get ALP compression, and random access is approximately 100x faster than parquet).

### Corrections to prior analysis

The claim that Arrow ListArray offsets are "structurally identical to CSR indptr" is correct within a single non-chunked ListArray but requires care in three scenarios.

First, chunked arrays: DuckDB `fetch_record_batch()` returns individual RecordBatch objects with contiguous offsets starting from 0, avoiding the problem.
Using `fetchall()` or `arrow()` on large results may return a ChunkedArray where each chunk has per-chunk offsets — concatenating naively produces invalid indptr.
The fix is to iterate chunks and accumulate offsets.

Second, null handling: a null list entry produces `offsets[i] == offsets[i+1]` (empty range), identical to a zero-length list.
The validity bitmap distinguishes them but `.offsets` does not carry it.
For CSR construction this is acceptable when nulls mean "no expressed genes."

Third, dtype alignment: Arrow ListArray offsets are int32 by default, matching SciPy `csr_matrix` expectations when gene counts and batch sizes fit in int32 (as they do for our use case).

The `list_contains` + `list_position` query pattern performs O(k) linear scans per cell (where k is the number of expressed genes, typically 2k-8k).
DuckDB does not exploit sort order for `list_contains` even on sorted lists.
At 100M cells with multiple gene lookups per row, this produces trillions of comparisons.
Mitigations include pre-filtering with metadata columns, materializing boolean gene-set membership columns at ingestion, and maintaining an inverted index table (`gene_token_id INT32, cell_rowids LIST<INT64>`) for gene-centric queries at scale.

## Schema DDL

```sql
CREATE TABLE gene_metadata (
    gene_token_id INT32 PRIMARY KEY,
    ensembl_id VARCHAR NOT NULL,
    gene_name VARCHAR NOT NULL,
    chromosome VARCHAR,
    gene_type VARCHAR
);

CREATE TABLE rna_cells (
    cell_id VARCHAR NOT NULL,
    gene_token_id LIST(INT32) NOT NULL,     -- sorted ascending invariant
    gene_expression LIST(FLOAT32) NOT NULL, -- parallel to gene_token_id
    cell_type VARCHAR,
    tissue VARCHAR,
    organism VARCHAR,
    dataset VARCHAR NOT NULL,
    batch VARCHAR,
    umap FLOAT[2],
    n_genes INT32,
    total_counts FLOAT,
    CHECK (len(gene_token_id) = len(gene_expression))
);

CREATE TABLE rna_layers_spliced (
    cell_id VARCHAR NOT NULL,
    gene_token_id LIST(INT32) NOT NULL,
    gene_expression LIST(FLOAT32) NOT NULL
);

CREATE TABLE rna_layers_unspliced (
    cell_id VARCHAR NOT NULL,
    gene_token_id LIST(INT32) NOT NULL,
    gene_expression LIST(FLOAT32) NOT NULL
);

CREATE TABLE rna_layers_ambiguous (
    cell_id VARCHAR NOT NULL,
    gene_token_id LIST(INT32) NOT NULL,
    gene_expression LIST(FLOAT32) NOT NULL
);

CREATE TABLE atac_cells (
    cell_id VARCHAR NOT NULL,
    peak_token_id LIST(INT32) NOT NULL,
    peak_accessibility LIST(FLOAT32) NOT NULL,
    dataset VARCHAR NOT NULL
);

CREATE TABLE gene_cell_index (
    gene_token_id INT32 NOT NULL,
    dataset VARCHAR NOT NULL,
    cell_rowids LIST(INT64) NOT NULL
);
```

## Network query architecture

DuckDB's httpfs extension provides first-class support for S3-compatible storage and HuggingFace paths (`hf://datasets/<user>/<dataset>/<path>` with glob support, branch references via `@` syntax, and authentication via DuckDB Secrets Manager).
The vortex-duckdb extension supports S3 remote storage via the Rust `object_store` crate with range requests, retaining vortex's random access and compression advantages over the network.

The gene metadata table is small (~60k rows, ~2 MB) and should be cached locally on first access rather than embedded in each partition.

### Progressive refinement query pattern

The pipeline minimizes bytes transferred at each stage.

Stage 1 (metadata scan, ~KB transferred): query scalar metadata columns to identify cells of interest via column projection.

Stage 2 (cell filtering with gene presence, ~MB transferred): check gene presence using `list_contains` on the `gene_token_id` column without extracting expression values.

Stage 3 (expression extraction for gene subset, ~10s of MB transferred): extract expression values for a small gene panel using parallel list indexing.

Stage 4 (batch materialization, ~100s of MB transferred): stream full expression vectors using `fetch_record_batch()` for training data preparation.

### DuckLake catalog role

DuckLake provides snapshot-based versioning, schema evolution tracking, and file-level metadata per table.
Partitioning is achieved through table organization and file layout rather than explicit partition management APIs.
Create one DuckLake table per modality with files partitioned by dataset, tissue, and organism.

## JAX GPU data loading

### The two data regimes

The full-transcriptome regime (60k genes, 5% density, large cell counts) and focused-gene regime (2k genes, 33% density) have different optimal architectures because the memory profiles differ by orders of magnitude.

For full-transcriptome: dense full dataset at 1M cells = 240 GB (exceeds host RAM for large cell counts).
Batch streaming from DuckDB with Arrow ListArray zero-copy extraction and numba parallel scatter to dense is the recommended path.
Each batch of 1024 cells produces a 245 MB dense array.

For focused-gene: dense full dataset at 8M cells × 2k genes = 64 GB (fits in host RAM).
Pre-loading the entire dataset as a dense NumPy array, shuffling with `np.random.permutation`, and streaming batches via `jax.device_put` with sharding is simpler and equally performant.
At 33% density with only 2k genes, dense parquet with gene columns is a competitive storage format — column projection works directly, no scatter reconstruction needed, and parquet's RLE encoding handles zeros well.

### Arrow ListArray to dense reconstruction

The core conversion path from list-column format to dense arrays:

```
DuckDB fetch_record_batch(batch_size)
  → Arrow RecordBatch (ListArray columns)
  → .offsets.to_numpy() + .values.to_numpy()  (zero-copy views)
  → numba parallel scatter to dense np.ndarray
  → jax.device_put() with sharding
```

The numba scatter function operates in parallel across rows using `numba.prange` and produces a `(batch_size, num_genes)` float32 array directly.
For gene-subset queries, a variant scatter function accepts a `gene_to_col` mapping array and produces a `(batch_size, panel_size)` dense matrix, filtering during scatter rather than as a separate step.

### Grain and ArrayRecord

Grain's Parquet support (`grain.experimental.ParquetIterDataset`) is sequential-access only with no RandomAccessDataSource implementation, no global shuffle capability, and degraded LIST column handling (PyArrow `to_numpy()` on LIST columns produces Python object arrays).
These limitations are architectural rather than implementation gaps.

For full Grain feature utilization (DataLoader, global shuffling, deterministic replay, Orbax checkpointing), converting to ArrayRecord is required.
ArrayRecord provides O(1) index-based access via a footer index, batch read performance of 310-490K QPS, and is the format recommended by the JAX ecosystem for production training.

### Dual format strategy

Maintain both vortex (with list-column schema) and ArrayRecord in S3 storage.
These formats serve non-overlapping access patterns: vortex is columnar and query-optimized for analytics and exploration; ArrayRecord is record-oriented with O(1) random access for ML training with global shuffle, deterministic replay, and checkpoint/resume.

Store only the count matrix in ArrayRecord, not metadata, embeddings, or gene annotations (those belong exclusively in vortex, accessed via DuckDB).
For the 2k-gene regime, store pre-scattered dense rows (8 KB per record).
For the 60k-gene regime, store sparse list-column data and scatter at training time.

The materialization pipeline runs once per dataset ingestion as a post-processing step after vortex/parquet files are written, decoupled from the primary ingestion pipeline.

### When ArrayRecord is unnecessary

For datasets that fit in host RAM (the 8M × 2k = 64 GB case), pre-loading from vortex via DuckDB, shuffling in host memory, and streaming batches to GPU is simpler and performs well without Grain.

The crossover where ArrayRecord becomes worth maintaining is approximately when the dataset exceeds host RAM, training runs are long enough that preemption recovery matters, multi-host distributed training requires coordinated sharding, or reproducibility requirements demand bit-exact replay.

## Multi-GPU training architecture

### Equinox and JAX sharding

The primary training framework is JAX with Equinox (functional semantics) and optax for optimization, implementing amortized simulation-based Bayesian inference.
NumPyro serves as a comparison framework for smaller models.

For 2× A100 40GB: data-parallel training with batch sharding across devices.
Batch of 8192 cells (4096 per device, 32 MB each) leaves ~39 GB per device for model and optimizer state.
Host-to-device transfer of 64 MB total batch takes ~2.5 ms at PCIe 4.0.

For 8× H100 80GB: the full 64 GB dataset can be sharded across devices (8 GB each), eliminating host-to-device transfer during training.
Use `jax.device_put(data, NamedSharding(mesh, PartitionSpec("data")))` at initialization.
With 65536-cell batches (8192 per device), training completes ~122 steps per epoch.

### Memory budget

| Configuration | Dense batch (4096 cells × 2k) | Full dataset (8M × 2k) |
| --- | --- | --- |
| Per device | 32 MB | 8 GB (sharded across 8 H100s) |
| Host RAM | 64 GB (pre-loaded) | 64 GB |
| Double-buffered prefetch | 64 MB CPU + 32 MB GPU | N/A (on-GPU) |

For 60k-gene batches: 1024 cells = 245 MB, 4096 cells = 983 MB per device.

## Ingestion pipeline output formats

The Rust-based processing pipeline is documented in `single-cell-processing-pipeline-binseq-to-count-matrices.md`.
In summary: SRA archives and FASTQ files are converted to binseq CBQ format, aligned with minimap2-rs in splice-aware mode, classified into splice states (spliced/unspliced/ambiguous) via CIGAR-based annotation overlap ported from precellar, and deduplicated at the UMI level.
The pipeline produces Arrow `RecordBatch` values in the list-column schema, which are then written to three output formats:

- Vortex files in pqdata directory layout: primary queryable format for analytics and exploration, with separate layer files for each splice state.
- Parquet files via duckdb-rs: compatibility format for tools without vortex support, benefiting from DuckDB's aggressive dictionary/RLE/zstd encoding.
- ArrayRecord with dense or sparse rows: training-optimized format, materialized as a separate post-processing step.

Arrow IPC serves as an additional in-process hand-off mechanism for downstream Rust or Python consumers without file format overhead.

For the 2k-gene regime, dense parquet with gene columns is an additional viable storage option since 2k columns is within parquet's comfortable range and column projection provides direct gene-level access without list functions.

## Summary of recommended architectures

For full-transcriptome data (60k genes, low density): vortex list-column schema, DuckDB streaming with Arrow ListArray extraction and numba scatter, batch streaming to GPU.
ArrayRecord for production training.

For focused-gene data (2k genes, moderate density): dense parquet with gene columns or vortex list-column (either works), pre-load to host RAM, shard across GPUs with JAX.
ArrayRecord when dataset exceeds host RAM or training requires deterministic replay.

The DuckLake catalog and gene_metadata table are shared infrastructure across both regimes.
The ingestion pipeline supports both output schemas from the same source data, selecting the appropriate format based on downstream use case.
