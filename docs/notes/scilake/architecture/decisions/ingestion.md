# Data ingestion tooling decision

## Status

Proposed

## Context

Omicslake ingests data from heterogeneous sources: GEO FTP, Zenodo, S3, Kaggle API, CellxGene Census, and direct HTTP downloads.
Sources vary in protocol, authentication, schema stability, and update patterns.
The ingestion layer must handle downloads, schema inference, and incremental updates while preserving the zero-copy DuckLake registration pattern.

## Options evaluated

### dlt (data load tool)

Python-native ELT framework with built-in source connectors.

**Strengths:**
- Production-ready sources for filesystem (S3, GCS, local), REST APIs, SQL databases
- Automatic schema inference and evolution (flattens nested JSON, handles type changes)
- State management for incremental loading (cursor-based, lag windows)
- `load_info` provides operator visibility into pipeline runs

**Critical limitation:**
dlt's DuckLake destination uses `INSERT INTO ... SELECT * FROM read_parquet()` — it copies data into the catalog rather than registering external files.
This conflicts with the marhar zero-copy pattern where external parquet files should be registered without duplication.

### Direct implementation

Custom HTTP/FTP/API handlers without framework.

**Strengths:**
- Full control over download logic
- No framework learning curve
- Direct integration with DuckLake registration

**Limitations:**
- Reimplements schema inference, incremental state, retry logic
- Higher maintenance burden for heterogeneous sources
- No standardized error handling

## Decision

**dlt for ingestion, decoupled from storage.**

Architecture:
1. dlt sources handle download + schema normalization
2. dlt outputs parquet to designated staging directory (not DuckLake)
3. Custom registration layer reads parquet metadata and registers with DuckLake via `ducklake_add_data_files()`

This preserves dlt's strength in handling heterogeneous sources while maintaining marhar's zero-copy semantics.

## Implementation pattern

```python
# dlt pipeline outputs to staging
pipeline = dlt.pipeline(
    pipeline_name="geo_ingest",
    destination="filesystem",  # NOT ducklake
    dataset_name="staging"
)
pipeline.run(geo_source())

# Custom registration (separate step)
duckdb.sql("""
    CREATE TABLE catalog.dataset AS
    SELECT * FROM read_parquet('staging/*.parquet') WITH NO DATA;

    CALL ducklake_add_data_files('catalog', 'dataset', 'staging/*.parquet');
""")
```

## Rationale

The research showed dlt's DuckLake destination copies data rather than registering files.
Decoupling preserves both benefits:
- dlt handles the complexity of heterogeneous sources, incremental state, and schema evolution
- Zero-copy registration avoids storage duplication for large bioinformatics datasets

## Consequences

- Two-phase ingestion: dlt → staging → DuckLake registration
- Staging directory management (cleanup after successful registration)
- Schema validation between dlt output and DuckLake expected schema
- dlt state persists independently of DuckLake catalog state

## References

- ~/projects/dlt — Local dlt source
- ~/projects/dlt/dlt/destinations/impl/ducklake/ — DuckLake destination (copies, not zero-copy)
- Research: dlt suitability assessment (agent ad54ccb)
