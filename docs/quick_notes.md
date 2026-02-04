# High-Level Design Notes

This document captures the initial design and intent for a
metadata-driven Databricks ingestion pipeline implemented in Java.

## Overview

The system is composed of three primary components:

1. Metadata and configuration definitions
2. A fake data generator for producing test input
3. A REST-triggered Spark ingestion pipeline that writes Delta tables

The design emphasizes determinism, repeatability, and operational clarity
over maximum throughput or real-time guarantees.

---

## Metadata Files

Metadata files define the structure and ingestion behavior of each table.

Metadata is stored as CSV files and includes:
- Table name
- Column names (in order)
- Column delimiter
- Additional per-table ingestion settings (future)
-which tables will have delta updates and which ones will only have trunk/load
These files:
- Live in the repository for version control
- Are deployed to DBFS for runtime access
- Drive both data generation and ingestion behavior

---

## Configuration Files

Configuration files define environment-specific paths and locations.

Key configuration values include:
1. Drop zone location (raw input)
2. Staging area location (pre-Delta processing)
3. Metadata folder location
4. CSV input folder location

Configuration is externalized to avoid hardcoding environment details
into pipeline logic.

---

## Fake Data Generator

The fake data generator is a standalone Java application running in a databricks workflow.

Responsibilities:
1. Generate CSV files based on metadata schemas
2. Produce full-table extracts (no deltas initially)
3. Compress generated CSV files into ZIP archives
4. Write output to the DBFS drop zone

Operational behavior:
- Runs on a fixed interval (every 5 minutes using workflow schedulers)
- Intended to mimic a production environment.
- Does not attempt to simulate concurrent writes
- will deploy full table for trunk/load operations and delta for cases where deltas are used.
---

## Data Pipeline (Trunk / Load)

The data pipeline is implemented as a Spark job written in Java.

Key characteristics:
- Triggered via REST API
- Processes one table per invocation
- Reads metadata to determine parsing and schema rules
- Loads data from the drop zone into Delta tables

The pipeline is designed to be:
- Idempotent per table and input batch
- Safe to re-run for the same inputs
- Deterministic in output

---

## Triggering the Pipeline

Pipeline execution is initiated externally.

Two triggering mechanisms exist:
1. A separate Java application that calls the REST API for each table
   defined in metadata
2. The fake data generator, which triggers ingestion after data creation

Constraints:
- Only one trigger mechanism is expected to run at a time
- Concurrent triggers are explicitly out of scope for initial versions

---

## Delta Update Strategy

Delta table updates are driven via REST-triggered pipeline execution.

Initial strategy:
- Full loads per table
- append key will be taken from meta data lookup table.
- schema evolution is disabled and any schema changes will cause the run to fail. 

Future enhancements may include:
- MERGE-based delta updates
- Change detection using metadata
- Incremental ingestion strategies
