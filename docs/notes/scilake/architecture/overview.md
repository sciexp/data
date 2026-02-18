# Scilake architecture

Scilake is a data provenance and reproducible workflow system for scientific data engineering.
It bridges computational modeling pipelines and data-driven web applications through DuckDB, DuckLake, SQLMesh, and parquet.

## Scope

Scilake addresses three interconnected problems.

Heterogeneous format ingestion: source datasets arrive as h5ad (AnnData), RDS (R objects), mtx.gz with metadata CSVs, pickle files, TSV/CSV tables, and FTP archives.
Each requires format-specific handling before standardization to parquet.

Lineage and provenance tracking: existing parquet files lack documented lineage.
Scilake traces data back to original sources (GEO accessions, Zenodo DOIs, S3 URIs) and encodes reproducible workflows.

DuckLake catalog generation: the output layer produces parquet files registered with DuckLake catalogs, published to HuggingFace Hub for downstream query via httpfs and the `hf://` protocol.

## Architecture layers

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          Publish Layer                                   │
│    HuggingFace Hub (git-lfs) <- DuckLake catalog + parquet               │
├─────────────────────────────────────────────────────────────────────────┤
│                          Transformation Layer                            │
│    SQLMesh models (staging, analytics) <- declarative SQL on DuckLake    │
├─────────────────────────────────────────────────────────────────────────┤
│                          Catalog Layer                                   │
│    DuckLake registration (zero-copy) <- parquet files                    │
├─────────────────────────────────────────────────────────────────────────┤
│                          Metadata Layer                                  │
│    SQLite (lineage, schema, provenance) + git versioning                 │
├─────────────────────────────────────────────────────────────────────────┤
│                          Convert Layer                                   │
│    omicsio (h5ad->parquet) | RDS->AnnData->parquet | dlt normalization   │
├─────────────────────────────────────────────────────────────────────────┤
│                          Ingest Layer                                    │
│    dlt sources (S3, HTTP, API) | custom sources (GEO FTP, Kaggle)       │
├─────────────────────────────────────────────────────────────────────────┤
│                          Orchestration Layer                             │
│    Dagster assets (lineage, scheduling, retry)                           │
└─────────────────────────────────────────────────────────────────────────┘
```

## Layer summary

| Layer | Responsibility | Primary tools | Inputs | Outputs |
|-------|---------------|---------------|--------|---------|
| Orchestration | Scheduling, retry, lineage | Dagster | Pipeline definitions | Coordinated execution |
| Ingest | Download, schema inference | dlt, custom handlers | URLs, APIs | Raw files |
| Convert | Format standardization | omicsio, rds2anndata, DuckDB | Raw files | Parquet |
| Metadata | Provenance tracking | SQLite + git | Ingestion events | Lineage records |
| Catalog | DuckLake registration | DuckDB + ducklake | Parquet files | Queryable catalog |
| Publish | Distribution | huggingface_hub | Catalog + parquet | hf:// URLs |

## Ingest layer

This layer downloads data from heterogeneous sources and performs initial schema inference.

dlt sources handle most cases: the `filesystem` source covers S3, GCS, and local files, `rest_api` covers Zenodo and CellxGene APIs, and custom resources handle GEO FTP and Kaggle API access.
The output is raw files in a staging directory with dlt state tracking incremental updates.

## Convert layer

This layer transforms source formats to standardized parquet.

The conversion paths are: h5ad to parquet via omicsio (preserving AnnData semantics), RDS to AnnData via rds2anndata then to parquet via omicsio, mtx.gz with CSV to AnnData via scanpy assembly then to parquet via omicsio, and CSV/TSV/pickle directly to parquet via DuckDB.
The output is parquet files with consistent schema conventions.

## Metadata layer

This layer tracks provenance, lineage, and schema versions in SQLite.

The schema includes three tables: `datasets` for source provenance (URL, accession, DOI, license), `ingestion_runs` for execution history (timestamps, tool versions, checksums), and `schema_versions` for schema evolution tracking.
The SQLite file is git-tracked, with changes committed alongside ingestion runs.

## Catalog layer

This layer registers parquet files with DuckLake using a zero-copy pattern.

```sql
-- Create table structure without loading data
CREATE TABLE catalog.dataset AS
SELECT * FROM read_parquet('path/*.parquet') WITH NO DATA;

