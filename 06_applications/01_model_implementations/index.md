---
layer: 06_applications
type: index
status: evergreen
tags: []
created: 2026-03-06
---

# Model Implementations

Practical implementations of models and algorithms from the modeling layer using real libraries, frameworks, and workflows.

> How is a model from `02_modeling` actually implemented in code?

This section focuses on **implementation**, not theory.

It serves as the practical companion to `02_modeling` by showing how model families, learning algorithms, and evaluation workflows are realized with tools such as `sklearn`, `PyTorch`, `XGBoost`, or related libraries.

This section does NOT cover:
- mathematical derivations (→ `01_foundations`)
- model definitions and conceptual comparisons (→ `02_modeling`)
- full production system architecture (→ `02_system_patterns`)
- project-specific business implementations (→ `07_projects`)

## Notes

### Supervised Learning
| Note | Description |
|---|---|
| [[linear_models_implementation\|Linear Models & GLMs]] | OLS, Ridge/Lasso, logistic regression, statsmodels GLMs with code |
| [[tree_ensembles_implementation\|Tree Ensembles]] | Random Forest, XGBoost (early stopping), LightGBM, Optuna tuning |
| [[kernel_methods_implementation\|Kernel Methods]] | SVC/SVR, kernel selection, GridSearchCV, decision boundary visualisation |
| [[probabilistic_models_implementation\|Probabilistic Models]] | GaussianNB, MultinomialNB, GMM + BIC, BayesianRidge |

### Unsupervised Learning
| Note | Description |
|---|---|
| [[unsupervised_learning_implementation\|Unsupervised Learning]] | K-means (elbow/silhouette), DBSCAN, PCA, t-SNE, UMAP, Isolation Forest |

### Forecasting
| Note | Description |
|---|---|
| [[time_series_implementation\|Time Series Models]] | Auto-ARIMA, SARIMAX, Holt-Winters ETS, walk-forward CV, LGBM features |

### Deep Learning
| Note | Description |
|---|---|
| [[neural_network_implementation\|Neural Network Implementation]] | PyTorch MLP, CNN, LSTM — full training loop with early stopping, validation, loss logging, model serialization |
| [[transformer_finetuning_implementation\|Transformer Fine-tuning]] | LoRA / QLoRA with PEFT + SFTTrainer — adapter config, training args, saving and merging adapters |

### Graphical Models
| Note | Description |
|---|---|
| [[graphical_model_implementation\|Graphical Model Implementation]] | pgmpy BayesianNetwork construction, MLE/Bayesian parameter fitting, VariableElimination inference |

### Interpretability
| Note | Description |
|---|---|
| [[interpretability_implementation\|Interpretability Implementation]] | SHAP TreeExplainer/KernelExplainer, PDP, permutation feature importance with scikit-learn |

## Role in the Vault

This section connects **model theory** to **practical code**:

```
01_foundations
        ↓
02_modeling
        ↓
06_applications/01_model_implementations
        ↓
07_projects
```

Typical note pattern:
- implement one modeling concept
- describe the code-level workflow
- record practical decisions and trade-offs
- link back to the relevant modeling note

## Links
- [[02_modeling/index|Modeling]]
- [[03_software_engineering/index|Software Engineering]]
- Projects