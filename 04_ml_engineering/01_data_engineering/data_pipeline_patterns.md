---
layer: 04_ml_engineering
type: engineering
tool: general
status: growing
tags: [data-engineering, pipelines, etl, batch-processing, streaming]
created: 2026-03-05
---

# Data Pipeline Patterns

## Purpose

Data pipelines move, transform, and validate data between systems. In ML contexts they serve two roles: preparing training data (offline pipelines) and computing features at inference time (online pipelines). Choosing the wrong pattern—e.g., batch when the model needs real-time features—introduces latency or staleness that undermines the model's value in production.

## Architecture

### Batch vs. Stream Processing

**Batch processing** operates on bounded datasets at scheduled intervals (hourly, daily). Tools like Apache Spark, dbt, and Hadoop excel here. Batch jobs are easier to debug, reprocess, and version than streaming pipelines. They are appropriate when the model tolerates stale features—e.g., a daily recommendation refresh or an overnight fraud score update.

**Stream processing** operates on unbounded, continuously arriving data. Apache Kafka (message bus), Apache Flink, and Spark Structured Streaming are common choices. Stream processing enables real-time features (e.g., "transactions in the last 5 minutes") critical to fraud detection, dynamic pricing, or personalisation. The cost is operational complexity: state management, exactly-once semantics, and late-data handling are non-trivial.

**Lambda architecture** runs batch and stream pipelines in parallel, merging their outputs. The batch layer is the source of truth; the stream layer fills the latency gap. Operationally expensive to maintain two codebases.

**Kappa architecture** replaces the batch layer with a replayable stream (e.g., Kafka with long retention). Simpler to maintain; requires the stream processor to handle backfill efficiently.

### ETL vs. ELT

**ETL (Extract → Transform → Load)**: Transformations happen before data lands in the warehouse. Historically necessary when storage was expensive. Enforces clean data contracts but makes schema evolution painful.

**ELT (Extract → Load → Transform)**: Raw data lands first; transformations run inside the warehouse using SQL (dbt is the dominant tool). Preferred in modern data stacks because raw data is preserved for reprocessing, transformations are versioned as SQL, and cloud warehouses (BigQuery, Snowflake, Redshift) make in-place transformation cheap.

In ML pipelines, ELT is typically preferred: preserve raw signals, apply feature transformations as late as possible, and version each transformation as a discrete step.

### Data Formats

| Format | Use case | Strengths | Weaknesses |
|---|---|---|---|
| **Parquet** | Columnar analytics, training data | Compressed, predicate pushdown, schema embedded | Not human-readable, write-once |
| **Avro** | Row-level streaming, Kafka messages | Schema evolution, compact binary | Less efficient for analytical queries |
| **JSON / JSONL** | Interchange, log ingestion | Human-readable, flexible schema | Verbose, slow for large datasets |
| **TFRecord** | TensorFlow training data | Efficient sequential read for TF | TF-specific, not interoperable |
| **Arrow / Feather** | In-memory, zero-copy sharing | Extremely fast read, language-agnostic | Not suitable for long-term storage |

For ML training data at scale, **Parquet** is the standard. JSONL is acceptable for small datasets or when schema is truly dynamic.

### OLTP vs. OLAP

**OLTP (Online Transaction Processing)** databases (PostgreSQL, MySQL) are optimised for high-throughput, low-latency reads and writes of individual rows. They are the systems of record for application data.

**OLAP (Online Analytical Processing)** databases (BigQuery, Redshift, ClickHouse) are columnar stores optimised for aggregating millions of rows. They are the source for feature engineering and model training.

ML pipelines almost always involve a data movement step from OLTP → OLAP. This is where schema drift, join complexity, and data quality issues typically surface.

### Data Warehouse vs. Data Lake vs. Lakehouse

- **Data warehouse**: Structured, schema-on-write, SQL-queryable, governed. Ideal for clean, curated datasets. High reliability, lower flexibility.
- **Data lake**: Raw data in object storage (S3, GCS) in any format. High flexibility, low governance. Tends to become a "data swamp" without discipline.
- **Lakehouse**: Adds ACID transactions, schema enforcement, and query optimisation on top of object storage via a table format (Delta Lake, Apache Iceberg, Apache Hudi). Combines warehouse reliability with lake flexibility. Increasingly the default for ML teams.

### Orchestration Tools

**Apache Airflow**: DAG-based orchestration, Python-defined workflows, rich ecosystem of operators. Dominant in data engineering. Operationally heavy; scheduler is a single point of failure without careful configuration.

**Prefect**: More Pythonic API than Airflow, native support for dynamic workflows, better local development experience. Growing adoption.

**Dagster**: Asset-oriented orchestration; models data assets explicitly rather than tasks. Excellent observability and lineage tracking. Strong fit for ML pipelines where data assets are first-class.

**dbt**: SQL-only transformation tool; not a general orchestrator but ubiquitous for the T in ELT.

### Data Quality Validation

Data quality failures are the most common root cause of silent model degradation. Validate at every stage boundary:

- **Schema validation**: Column names, types, nullability constraints. Tools: Great Expectations, Pydantic, Pandera.
- **Distribution checks**: Monitor mean, std, min/max, quantiles, and null rates against a reference window. Alert on >2σ deviation.
- **Referential integrity**: Foreign keys, join key overlap between train and serving data.
- **Freshness checks**: Assert that the most recent row is within an expected time window.

## Implementation Notes

- Treat pipeline code as production software: version control, unit tests for transformations, integration tests with representative data samples.
- Design pipelines to be idempotent: re-running a pipeline on the same input should produce the same output without side effects (e.g., duplicate rows).
- Partition training data by date; avoid `SELECT *` on full history tables. Time-partitioned reads are dramatically cheaper in cloud warehouses.
- Log pipeline run metadata (row counts, null rates, timing) to a metadata store for retrospective debugging.

## Trade-offs

| Decision | Trade-off |
|---|---|
| Batch vs. stream | Simplicity vs. feature freshness |
| ETL vs. ELT | Control vs. flexibility |
| Parquet vs. JSON | Performance vs. readability |
| Strict schema vs. schemaless | Reliability vs. agility |
| Warehouse vs. lakehouse | Maturity vs. flexibility |

## References

- Kleppmann, M. (2017). *Designing Data-Intensive Applications*. O'Reilly. Chapters 10–11.
- Reis, J. & Housley, M. (2022). *Fundamentals of Data Engineering*. O'Reilly.
- dbt Labs. *dbt Documentation*. docs.getdbt.com.

## Links
- [[feature_store|Feature Store]]
- [[ml_lifecycle|ML Lifecycle]]
- [[dataset_versioning|Dataset Versioning and Lineage]]
- [[ml_system_design|ML System Design]]
