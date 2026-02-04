# Pipeline Design

This document describes the behavior and execution flow of the
metadata-driven ingestion pipeline.

It focuses on *what the pipeline does* and *how it behaves at runtime*,
rather than overall system architecture.

---

## Pipeline Responsibilities

The pipeline is responsible for:

- Reading metadata definitions for a target table
- Locating and parsing raw input data
- Applying schema and load rules defined in metadata
- Writing data to Delta Lake in a deterministic manner

Each pipeline execution handles exactly one logical table.

---

## Triggering Mechanism

The pipeline is triggered via a REST API.

Trigger inputs include:
- Target table name
- Environment (dev / test / prod)
- Optional runtime parameters (dates, paths)

The pipeline does not manage its own scheduling and does not assume
continuous execution.

---

## Pipeline Execution Flow

High-level steps:

1. Validate request parameters
2. Load configuration for the target environment
3. Read metadata for the target table
4. Locate raw input files in the drop zone
5. Parse input data using metadata-defined rules
6. Apply load strategy (truncate/load initially)
7. Write output to Delta Lake
8. Emit logs and completion status

---

## Metadata Usage

Metadata drives the following pipeline behaviors:

- CSV parsing rules (delimiter, column order)
- Expected schema
- Business / merge keys
- Load strategy (full refresh vs future incremental)

This design allows new tables to be onboarded without code changes.

---

## Load Strategies

### Initial Strategy: Truncate / Load

- Existing Delta table data is fully replaced
- Output is deterministic for a given input set
- Safe to re-run for the same inputs

This strategy is intentionally simple and robust for early iterations.

### Future Strategies (Planned)

- Append-only loads
- MERGE-based upserts
- Incremental ingestion based on metadata rules

---

## Delta Lake Interaction

- Delta Lake is the system of record
- Writes use overwrite semantics for full refreshes
- Partitioning may be applied based on table configuration
- Schema enforcement is expected at write time

Schema evolution is not automated in early versions.

---

## Error Handling

Pipeline failures fall into two categories:

### Configuration / Metadata Errors
- Missing or invalid metadata
- Invalid configuration values
- Fail fast with clear error messages

### Data Errors
- Malformed input files
- Schema mismatches
- Handled by failing the job (initially)

Quarantine and partial success handling may be added later.

---

## Idempotency Guarantees

The pipeline provides the following guarantees:

- Re-running a job for the same table and inputs produces the same output
- No duplicate data is introduced by retries
- Partial writes do not leave tables in an inconsistent state

These guarantees rely on full refresh semantics in early versions.

---

## Non-Goals (Initial Versions)

The following are intentionally out of scope:
- Streaming ingestion
- Concurrent processing of the same table
- Exactly-once guarantees across job failures
- Automated schema evolution

---

## Summary

The pipeline design prioritizes:

- Predictable behavior
- Clear operational semantics
- Metadata-driven flexibility
- Enterprise-style execution patterns

It is intentionally conservative in early iterations to establish
a stable foundation for future enhancements.