# Notes

A personal knowledge management (PKM) system for professional work in data science, machine learning, and AI. Built in [Obsidian](https://obsidian.md).

---

## Purpose

This vault is a structured, long-lived knowledge base — not a note dump. It is organized by **epistemic layer and abstraction level**: each layer answers a distinct question, from *why does this work mathematically?* through to *why are we solving this business problem?*

The goal is to accumulate knowledge that compounds over time: timeless theory in foundations, analytical practice in data science, reusable model knowledge in modeling, proven engineering patterns in the engineering layers, domain use cases in applications, and concrete implementations in reference implementations.

---

## Structure

```
notes/
├── 00_meta/                        # Vault governance: conventions, templates, dashboard, glossary
├── 01_foundations/                 # Timeless mathematical and theoretical foundations
├── 02_data_science/                # Analytical reasoning: from data to decisions
├── 03_modeling/                    # Model families, training, and model selection
├── 04_software_engineering/        # Programming languages, system design, tooling
├── 05_ml_engineering/              # ML production lifecycle: data, training, deployment, monitoring
├── 06_ai_engineering/              # Foundation model systems: RAG, fine-tuning, inference, LLMOps
├── 07_applications/                # Domain use cases and business application patterns
├── 08_implementations/   # Cross-layer executable patterns: system patterns and end-to-end examples (concept-specific impl lives with the concept)
├── 09_projects/                    # Time-bound execution instances
│   ├── _active/
│   ├── _completed/
│   └── _experimental/
├── 10_reading/                     # Structured intake: papers, books, articles, reports
├── 11_logs/                        # Operational memory: daily notes, reviews, brain dumps
└── 99_archive/                     # Retired content
```

---

## Layer Guide

| Layer | Guiding Question | Examples |
|---|---|---|
| `01_foundations` | Why does this work mathematically? | Calculus, linear algebra, backpropagation theory |
| `02_data_science` | How do we reason from data to decisions? | EDA, feature engineering, experiment design, interpretability |
| `03_modeling` | Which model should we use and why? | GLMs, neural networks, transformers, evaluation, regularization |
| `04_software_engineering` | How do we build robust software? | Python, Go, TypeScript, APIs, databases, testing, DevOps |
| `05_ml_engineering` | How do we productionize ML systems? | Feature stores, MLflow, drift detection, serving |
| `06_ai_engineering` | How do we operate LLM systems? | RAG, LoRA fine-tuning, quantization, LangSmith |
| `07_applications` | Why are we solving this problem? | Fraud detection, churn prediction, document intelligence |
| `08_implementations` | How is this cross-cutting executable pattern implemented? | Reusable system patterns, end-to-end reference architectures (concept-specific impl → home layer) |
| `09_projects` | What are we executing right now? | Active work, experiments |
| `10_reading` | What have we learned from external sources? | Paper notes, book summaries |
| `11_logs` | What happened today / this week? | Daily notes, reflections |

---

## Note Types

Notes follow templates matched to their layer (see `00_meta/templates/`):

| Template | Layer | Structure |
|----------|-------|-----------|
| `tpl_foundation.md` | 01_foundations | Statement → Intuition → Mathematical Formulation → Assumptions → Consequences |
| `tpl_proof.md` | 01_foundations | Statement → Assumptions → Proof → Notes |
| `tpl_data_science.md` | 02_data_science | Problem Context → Analytical Goal → Data Considerations → Method → Validation |
| `tpl_model.md` | 03_modeling | Core Idea → Math Formulation → Inductive Bias → Strengths/Weaknesses → Variants |
| `tpl_software_note.md` | 04_software_engineering | Purpose → Core Concepts → Design Notes → Trade-offs → Implementation Notes |
| `tpl_ml_system.md` | 05_ml_engineering | Purpose → Architecture → Data/Feature Flow → Operational Considerations → Trade-offs |
| `tpl_ai_system.md` | 06_ai_engineering | Use Case → Model Strategy → Evaluation → Guardrails → Architecture → Cost/Latency |
| `tpl_application.md` | 07_applications | Problem → Stakeholders → Domain Context → Inputs/Outputs → Modeling Options → Risks |
| `tpl_reference_implementation.md` | 08_implementations | Goal → Conceptual Counterpart → Dependencies → Code Pattern → Practical Notes |
| `tpl_end_to_end_example.md` | 08_implementations | Goal → Problem Setup → Data → Design → Implementation → Evaluation → Extensions |
| `tpl_project_overview.md` | 09_projects | Goal → Scope → Deliverables → Data → Modeling → Engineering → Timeline |
| `tpl_reading_note.md` | 10_reading | Source → Type → Why It Matters → Key Ideas → Relevance to Vault |

All notes carry frontmatter: `layer`, `type`, `status` (`seed` / `growing` / `stable`), `tags`, `created`.

---

## Conventions

Full conventions are in [`00_meta/conventions.md`](00_meta/conventions.md). Key rules:

- **File names**: lowercase with underscores (`chain_rule.md`, not `ChainRule.md`)
- **Top-level structure is stable** — changes require updating `conventions.md`
- **No duplicate concepts** — if overlap exists, merge
- **Cross-linking**: modeling notes link to foundations; application notes link to modeling and engineering; reference implementations link to their conceptual counterpart
- **Knowledge flows**: Reading → Logs → Projects → Core layers (stable)
- **Tags**: cross-cutting metadata only; never restate the folder path (see §12 of conventions)

---

## Getting Started

- Start at [`00_meta/dashboard.md`](00_meta/dashboard.md) for active work
- Browse any layer's `index.md` for entry points into that layer
- See [`00_meta/glossary.md`](00_meta/glossary.md) for term definitions
- Consult [`00_meta/conventions.md`](00_meta/conventions.md) before adding new notes
