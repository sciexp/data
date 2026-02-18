---
title: "Single-cell processing pipeline: binseq to splice-state count matrices"
---

## Context

This document describes the upstream processing pipeline that produces the count matrices consumed by the query and training architecture documented in `single-cell-data-architecture-for-queries-and-training.md`.
That companion document covers everything downstream of the Arrow in-memory representation: schema design, network query patterns, JAX GPU data loading, and multi-GPU training architecture.
This document covers everything upstream: how raw sequencing data (SRA archives or FASTQ files) becomes splice-state-annotated gene-by-cell count matrices in vortex and parquet formats, with no FASTQ text or HDF5/H5AD intermediaries in the processing path.

Source conversation transcript is archived alongside this document:
- `binseq-minimap2-splice-pipeline-design.md` (pending archival)

## Design principles

The pipeline is built on three constraints that shape every technology choice.

First, no FASTQ text format in the processing pipeline.
FASTQ is an ASCII-encoded, line-oriented format that requires gzip decompression and text parsing before any useful work can happen.
The binseq CBQ format replaces this with 2-bit nucleotide encoding (32 bases per 64-bit word), memory-mapped zero-copy access, and block-based parallel processing.
The only point where ASCII nucleotide encoding appears is at the minimap2 C FFI boundary, where a trivial bit-unpack produces the byte sequences the aligner expects.
This is orders of magnitude cheaper than FASTQ decompression and parsing.

Second, no HDF5 or H5AD anywhere.
The traditional single-cell output format (AnnData stored as H5AD) is replaced by vortex files and DuckDB-written parquet files organized in the pqdata directory layout convention.
Both output formats are produced from a shared Arrow in-memory representation, which is also the handoff point to the downstream query and training architecture.

Third, genome-based splice state quantification rather than splici pseudo-alignment.
The pipeline aligns reads to the genome with a splice-aware aligner (minimap2 in splice:sr mode), then classifies each alignment by overlapping its CIGAR-derived segments with GTF exon annotations.
This approach preserves actual splice junction evidence from the aligner (CIGAR N operators marking introns) rather than inferring splice state from which reference sequences a read maps to.
The trade-off is that genome alignment is slower than pseudo-alignment, but the splice junction evidence is more informative, and the pipeline avoids the need to construct and maintain splici reference indices.

## Input sources

Raw sequencing data enters the pipeline through two paths, both producing binseq CBQ files as the common internal representation.

SRA archives are converted using xsra's `recode` subcommand, which reads the NCBI SRA binary format via the ncbi-vdb C library and writes directly to binseq without an intermediate FASTQ stage.
xsra supports streaming from NCBI, GCP, and AWS URLs without local prefetch, and its multi-threaded extraction divides spots across threads with per-thread local buffering.

FASTQ files are converted using bqtools, which reads compressed FASTQ and writes binseq.
This path exists because many datasets are distributed as FASTQ rather than SRA, and some sequencing facilities deliver FASTQ directly.

Both tools produce CBQ (columnar binary sequence) files, the recommended binseq variant.
CBQ uses columnar storage within blocks for better compression, native N-base support via Elias-Fano encoded position tracking, and 2-bit nucleotide encoding.
Block-based organization enables parallel processing via the `ParallelProcessor` trait without inter-thread synchronization.
Quality scores are stored as an optional column, preserved for workflows that need per-base quality (barcode correction, SNV calling) and omitted otherwise.

## Processing pipeline

The pipeline has six stages between binseq input and Arrow output.
Each stage operates on the output of the previous one, with the binseq memory-mapped representation persisting through the early stages and Arrow arrays accumulating in the later stages.

### Barcode and UMI extraction

Barcode and UMI segments are extracted from binseq records according to a seqspec YAML configuration that describes the read structure for a given sequencing platform.
seqspec defines which byte ranges within a read correspond to cell barcodes, UMIs, and insert sequences, abstracting over platform-specific layouts (10x Genomics, sci-RNA-seq, SHARE-seq, etc.).

