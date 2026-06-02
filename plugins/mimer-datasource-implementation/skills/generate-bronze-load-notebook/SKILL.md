---
name: generate-bronze-load-notebook
description: Generate a Databricks Bronze load notebook for ingesting datasource files or raw source extracts into a Bronze Delta table.
---

# Generate Bronze Load Notebook

Use this skill when the user asks Cursor to create or update a Bronze load notebook for a Mimer datasource.

## Output Contract

Create a valid Databricks-compatible `.ipynb` notebook.

Output path:

```text
Datasources-Science/<datasource>/bronze/Load_Bronze_<datasource>.ipynb
```

The generated notebook must:
- Read raw datasource files or extracts.
- Preserve source structure as much as possible.
- Add mandatory metadata columns.
- Write to a Bronze Delta table.
- Avoid business transformations.

## Required Inputs

Infer where possible:
- datasource
- input path
- storage account
- catalog
- environment
- Bronze database
- Bronze table name
- file format
- schema or schema inference rule

## Defaults

```python
bronze_database = "dbsymphogen_scientific_ingested_data"
bronze_table = "<datasource>_bronze"
write_mode = "append"
```

## Required Widgets

```python
dbutils.widgets.text("environment", "")
dbutils.widgets.text("datasource", "")
dbutils.widgets.text("catalog", "")
dbutils.widgets.text("storage_account", "")
dbutils.widgets.text("input_path", "")
dbutils.widgets.text("load_id", "")
```

## Required Notebook Structure

1. Databricks marker and imports.
2. Widgets and parameter resolution.
3. Resolve source and target paths.
4. Read raw source data.
5. Add metadata columns.
6. Validate source read and required columns.
7. Write Bronze Delta output.
8. Log row counts, load id, and output table.

## Required Metadata Columns

Add:

```python
.withColumn("_ingestion_timestamp", F.current_timestamp())
.withColumn("_ingestion_date", F.current_date())
.withColumn("_source_file", F.input_file_name())
.withColumn("_load_id", F.lit(load_id))
.withColumn("_environment", F.lit(environment))
```

## Write Rules

Use Delta:

```python
(df.write
    .format("delta")
    .mode("append")
    .option("mergeSchema", "true")
    .saveAsTable(f"{bronze_database}.{bronze_table}"))
```

Overwrite is allowed only when the user explicitly asks for a controlled rebuild.

## Forbidden in Bronze

Do not:
- Apply business logic.
- Aggregate records.
- Build curated output.
- Deduplicate aggressively.
- Remove source columns without explicit instruction.
- Write to Silver database or Silver path.

## Validation Rules

Verify:
- Input path is resolved.
- Source read produces a DataFrame.
- Required source columns exist when supplied.
- Metadata columns exist.
- Target table is Bronze.
- Row count and target table are logged.

## Final Response

Return only:
- Created or updated file path.
- Datasource name.
- Target Bronze table.
- Assumptions made.
