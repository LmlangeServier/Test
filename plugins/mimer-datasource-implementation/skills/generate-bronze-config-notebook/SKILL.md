---
name: generate-bronze-config-notebook
description: Generate a Databricks .ipynb notebook that creates or evolves a Mimer Bronze Delta table for a datasource.
---

# Generate Bronze Config Notebook

Use this skill when the user asks Cursor to create, update, or validate a Databricks Bronze configuration notebook from a sample dataset, SQL DDL, Python column list, datasource configuration, or table definition.

## Output Contract

Create a valid Databricks-compatible Jupyter notebook JSON file.

Output path:

```text
Datasources-Science/<datasource>/configs/Create_Bronze_<datasource>.ipynb
```

The generated notebook must:
- Use `nbformat` 4.
- Contain code cells only.
- Start with `# Databricks notebook source`.
- Use `# COMMAND ----------` between logical Databricks cells.
- Use Databricks SQL magic cells for SQL sections.

## Defaults

Use these defaults unless explicitly supplied:

```python
database = "dbsymphogen_scientific_ingested_data"
table_name = "<datasource>_bronze"
partition_columns = ["batch_id"]
delta_isolation_level = "WriteSerializable"
```

Storage root pattern:

```python
location = f"abfss://bronze@{storage_account}.dfs.core.windows.net/{datasource}"
```

Do not hardcode storage account names or environment names.

## Extraction Rules

Extract:
- `datasource` from user text, folder path, source filename, table name, or config.
- `database` from SQL `CREATE DATABASE`, `USE`, config, or user text.
- `table_name` from SQL DDL, config, Python variables, or user text.
- `columns` from SQL DDL, Python tuples, JSON config, or inferred source schema.
- `partition_columns` from config, explicit input, or defaults.
- `location` from config or generated storage pattern.
- Delta table properties, especially `delta.isolationLevel`.

Preserve exact identifiers unless the user explicitly requests renaming.

## Required Notebook Cells

1. Databricks marker and imports.
2. Widgets and parameter resolution.
3. Catalog/database setup.
4. Variables for datasource, database, table name, partition columns, location, and Delta properties.
5. Helper functions:
   - `table_exists`
   - `existing_columns`
   - `create_or_evolve_delta_table`
6. Datasource column list.
7. Create or evolve the Bronze Delta table.
8. Validation and summary logging.

## Mandatory Metadata Columns

Include these columns unless already present:

```text
_ingestion_timestamp TIMESTAMP
_ingestion_date DATE
_source_file STRING
_load_id STRING
_environment STRING
```

## Validation Rules

Before finalizing the notebook, verify:
- The notebook JSON is valid.
- `nbformat` is 4.
- The notebook name is `Create_Bronze_<datasource>`.
- The database is Bronze: `dbsymphogen_scientific_ingested_data` unless overridden.
- The table defaults to `<datasource>_bronze`.
- The location uses the Bronze storage pattern.
- Metadata columns exist.
- No Silver database, Silver path, or Silver table naming appears by mistake.

## Final Response

When complete, return only:
- Created or updated file path.
- Datasource name.
- Table name.
- Any assumptions made.