The extraction produces three components per read: the barcode sequence, the UMI sequence, and the insert sequence(s) for alignment.
With binseq's paired-end support (primary and extended sequences in a single record), read1 and read2 are accessed without multi-file coordination.

### Barcode correction

Cell barcodes are corrected against a whitelist using probabilistic error correction with per-base quality scores, allowing up to 2 mismatches.
When no whitelist is provided, an automatic cell-calling algorithm (OrdMag or similar) identifies genuine barcodes from the frequency distribution.
This logic is well-established in precellar's `barcode.rs` and ports directly.

### Alignment

Insert sequences are aligned to the genome using minimap2-rs with `Preset::SpliceSr` (splice-aware short read mode).
minimap2-rs is a Rust FFI wrapper around minimap2 v2.30 that accepts raw `&[u8]` byte sequences and returns `Mapping` structs containing CIGAR strings, mapping quality, strand, and splice-specific fields (`is_spliced: bool`, `trans_strand: Option<Strand>`).

At this boundary, binseq's 2-bit encoded sequences are decoded to ASCII bytes for the minimap2 C library.
This is a bit-unpack operation, not a format conversion: no quality score re-encoding, no read name formatting, no newline insertion.

minimap2-rs also supports junction BED file guidance (`read_junction()`, `read_junction_lr()`) for improving splice alignment accuracy with known junction databases, and exposes junction scoring with canonical splice site detection (GT-AG, GC-AG, AT-AC).

The choice of minimap2 over STAR for this pipeline reflects the availability of clean Rust FFI bindings (minimap2-rs) that accept raw byte sequences, versus STAR's Rust wrapper (star-aligner) which re-serializes to FASTQ text before the C FFI call and does not expose STARsolo's classification logic.
minimap2's `splice:sr` preset is the appropriate mode for short-read splice-aware alignment.

### Annotation overlap and splice state classification

Each alignment's CIGAR string is parsed to extract aligned segments separated by N (skip/intron) operators.
These segments are overlapped with exon coordinates from a GTF gene annotation to classify each alignment into one of four categories: exonic (all segments within exons with concordant junctions), intronic (no exon overlap), spanning (segments cross exon-intron boundaries), or discordant (splice junctions contradict transcript structure).

This logic is ported from precellar's `transcriptome/align.rs` (~273 lines) which builds `SplicedRecord` structs from CIGAR parsing, and `transcriptome/annotate.rs` (~61 lines) which performs the exon overlap classification.
The port consumes minimap2-rs `Mapping` structs (which provide CIGAR as `Vec<(u32, u8)>` where operation code 3 is the N/intron operator) instead of noodles `RecordBuf`.

Splice state is determined at the UMI level, not the individual read level.
If all reads supporting a UMI are exonic-only, the molecule is classified as spliced.
If any read is spanning, or all compatible transcripts have at least one intronic/boundary read, the molecule is classified as unspliced.
Everything else is ambiguous.

### UMI deduplication

UMIs within each cell-gene group are deduplicated using Hamming-distance-1 correction and fingerprint-based duplicate detection.
For ATAC-seq data (when the pipeline is extended to support it), deduplication uses 5-prime coordinate fingerprints instead.

### Count matrix construction

Deduplicated UMI counts are accumulated into a gene-by-cell count matrix with three splice state layers: spliced, unspliced, and ambiguous.
The matrix is represented as Arrow arrays in the X-Atlas-Orion list-column schema: each cell is a row with parallel `gene_token_id LIST(INT32)` and `gene_expression LIST(FLOAT32)` columns containing only non-zero entries, with gene token IDs sorted ascending as a schema invariant.

The three splice state layers use the same list-column structure, stored as separate tables (or separate files in the pqdata directory layout).

## Output formats

All three output formats are produced from the same in-memory Arrow `RecordBatch` representation.
This is the boundary where this document ends and the companion architecture document begins.

### Vortex files in pqdata directory layout

