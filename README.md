# Notes

A personal knowledge management (PKM) system for professional work in data science, machine learning, and AI. Built in [Obsidian](https://obsidian.md).

---

## Purpose

This vault is a structured, long-lived knowledge base — not a note dump. It is organized by **epistemic layer and abstraction level**: each layer answers a distinct question, from *why does this work mathematically?* down to *why are we solving this problem?*

The goal is to accumulate knowledge that compounds over time: timeless theory in foundations, reusable models and algorithms in modeling, proven engineering patterns in the engineering layers, and grounded domain knowledge in applications.

---

## Structure

```
notes/
├── 00_meta/              # Vault governance: conventions, templates, dashboard, glossary
├── 01_foundations/       # Timeless mathematical and theoretical foundations
├── 02_modeling/          # Model classes and modeling strategies (framework-agnostic)
├── 03_software_engineering/  # Programming languages, system design, tooling
├── 04_ml_engineering/    # ML production lifecycle: data, training, deployment, monitoring
├── 05_ai_engineering/    # Foundation model systems: RAG, fine-tuning, inference, LLMOps
├── 06_applications/      # Domain knowledge: insurance, business context, constraints
├── 07_projects/          # Time-bound execution instances
│   ├── _active/
│   ├── _completed/
│   └── _experimental/
├── 08_reading/           # Structured intake: papers, books, articles, reports
├── 09_logs/              # Operational memory: daily notes, reviews, brain dumps
└── 99_archive/           # Retired content
```

---

## Layer Guide

| Layer | Guiding Question | Examples |
|---|---|---|
| `01_foundations` | Why does this work mathematically? | Calculus, linear algebra, backpropagation theory |
| `02_modeling` | How do we model this problem? | GLMs, neural networks, transformers, interpretability |
| `03_software_engineering` | How do we build robust software? | Python, Go, TypeScript, APIs, databases, testing, DevOps, AI-assisted SE |
| `04_ml_engineering` | How do we productionize ML systems? | Feature stores, MLflow, drift detection, serving |
| `05_ai_engineering` | How do we operate LLM systems? | RAG, LoRA fine-tuning, quantization, LangSmith |
| `06_applications` | How are models and systems implemented in practice? | Model implementations, system patterns, end-to-end ML/AI pipelines |
| `07_projects` | What are we executing right now? | Active work, experiments |
| `08_reading` | What have we learned from external sources? | Paper notes, book summaries |
| `09_logs` | What happened today / this week? | Daily notes, reflections |

---

## Note Types

Notes follow one of six templates (see `00_meta/templates/`), matched to their layer and purpose:

**Concept note** (`type: concept`) — `01_foundations`, `02_modeling`
```
Definition → Intuition → Formal Description → Applications → Trade-offs → Links
```

**Proof note** (`type: proof`) — `01_foundations`
```
Statement → Assumptions → Proof Sketch → Full Proof → Notes / Intuition → Links
```

**Engineering note** (`type: engineering`) — `03_software_engineering`, `04_ml_engineering`, `05_ai_engineering` (tooling)
```
Purpose → Architecture → Implementation Notes → Trade-offs → References → Links
```

**AI system note** (`type: ai_system`) — `05_ai_engineering` (system design)
```
Goal → Architecture → Components → Evaluation → Failure Modes → Cost / Latency → Links
```

**Application note** (`type: application`) — `06_applications`
```
Problem Definition → Domain Context → Data Requirements → Modeling Options → Deployment Constraints → Risks → Links
```

**Project overview** (`type: project`) — `07_projects`
```
Goal → Scope (In / Out) → Deliverables → Data → Modeling → Engineering → Timeline → Links
```

All notes carry frontmatter: `layer`, `type`, `status` (`seed` / `growing` / `evergreen`), `tags`, `created`.

---

## Conventions

Full conventions are in [`00_meta/conventions.md`](00_meta/conventions.md). Key rules:

- **File names**: lowercase with underscores (`chain_rule.md`, not `ChainRule.md`)
- **Top-level structure is stable** — changes require updating `conventions.md`
- **No duplicate concepts** — if overlap exists, merge
- **Cross-linking**: modeling notes link to foundations; application notes link to modeling and engineering
- **Knowledge flows**: Reading → Logs → Projects → Core layers (stable)

---

## Getting Started

- Start at [`00_meta/dashboard.md`](00_meta/dashboard.md) for active work
- Browse any layer's `index.md` for entry points into that layer
- See [`00_meta/glossary.md`](00_meta/glossary.md) for term definitions
- Consult [`00_meta/conventions.md`](00_meta/conventions.md) before adding new notes
