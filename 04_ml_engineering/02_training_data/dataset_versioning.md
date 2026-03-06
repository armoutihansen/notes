---
layer: 04_ml_engineering
type: engineering
tool: dvc
status: growing
tags: [dataset-versioning, dvc, data-lineage, reproducibility, mlops]
created: 2026-03-05
---

# Dataset Versioning and Lineage

## Purpose

Dataset versioning is the practice of treating data as a first-class versioned artifact—the same way source code is tracked in Git. Without it, reproducing a model trained three months ago is often impossible: the training data may have been overwritten, the preprocessing scripts may have changed, and the exact split used for evaluation is unknown. Dataset versioning enables reproducibility, auditability, and safe iteration.

## Architecture

### Why Dataset Versioning Matters

The core problem is that ML experiments have two variable inputs: code and data. Git tracks code reliably; without a parallel system for data, the full state of an experiment cannot be reconstructed. Consequences include:

- **Irreproducible results**: A reported accuracy cannot be verified because the exact training set is unknown.
- **Silent regression**: A pipeline change alters feature engineering logic; without data versioning, there is no diff to inspect.
- **Regulatory exposure**: In regulated industries (finance, healthcare), auditors may require proof that a model was trained on exactly the data that was reviewed.
- **Debugging difficulty**: Investigating why model performance degraded requires knowing which data each model version was trained on.

### DVC (Data Version Control)

[DVC](https://dvc.org) extends Git to version large data files and ML artifacts. It stores metadata in Git and the actual data in a configurable remote storage backend (S3, GCS, Azure Blob, SSH, local).

**Core mechanism:**

1. `dvc add data/train.parquet` computes a content hash of the file and writes a `.dvc` file (a small pointer):
   ```
   outs:
   - md5: a3f8b2c1d9e4...
     size: 1073741824
     path: data/train.parquet
   ```
2. The `.dvc` pointer file is committed to Git. The data file itself is added to `.gitignore`.
3. `dvc push` uploads the actual data to the remote.
4. `dvc pull` on any machine with Git access downloads exactly the version of the data corresponding to the current Git commit.

**DVC pipelines** (`dvc.yaml`) define directed acyclic graphs of data transformation stages:

```yaml
stages:
  preprocess:
    cmd: python preprocess.py --input data/raw --output data/processed
    deps:
      - preprocess.py
      - data/raw
    outs:
      - data/processed
  featurise:
    cmd: python featurise.py
    deps:
      - featurise.py
      - data/processed
    outs:
      - data/features
```

`dvc repro` executes only the stages whose inputs have changed—analogous to `make`. This ensures the pipeline is always in a consistent state relative to the code.

**Experiment tracking**: `dvc exp run` creates a new experiment, captures parameters (`params.yaml`), metrics, and plots alongside the data version and code commit. All experiments are browsable with `dvc exp show`.

### Data Lineage Tracking

Data lineage answers: where did this dataset come from, what transformations were applied, and who used it? Lineage is essential for:

- **Debugging**: Tracing a feature's value back to its raw source record.
- **Impact analysis**: Understanding which models are affected when a data source changes.
- **Compliance**: Demonstrating data provenance for GDPR/CCPA data deletion requests.

Lineage can be captured at different granularities:

- **Dataset-level lineage** (DVC, dbt): Which datasets were consumed and produced by each pipeline stage.
- **Column-level lineage** (dbt, OpenLineage, Marquez): Which source columns contributed to each output column.
- **Row-level lineage**: Which source records contributed to each output record. Expensive to track; only necessary for compliance-critical applications.

**OpenLineage** is an emerging open standard for lineage metadata, supported by Airflow, dbt, Spark, and Flink. It emits lineage events to a metadata backend (Marquez) at job run time.

### Dataset Cards and Documentation

A dataset card (modelled after Hugging Face's Dataset Cards and Google's Data Cards) is a structured document that accompanies every published dataset version. Minimum contents:

- **Motivation**: Why was the dataset created? What task is it intended for?
- **Composition**: How many examples? What labels? What languages/modalities? How were they collected?
- **Collection process**: What consent was obtained? What sampling strategy was used?
- **Preprocessing**: What cleaning, filtering, and transformations were applied?
- **Known limitations**: What biases exist? What populations are under-represented?
- **Versioning history**: What changed between versions?

Dataset cards reduce the risk of misuse and make it possible for new team members to evaluate whether a dataset is appropriate for their task.

### Versioning Raw vs. Processed Data

Best practice is to version **both** raw and processed data independently:

- **Raw data** should be treated as immutable. Never modify raw data in place; append-only or copy-on-write semantics ensure that all downstream artifacts can be regenerated from source.
- **Processed data** (cleaned, featurised, split) is derived from raw data plus code. Its version is determined by the raw data version + the processing code version. DVC captures this dependency graph automatically via pipeline stages.

A common pattern: raw data lives in a versioned `data/raw/v{n}/` directory, processed data in `data/processed/v{n}/`. Each "version bump" corresponds to a change in either source data or processing logic.

### Integration with Experiment Tracking

Dataset versions should be linked to experiment runs in the experiment tracker:

- **MLflow**: Log `mlflow.log_param("data_version", "v3")` and the dataset hash.
- **Weights & Biases**: Use `wandb.Artifact` to log datasets and link them to runs.
- **DVC + Experiments**: The DVC experiment mechanism handles this natively—each experiment captures the full state of data and code.

The goal is that given any model artifact or experiment ID, the exact training data can be reproduced in one command.

## Implementation Notes

- Commit `.dvc` pointer files to Git immediately after `dvc add`; never commit the data files themselves.
- Use a shared DVC remote (S3 bucket with versioning enabled) accessible to all team members and CI/CD pipelines.
- Add `dvc repro` to the CI pipeline to detect when code changes break the data pipeline before they reach production.
- For large datasets (>100 GB), use DVC's chunked transfer and consider keeping only recent versions in the hot remote; archive older versions to cold storage.

## Trade-offs

| Decision | Trade-off |
|---|---|
| Git + DVC vs. database versioning | Simplicity vs. row-level granularity |
| Versioning raw only vs. raw + processed | Storage savings vs. reproducibility speed |
| Full dataset versions vs. incremental patches | Storage cost vs. lineage complexity |
| DVC (open-source) vs. managed (Pachyderm, LakeFS) | Control vs. operational burden |

## References

- DVC documentation: dvc.org/doc
- Huyen, C. (2022). *Designing Machine Learning Systems*. O'Reilly. Chapter 4.
- Gebru, T. et al. (2018). *Datasheets for Datasets*. arXiv:1803.09010.
- OpenLineage specification: openlineage.io

## Links
- [[data_pipeline_patterns|Data Pipeline Patterns]]
- [[feature_store|Feature Store]]
- [[ml_lifecycle|ML Lifecycle]]
- [[data_labeling|Data Labeling Strategies]]
- [[github_workflows|GitHub Workflows]]
