---
layer: 04_software_engineering
type: index
status: evergreen
tags: []
created: 2026-03-02
---

# DevOps & Infrastructure

Container orchestration, CI/CD pipelines, ML framework tooling, Git internals, and GitHub collaboration patterns. This sublayer consolidates infrastructure and version control: both concern how code moves from a developer's machine to production.

← Prev [[04_software_engineering/06_testing_and_quality/index|← Testing & Quality]] | Next → [[04_software_engineering/08_security/index|Security →]]

## Infrastructure & CI/CD

- [[docker_patterns|Docker Patterns]] — Dockerfile best practices, layer caching, multi-stage builds, Docker Compose, ML-specific patterns
- [[kubernetes_basics|Kubernetes Basics]] — control plane, core objects (Pod/Deployment/Service/Ingress), HPA, Helm, ML workloads
- [[cicd_pipelines|CI/CD Pipelines]] — GitHub Actions pipeline patterns for testing, building, deploying, and ML retraining
- [[kubernetes_deployment|Kubernetes Deployment]] — Deployment/Service/Ingress manifests, Helm charts, HPA, and rolling update patterns

## Git

- [[04_software_engineering/07_devops_and_infrastructure/01_git/index|Git]] — object model, staging, branching, rebase, reset, remotes, and workflow patterns
  - [[git_internals|Git Internals]] — object model (blobs, trees, commits, refs)
  - [[git_branching|Git Branching]] — branch creation, tracking, management
  - [[git_merge|Git Merge]] — merge strategies and conflict resolution
  - [[git_rebase|Git Rebase]] — interactive rebase, squash, fixup
  - [[git_workflow|Git Workflow]] — trunk-based development and GitHub Flow

## GitHub

- [[github_actions|GitHub Actions]] — workflow YAML, job matrix, caching, Docker build/push, self-hosted runners, OIDC auth
- [[github_workflows|GitHub Workflows]] — GitHub Flow, trunk-based development, PR best practices, branch protection, merge strategies
- [[github_repository_management|GitHub Repository Management]] — repo settings, issue/PR templates, releases, GitHub Packages, Dependabot

## Subdirectories

- [[04_software_engineering/07_devops_and_infrastructure/01_git/index|01_git/]] — full Git reference (12 notes)
- [[04_software_engineering/07_devops_and_infrastructure/00_ml_frameworks/index|00_ml_frameworks/]] — PyTorch, TensorFlow, JAX, and HuggingFace Transformers usage patterns

## Links
- [[04_software_engineering/index|Software Engineering]]
- [[04_software_engineering/06_testing_and_quality/index|Testing & Quality]]
- [[04_software_engineering/08_security/index|Security]]
- [[04_software_engineering/09_ai_assisted_software_engineering/index|AI-Assisted Engineering]]
- [[05_ml_engineering/09_infrastructure_and_platform/index|ML Infrastructure & Platform]]