-- Register files (metadata only, no data copy)
CALL ducklake_add_data_files('catalog', 'dataset', 'path/*.parquet');
```

The output is a DuckLake catalog consisting of `.ducklake` metadata and registered parquet files.

## Publish layer

This layer publishes DuckLake catalogs to HuggingFace Hub for remote query access.
Git-lfs tracks `.parquet` and `.ducklake` files, which are pushed to a HuggingFace dataset repo.
Downstream consumers query via `hf://datasets/org/repo/path.ducklake` using the DuckDB httpfs extension.

## Layer boundaries

Each layer communicates through well-defined interfaces.

| Boundary | Interface | Format |
|----------|-----------|--------|
| Ingest to Convert | File paths | Raw files in staging |
| Convert to Catalog | File paths | Parquet in output dir |
| All to Metadata | Events | SQLite inserts |
| Catalog to Publish | Directory | DuckLake catalog tree |

Dagster assets encode these dependencies explicitly, enabling partial re-execution (re-run convert without re-ingest), lineage queries (which source produced which output), and failure isolation (ingest failure does not corrupt the catalog).

## Format landscape

Source formats requiring conversion to parquet:

| Format | Conversion path | Tool | Examples |
|--------|-----------------|------|----------|
| h5ad | h5ad to AnnData to Arrow to parquet | omicsio | NeurIPS BMMC, scPerturb, Replogle |
| RDS | RDS to AnnData to parquet | rds2anndata + omicsio | ZSCAPE, Weinreb LARRY |
| mtx.gz + CSV | assemble to AnnData to parquet | scanpy + omicsio | GEO raw counts |
| CSV/TSV | direct load | DuckDB | DepMap, BindingDB, eQTL tables |
| pickle | unpickle to DataFrame to parquet | pandas/DuckDB | genebass, gwas_catalog |
| FTP tarball | extract then route by contents | custom | GEO supplementary |

## Key architectural decisions

| Decision | Choice | Alternatives considered | Rationale |
|----------|--------|------------------------|-----------|
| Workflow orchestration | Dagster | Flyte v2, Temporal | Asset-centric lineage; Python-native; no external cluster required |
| Ingestion tooling | dlt (decoupled) | Direct HTTP/FTP | Schema inference; incremental loading; heterogeneous source support |
| DuckLake registration | Zero-copy | dlt DuckLake destination | Preserves storage efficiency; no data duplication |
| Transformations | SQLMesh | dbt, raw SQL views | Incremental processing; DuckLake integration; declarative models |
| Metadata store | SQLite + git | Dolt, Doltgres | Embeddable; no server process; git provides versioning |
| Parquet conversion | omicsio | Direct DuckDB | omicsio handles AnnData semantics (X, obs, var, layers) |

ADR-style documentation for each decision is in the `decisions/` directory.

## Data flow

```
Source (GEO/Zenodo/S3/API)
    |
    v [Ingest: dlt or custom]
Staging (raw files)
    |
    v [Convert: omicsio/rds2anndata/DuckDB]
Parquet files (standardized)
    |
    |---> [Metadata: SQLite] --- lineage, schema, provenance
    |
    v [Catalog: DuckLake zero-copy registration]
DuckLake catalog (.ducklake + .parquet)
    |
    v [Publish: huggingface_hub]
HuggingFace Hub (hf://datasets/sciexp/...)
    |
    v [Query: httpfs]
DuckDB client (DuckDB-WASM or server)
```

## References

- dagster-sqlmesh: Dagster + SQLMesh integration example
- ducklake-dagster-crypto-pipeline: DuckLake + dlt + Dagster integration example
- sciexp-fixtures: working DuckLake reference implementation
- SQLMesh DuckLake integration: https://sqlmesh.readthedocs.io/en/stable/integrations/engines/duckdb/#ducklake-catalog-example
- Tobiko DuckLake tutorial: https://www.tobikodata.com/blog/ducklake-sqlmesh-tutorial-a-hands-on
