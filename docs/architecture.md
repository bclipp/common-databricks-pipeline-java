# Architecture Overview

This document describes the high-level architecture for the
metadata-driven Databricks ingestion pipeline implemented in Java.

The goal is to provide a clear, production-style architectural view
without tying the design to a specific environment or dataset.

---

## Architectural Goals

The architecture is designed to:

- Support metadata-driven ingestion with minimal code changes
- Enable deterministic, repeatable batch processing
- Mirror common Databricks enterprise ingestion patterns
- Be simple to reason about and operate
- Serve as a foundation for future enhancements (incremental loads, merges)

---

## System Components

The system consists of the following logical components:

### 1. Metadata Layer
- Defines table schemas and ingestion behavior
- Stored as CSV files
- Version-controlled in Git
- Deployed to DBFS for runtime access

### 2. Configuration Layer
- Defines environment-specific paths and settings
- Externalized from code
- Passed via config files or Databricks job parameters

### 3. Fake Data Generator
- Produces input data based on metadata definitions
- Writes data to a raw drop zone in DBFS
- Used for development and validation only

### 4. Ingestion Pipeline
- Spark job written in Java
- Triggered via REST API
- Reads metadata and configuration at runtime
- Loads data into Delta Lake tables

---

## High-Level Data Flow

Metadata (CSV, Git/DBFS) -> Fake Data Generator -> DBFS Drop Zone (Raw Files) -> REST-Triggered Spark Job -> Delta Lake Tables

Each pipeline invocation processes exactly one logical table.

---

## Execution Model

- Batch-oriented ingestion
- One table per pipeline run
- Explicit REST-based triggering
- No implicit scheduling within the pipeline itself

This mirrors common enterprise patterns where orchestration is handled
externally (Airflow, Control-M, custom services, etc.).

---

## Failure and Recovery Model

The system is designed for safe re-execution:

- Full refresh (truncate/load) semantics in early versions
- Deterministic output for identical inputs
- Failed runs do not corrupt existing Delta tables
- Re-running a failed job produces the same result

Checkpointing and incremental recovery are deferred to later phases.

---

## Observability and Diagnostics

The architecture assumes basic observability features:

- Structured logging at job and step boundaries
- Record counts at major pipeline stages
- Clear failure signals via job exit codes

Advanced monitoring (metrics export, alerts) is out of scope initially.

---

## Scalability Considerations

Initial versions prioritize clarity over scale.

Planned future enhancements may include:
- Incremental ingestion
- MERGE-based updates
- Partition-aware processing
- Concurrency controls

The architecture intentionally leaves room for these improvements
without requiring major redesign.

---

## Summary

This architecture represents a simplified but realistic enterprise
Databricks ingestion framework.

It balances:
- Practicality over theoretical completeness
- Explicit behavior over hidden magic
- Design clarity over premature optimization