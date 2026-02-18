# Metadata store decision

## Status

Proposed

## Context

Omicslake needs to track dataset metadata including:
- Source provenance (URL, accession, DOI, license, download timestamp)
- Schema versions for each dataset
- Lineage (which workflow produced which output, tool versions)
- Changes over time (when datasets are re-ingested or updated)

The metadata store must support versioning to track schema evolution and enable reproducibility.

## Options evaluated

### Dolt / Doltgres

Git-like versioning for SQL databases.

**Strengths:**
- Native row-level versioning with branching and merging
- Diff/merge capabilities for collaborative schema evolution
- Full audit trail at row level
- SQL-compatible interface (MySQL or Postgres wire protocol)

**Critical limitation:**
Both Dolt and Doltgres are server-centric, not embeddable libraries.
They must run as separate processes with network connections.
There is no embedded SQLite-like mode.
Python integration requires spawning a server or connecting over TCP.

### SQLite + git

Embedded database with file-level version control.

**Strengths:**
- Embedded operation; no separate process
- Zero dependencies; integrates trivially with Python
- Git provides versioning at file level
- Schema migrations via standard patterns (Alembic, raw SQL)
- Well-understood tooling

**Limitations:**
- No row-level versioning; changes tracked at file/commit level
- Merge conflicts require manual resolution
- No native diff of table contents (requires tooling)

## Decision

**SQLite + git** for metadata storage.

## Rationale

Dolt's advantages (row-level versioning, merge conflict resolution) matter most for collaborative data modification scenarios with concurrent writers.
Omicslake's metadata tracking has different characteristics:
- Single writer (the ingestion pipeline)
- Schema changes are rare and can be manually coordinated
- Audit trail needs are satisfied by git commit history
- Embedding without server process is strongly preferred

SQLite + git provides:
- Embedded operation matching Dagster's local-first development model
- Git versioning aligns with existing marhar patterns (git-tracked .ducklake files)
- Standard Python tooling (sqlite3, sqlalchemy) without framework overhead

## Metadata schema

```sql
-- Dataset provenance
CREATE TABLE datasets (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    source_url TEXT,
    accession TEXT,
    doi TEXT,
    license TEXT,
    format TEXT,  -- h5ad, rds, csv, parquet, etc.
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP
);

-- Ingestion runs
CREATE TABLE ingestion_runs (
    id TEXT PRIMARY KEY,
    dataset_id TEXT REFERENCES datasets(id),
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    status TEXT,  -- pending, running, success, failed
    tool_versions JSON,  -- {"omicsio": "1.2.0", "dlt": "0.4.0"}
    output_path TEXT,
    row_count INTEGER,
    checksum TEXT
);

-- Schema versions
CREATE TABLE schema_versions (
    id TEXT PRIMARY KEY,
    dataset_id TEXT REFERENCES datasets(id),
    version INTEGER,
    schema JSON,  -- column names, types, nullability
    created_at TIMESTAMP,
    migration_notes TEXT
);
```

## Consequences

- Metadata database is a single SQLite file, git-tracked
- Schema migrations managed via numbered SQL files or Alembic
- No real-time collaboration; changes coordinated through git workflow
- Lineage queries use standard SQL joins
- Dagster can emit metadata to SQLite via custom I/O manager

## References

- Research: dolt suitability assessment (agent a0c2262)
- biomni patterns: identified metadata gaps (no version tracking, no provenance)
