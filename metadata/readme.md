
# Metadata

This directory contains metadata definitions that drive ingestion behavior
for the Databricks pipeline.

Metadata files define table schemas, parsing rules, and load strategies.
They are version-controlled and deployed to DBFS as part of the Databricks
bundle.

Changes to metadata should be reviewed with the same rigor as code changes.

Each logical table is represented by a single line in the metadata CSV file.