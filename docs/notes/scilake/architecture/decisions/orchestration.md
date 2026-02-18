# Workflow orchestration decision

## Status

Proposed

## Context

Omicslake needs to orchestrate data ingestion pipelines that download from heterogeneous sources, convert formats, track lineage, register with DuckLake, and publish to HuggingFace Hub.
The complexity ranges from simple single-step conversions to multi-stage pipelines with dependencies, retries, and provenance tracking.

## Options evaluated

### Dagster

Asset-centric workflow orchestrator with built-in lineage tracking.

**Strengths:**
- Asset model directly represents datasets; dependencies are explicit in the graph
- Built-in lineage and column-level metadata via `MaterializeResult`
- Resources abstract heterogeneous data sources (S3, HTTP, APIs)
- Python-native; integrates cleanly with omicsio and DuckDB
- No external cluster required; can run locally or in production mode
- Pre-built integrations for S3, GCP, databases

**Limitations:**
- Learning curve for asset graph composition and resource wiring
- No built-in HuggingFace Hub connector (requires custom ~50 LOC)
- DuckLake registration requires custom implementation

### Flyte SDK v2

Async-first workflow engine with pure Python control flow.

**Strengths:**
- True Python control flow (loops, conditionals) without DSL constraints
- `asyncio.gather()` for native distributed parallelism
- `flyte.io.File/Dir` handles file transport across storage backends
- Tracing infrastructure for function-level observability
- Can run locally without cluster via `flyte.init()`

**Limitations:**
- Lineage is implicit (execution history), not explicit data provenance
- Only BigQuery connector is stable; GEO/Zenodo/Kaggle require custom wrappers
- Migration from pyrovelocity's v1 flytekit requires rewrites
- No data catalog integration

### Temporal

Durable execution engine with event-sourced workflow history.

**Strengths:**
- Long-running downloads survive failures; automatic checkpoint/resume
- Built-in retry with configurable backoff for transient network failures
- Event-sourced history creates audit trail
- Excellent observability via OTEL integration

**Limitations:**
- Requires separate server cluster (Temporal Cloud or self-hosted)
- Tracks execution lineage, not data lineage
- No format conversion integration; purely orchestration
- Infrastructure overhead exceeds benefit for data-focused pipelines

### Marhar pattern (no orchestrator)

Shell scripts + SQL + DuckDB CLI.

**Strengths:**
- Zero framework overhead; uses only standard tools
- Scripts are transparent and readable
- Fast iteration; `duckwatch` enables interactive development
- Native DuckDB integration with full SQL syntax

**Limitations:**
- Manual ordering; DAG enforcement prevents silent failures at scale
- No automatic retry on transient failures
- No lineage tracking beyond git history
- Breaks down beyond ~5 sequential steps

## Decision

**Dagster** for pipelines requiring lineage tracking, retry logic, or >5 steps.

**Marhar pattern** remains valid for simple, one-off conversions where Dagster overhead is unjustified.

## Rationale

Dagster's asset-centric model aligns with scilake's core abstraction: datasets as versioned, traceable artifacts.
The research findings showed:

1. Dagster provides automatic lineage without extra implementation
2. Asset dependencies make data flow explicit and debuggable
3. Python-native resources integrate cleanly with omicsio and DuckDB
4. No external cluster required reduces operational burden

Flyte v2 and Temporal are better suited for compute-heavy workflows where durability matters more than data lineage.
Omicslake's primary concern is provenance tracking, where Dagster excels.

## Consequences

- Dagster assets become the primary abstraction for dataset ingestion
- Custom resources needed for: GEO FTP, Zenodo, Kaggle API, HuggingFace Hub
- DuckLake registration implemented as downstream asset depending on parquet materialization
- Learning curve for team members unfamiliar with Dagster concepts
- Simple pipelines can still use marhar scripts without Dagster

## References

- ~/projects/scilake-workspace/dagster — Local Dagster source
- Dagster assets documentation: https://docs.dagster.io/concepts/assets
- Research: Dagster suitability assessment (agent a29b7f8)
