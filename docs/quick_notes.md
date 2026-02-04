# High-Level Design Notes

This document captures the initial high-level design for a
metadata-driven Databricks ingestion pipeline implemented in Java.

The intent of this project is to demonstrate common enterprise
data-engineering patterns (metadata-driven ingestion, REST-triggered jobs,
Delta Lake best practices) in a clear, maintainable way.

---

## Overview

The system is composed of four primary components:

1. Metadata definitions
2. Configuration management
3. A fake data generator
4. A REST-triggered Spark ingestion pipeline

The design favors determinism, idempotency, and operational clarity over
maximum throughput or real-time guarantees.

---

## Metadata Files

Metadata defines how each logical table is ingested and processed.

Metadata is stored as CSV files and includes:
- Table name
- Column names (ordered)
- Column delimiter
- Merge / business key(s)
- Load strategy (full refresh vs append / merge)

Metadata files:
- Are version-controlled in this repository
- Are deployed to DBFS for runtime access
- Drive both data generation and ingestion behavior

The pipeline is intentionally metadata-driven to minimize code changes when
new tables are added.

---

## Configuration Files

Configuration files define environment-specific locations and runtime behavior.

Key configuration values include:
1. Drop zone location (raw input data)
2. Staging area location (pre-Delta processing)
3. Metadata folder location
4. CSV input folder location

Configuration is externalized to:
- Avoid hardcoding environment details
- Enable consistent behavior across dev / test / prod
- Support Databricks job parameterization

---

## Fake Data Generator

The fake data generator is a standalone Java application.

Responsibilities:
1. Read table metadata definitions
2. Generate CSV files that conform to table schemas
3. Produce full-table extracts (no deltas initially)
4. Compress generated CSV files into ZIP archives
5. Write output to the DBFS drop zone

Operational behavior:
- Runs on a fixed interval (every 5 minutes)
- Intended for development and validation only
- Does not attempt to simulate concurrent writers
- Assumes schema correctness based on metadata

---

## Data Pipeline (Truncate / Load)

The ingestion pipeline is implemented as a Spark job written in Java.

Key characteristics:
- Triggered via REST API
- Processes one table per invocation
- Reads metadata to determine parsing, schema, and load strategy
- Loads data from the drop zone into Delta tables

Initial load strategy:
- Full refresh (truncate / load) per table
- Deterministic output for the same inputs
- Safe re-runs without producing duplicate data

Future enhancements may introduce incremental or merge-based loads.

---

## Triggering the Pipeline

Pipeline execution is initiated externally via REST.

Two triggering mechanisms exist:
1. A standalone Java application that calls the REST API for each table
   defined in metadata
2. The fake data generator, which triggers ingestion after data generation

Constraints:
- Only one trigger mechanism is expected to run at a time
- Concurrent pipeline executions are explicitly out of scope for early versions

This design mirrors common enterprise ingestion orchestration patterns while
keeping operational complexity low.

---

## Delta Update Strategy

Delta Lake is used as the primary storage format.

Initial strategy:
- Full refresh loads using overwrite semantics
- Partitioning applied where appropriate
- No MERGE logic in early iterations

Planned future enhancements:
- Metadata-driven MERGE (upsert) support
- Business-key-based change detection
- Incremental ingestion strategies
- Late-arriving data handling

---

## Non-Goals (Initial Versions)

The following are explicitly out of scope for early iterations:
- Concurrent ingestion of the same table
- Streaming ingestion
- Schema evolution automation
- Exactly-once semantics across failures

These may be explored in later phases once the core pipeline is stable.

---

## Summary

This project is intentionally designed to resemble a real-world
enterprise Databricks ingestion framework while remaining simple enough
to reason about, test, and extend.

The focus is on:
- Clear separation of concerns
- Metadata-driven behavior
- Operational predictability
- Production-style documentation and structure
