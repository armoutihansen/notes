---
layer: 04_software_engineering
type: index
status: evergreen
tags: []
created: 2026-03-05
---

# 04 — Software Engineering

General software engineering knowledge supporting all technical work in the vault. Covers the full development lifecycle from design principles through to AI-assisted tooling, with depth in Python, Go, TypeScript, distributed systems, APIs, databases, testing, and DevOps.

> Guiding question: "How do we build robust software systems?"

This layer does NOT cover: statistical derivations (→ `01_foundations`), model algorithm selection (→ `03_modeling`), ML production systems (→ `05_ml_engineering`), or LLM system design (→ `06_ai_engineering`).

## Sublayers

### [[04_software_engineering/01_principles_and_patterns/index|01 — Principles & Patterns]]
SOLID, design patterns, and clean code practices that govern how good software is structured.

### [[04_software_engineering/02_programming_languages/index|02 — Programming Languages]]
Language reference notes for Python, Go, JavaScript, and TypeScript.
- [[04_software_engineering/02_programming_languages/00_python/index|Python]] — core language, data model, async, tooling
- [[04_software_engineering/02_programming_languages/01_go/index|Go]] — types, concurrency, interfaces, packages
- [[04_software_engineering/02_programming_languages/02_javascript/index|JavaScript]] — core language, async, modules, runtimes
- [[04_software_engineering/02_programming_languages/03_typescript/index|TypeScript]] — type system, generics, utility types

### [[04_software_engineering/03_system_design/index|03 — System Design]]
Fundamentals of designing large-scale distributed systems: CAP theorem, consistency, scalability, and availability trade-offs.

### [[04_software_engineering/04_apis_and_services/index|04 — APIs & Services]]
REST API design, FastAPI patterns, gRPC and Protobuf.

### [[04_software_engineering/05_databases_and_storage/index|05 — Databases & Storage]]
SQL, NoSQL, and caching patterns for persistent data management.

### [[04_software_engineering/06_testing_and_quality/index|06 — Testing & Quality]]
Testing strategies, TDD, property-based testing, and quality gates.

### [[04_software_engineering/07_devops_and_infrastructure/index|07 — DevOps & Infrastructure]]
Docker, Kubernetes, CI/CD pipelines, ML framework tooling, Git internals, and GitHub workflows. Version control content lives here.

### [[04_software_engineering/08_security/index|08 — Security]]
Defensive engineering patterns: filesystem sandboxing, authentication, and trust boundary enforcement.

### [[04_software_engineering/09_ai_assisted_software_engineering/index|09 — AI-Assisted Software Engineering]]
LLM code generation, MCP protocol, RAG for code, and agentic coding workflows.

## Relationship to Other Layers

- ← [[03_modeling/index|Modeling]] — algorithm and model families that SE implements
- → [[05_ml_engineering/index|ML Engineering]] — production ML systems built on SE patterns
- → [[06_ai_engineering/index|AI Engineering]] — LLM systems and AI app architecture
- → [[08_implementations/index|Reference Implementations]] — concrete end-to-end examples synthesizing SE patterns

---

**Navigation:** ← [[03_modeling/index|Modeling]] | [[04_software_engineering/01_principles_and_patterns/index|Start with Principles & Patterns →]]
