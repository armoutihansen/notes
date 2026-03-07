# Dashboard

## Core Layers
- [[01_foundations/index|01 — Foundations]]
- [[02_data_science/index|02 — Data Science]]
- [[03_modeling/index|03 — Modeling]]

## Engineering Stack
- [[04_software_engineering/index|04 — Software Engineering]]
- [[05_ml_engineering/index|05 — ML Engineering]]
- [[06_ai_engineering/index|06 — AI Engineering]]

## Applications & Implementations
- [[07_applications/index|07 — Applications]]
- [[08_implementations/index|08 — Reference Implementations]]

## Active Projects

```dataview
LIST
FROM "09_projects/_active"
WHERE file.name = "overview"
```

## Recent Notes

```dataview
TABLE layer, status
FROM ""
WHERE file.mtime > date(today) - dur(7 days)
AND !contains(file.path, "00_meta")
SORT file.mtime DESC
LIMIT 20
```