The primary output uses vortex's Rust writer API to produce `.vortex` files organized in the pqdata directory convention:

```
dataset.vortex.pqdata/
  pqdata.json
  obs.vortex              # cell metadata
  var.vortex              # gene metadata (gene_token_id, ensembl_id, gene_name)
  X.vortex                # total count matrix (list-column schema)
  layers/
    spliced.vortex        # spliced UMI counts (list-column schema)
    unspliced.vortex      # unspliced UMI counts (list-column schema)
    ambiguous.vortex      # ambiguous UMI counts (list-column schema)
  obsm/
    ...                   # embeddings (populated by downstream analysis)
```

Vortex accepts Arrow arrays zero-copy and automatically selects optimal encodings: sorted INT32 token ID lists trigger FastLanes delta compression, FLOAT32 expression values get ALP compression, and sparse arrays with >90% fill value use the sparse encoding.

### Parquet files via duckdb-rs

The same Arrow `RecordBatch` values are passed to duckdb-rs via `append_record_batch()`, then written to parquet with `COPY ... TO ... (FORMAT PARQUET)`.
DuckDB's parquet writer applies aggressive dictionary encoding, run-length encoding, and page-level zstd compression, producing smaller files than the standard `parquet-rs` writer.
The output directory follows the same pqdata layout with `.parquet` extensions.

This path serves tools and environments without vortex support, and provides the broadest compatibility for network queries via DuckDB's httpfs extension.

### Arrow IPC

For in-process hand-off to downstream Rust or Python consumers (e.g., a post-processing step that computes embeddings or materializes ArrayRecord files), Arrow IPC serialization avoids any file format overhead.
This is not a persistent storage format but a zero-serialization-cost interprocess boundary.

## Key dependencies

- **binseq** (Arc Institute): binary sequence format library, CBQ variant with columnar storage, 2-bit encoding, memory-mapped parallel access
- **xsra** (Arc Institute): SRA archive extraction to FASTQ or binseq formats
- **bqtools** (Arc Institute): FASTQ to binseq conversion
- **minimap2-rs**: Rust FFI bindings to minimap2 v2.30, splice-aware alignment with CIGAR output, junction BED support
- **arrow-rs**: Apache Arrow Rust implementation, the in-memory interchange format shared by all output paths
- **vortex**: columnar file format with zero-copy Arrow compatibility, FastLanes/ALP/sparse encodings, Rust writer API
- **duckdb-rs**: DuckDB Rust crate, accepts Arrow RecordBatch, writes optimized parquet
- **seqspec**: read structure specification for platform-agnostic barcode/UMI extraction
- **noodles**: bioinformatics I/O library for GTF annotation parsing (exon coordinates)

## Comparison with alternative approaches

The pipeline's genome-based splice quantification differs from the two established approaches in the field.

The alevin-fry/piscem approach builds a "splici" reference containing both spliced transcripts and intronic sequences, maps reads against it using pseudo-alignment (k-mer lookup in a colored compacted de Bruijn graph), and classifies by which reference sequences match.
This is faster (no genome alignment) but loses splice junction evidence: a read landing entirely within a single exon is classified by reference identity alone, with no information about whether the molecule was actually spliced.

The STARsolo approach aligns to the genome with STAR, uses CIGAR N operators and exon annotation overlap to classify reads (essentially the same logic this pipeline ports), and integrates the classification with STARsolo's built-in UMI counting.
This is the closest equivalent to what our pipeline does, but STARsolo's classification logic is embedded in C++ and not exposed through the star-aligner Rust FFI wrapper.

Our pipeline takes the STARsolo-equivalent approach (genome alignment + annotation overlap) but implements it with minimap2-rs for alignment and a Rust port of precellar's classification logic.
The result is a pipeline that operates entirely in Rust, consumes binseq natively, and produces Arrow arrays directly, with no subprocess orchestration, no intermediate file formats, and no language boundary crossings except the minimap2 C FFI.
