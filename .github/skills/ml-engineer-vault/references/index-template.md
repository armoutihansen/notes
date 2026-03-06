# Index File Templates and Examples

Index files are the navigational backbone of the vault. Every numbered folder MUST have one.

---

## Canonical Template

```markdown
---
layer: <04_ml_engineering|03_software_engineering|etc>
type: index
status: evergreen
tags: []
created: YYYY-MM-DD
---

# <Sublayer Display Name>

<1-2 sentence description of what this sublayer covers and its role in the lifecycle.>

## Notes

- [[filename|Display Name]] — concise 1-line description of what the note covers
- [[filename2|Display Name 2]] — concise 1-line description

## Links

- [[adjacent_sublayer/index|Previous Sublayer Name]]
- [[adjacent_sublayer/index|Next Sublayer Name]]
- [[layer_index|Layer Index]]
```

---

## Rules for Index Files

1. **Include every `.md` file** in the directory (except `index.md` itself)
2. **Use meaningful display names** — not the raw filename
3. **Write real 1-line descriptions** — not just restating the filename
4. **Order by lifecycle/reading order** — not alphabetically
5. **Link to neighbors** — the sublayer before and after in the lifecycle
6. **Link up** — to the parent layer's index

---

## Example: `04_ml_engineering/04_model_development/index.md`

```markdown
---
layer: 04_ml_engineering
type: index
status: evergreen
tags: []
created: 2026-03-05
---

# Model Development

The model development stage covers everything from baseline experiments to
full-scale distributed training and rigorous offline evaluation before deployment.

## Notes

- [[experiment_tracking|Experiment Tracking]] — MLflow and W&B patterns for reproducible ML experiments
- [[distributed_training|Distributed Training]] — DeepSpeed, FSDP, and Accelerate for multi-GPU/multi-node training
- [[offline_evaluation|Offline Evaluation]] — evaluation protocols, test set discipline, and offline metrics

## Links

- [[03_feature_engineering/index|Feature Engineering]] ← previous stage
- [[05_deployment_and_serving/index|Deployment & Serving]] → next stage
- [[04_ml_engineering/index|ML Engineering]]
```

---

## Example: `04_ml_engineering/index.md` (layer-level index)

```markdown
---
layer: 04_ml_engineering
type: index
status: evergreen
tags: []
created: 2026-03-05
---

# ML Engineering

Production machine learning systems from data ingestion to model retirement.
Organized by the ML production lifecycle.

## Sublayers

### [[00_principles_and_lifecycle/index|00 — Principles & Lifecycle]]
ML system design principles, the iterative lifecycle, and reliability contracts.

### [[01_data_engineering/index|01 — Data Engineering]]
Ingestion pipelines, ETL/ELT patterns, batch vs. stream processing, and feature stores.

### [[02_training_data/index|02 — Training Data]]
Labeling workflows, class imbalance handling, augmentation, and dataset versioning.

...

## Links

- [[03_software_engineering/index|Software Engineering]]
- [[05_ai_engineering/index|AI Engineering]]
```

---

## Common Mistakes to Avoid

| Mistake | Fix |
|---------|-----|
| Alphabetical order | Use lifecycle/reading order |
| Missing entries | List ALL notes in directory |
| Empty descriptions | Write a real 1-line description |
| No outbound links | Always link to neighbors and parent |
| Stale entries (pointing to deleted files) | Remove or update dead links |
| `status: seed` on an index | Indexes should be `status: evergreen` |
