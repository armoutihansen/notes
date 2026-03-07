---
layer: 03_modeling
type: application
status: growing
tags: [algorithm, interpretability, tabular]
created: 2026-05-10
---

# Interpretability Implementation (SHAP, PDP, Permutation Importance)

## Goal

Explain ML model predictions using SHAP values, Partial Dependence Plots, and Permutation Feature Importance.

## Conceptual Counterpart

- [[03_modeling/07_evaluation_and_model_selection/index|Evaluation and Model Selection]] — model evaluation and selection framework
- [[07_applications/04_classification_and_decisioning/credit_scoring|Credit Scoring]] — regulatory interpretability requirements (FCRA adverse action notices)
- [[07_applications/08_domain_verticals/01_insurance/underwriting_support|Underwriting Support]] — model explainability in regulated insurance context

## Purpose

Practical guide for explaining ML model predictions using SHAP, PDP, and Permutation Feature Importance.

### Examples

**Compliance and audit**: Explain individual credit risk decisions with SHAP waterfall plots.

**Feature selection**: Use permutation importance to prune low-signal features before retraining.

**Model debugging**: Identify spurious feature correlations via PDP interaction plots.

---

## Architecture

```
Trained model + test set
        ↓
   Choose explainer based on model type
        ├── Tree model → TreeExplainer (exact, fast)
        ├── Black-box  → KernelExplainer (slow, approximate)
        └── Neural net → DeepExplainer / GradientExplainer
        ↓
   Compute SHAP values
        ↓
   Visualise (waterfall, beeswarm, summary bar, force plot)
        ↓
   Partial dependence (marginal effects)
        ↓
   Permutation importance (metric-based ranking)
```

---

## SHAP — TreeExplainer (Tree Models)

```python
import shap
import xgboost as xgb
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split

X, y = load_breast_cancer(return_X_y=True, as_frame=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Train model
model = xgb.XGBClassifier(n_estimators=200, max_depth=5,
                           eval_metric="logloss", random_state=42)
model.fit(X_train, y_train)

# Compute SHAP values — exact, fast for tree ensembles
explainer   = shap.TreeExplainer(model)
shap_values = explainer(X_test)   # Explanation object with .values, .base_values, .data

# --- Local explanation: single prediction ---
# Waterfall plot shows which features pushed prediction above/below baseline
shap.plots.waterfall(shap_values[0])

# --- Global summary: beeswarm ---
# Each dot = one sample; x = SHAP value; colour = feature value
shap.plots.beeswarm(shap_values)

# --- Global summary: mean |SHAP| bar chart ---
shap.plots.bar(shap_values)

# --- Force plot (interactive HTML) ---
shap.initjs()
shap.force_plot(explainer.expected_value, shap_values.values[0], X_test.iloc[0])
```

---

## SHAP — KernelExplainer (Any Black-Box Model)

```python
from sklearn.ensemble import RandomForestClassifier
import numpy as np

model_rf = RandomForestClassifier(n_estimators=100, random_state=42).fit(X_train, y_train)

# Use background dataset (summary of training data reduces computation)
background = shap.sample(X_train, 100)  # representative 100-row background
explainer_ke = shap.KernelExplainer(model_rf.predict_proba, background)

# Explain a small subset (KernelSHAP is slow — O(n_features * n_samples * n_background))
X_explain = X_test.iloc[:50]
shap_vals_ke = explainer_ke.shap_values(X_explain, nsamples=100)
# shap_vals_ke is a list of arrays, one per class

# Plot for the positive class (index 1)
shap.summary_plot(shap_vals_ke[1], X_explain)
```

---

## Partial Dependence Plots (scikit-learn)

```python
from sklearn.inspection import PartialDependenceDisplay

# Single-feature PDP — marginal effect of 'mean radius' on prediction
fig, ax = plt.subplots(figsize=(8, 4))
PartialDependenceDisplay.from_estimator(
    model_rf,
    X_train,
    features=["mean radius", "mean texture"],
    kind="average",   # "individual" for ICE plots, "both" for ICE+PDP
    ax=ax,
)
plt.tight_layout()

# 2-D interaction PDP — shows interaction between two features
PartialDependenceDisplay.from_estimator(
    model_rf,
    X_train,
    features=[("mean radius", "mean texture")],
    kind="average",
)
```

---

## Permutation Feature Importance

```python
from sklearn.inspection import permutation_importance
from sklearn.metrics import roc_auc_score

result = permutation_importance(
    model_rf, X_test, y_test,
    scoring="roc_auc",
    n_repeats=10,          # repeat shuffling for stable estimates
    random_state=42,
    n_jobs=-1,
)

# Sort by mean importance
import pandas as pd
importance_df = pd.DataFrame({
    "feature": X_test.columns,
    "importance_mean": result.importances_mean,
    "importance_std": result.importances_std,
}).sort_values("importance_mean", ascending=False)

print(importance_df.head(10))

# Box plot of all repeats
import matplotlib.pyplot as plt
sorted_idx = result.importances_mean.argsort()
plt.boxplot(result.importances[sorted_idx].T,
            vert=False,
            labels=X_test.columns[sorted_idx])
plt.title("Permutation Importances (test set)")
plt.tight_layout()
```

---

## Choosing the Right Method

| Method | Speed | Accuracy | Model-agnostic | Use when |
|---|---|---|---|---|
| TreeExplainer | Fast | Exact | Tree models only | XGBoost, LightGBM, RF, sklearn trees |
| KernelExplainer | Slow | Approximate | Yes | Neural nets, linear models, any sklearn estimator |
| DeepExplainer | Medium | Approximate | PyTorch/TF only | Neural network explanations |
| PDP | Fast | Exact marginal | Yes | Visualising feature effects for non-technical stakeholders |
| Permutation importance | Medium | Metric-based | Yes | Feature selection, identifying useless features |

**Production caution:** SHAP values reflect correlations, not causality. Highly correlated features will split attribution between them — interpret collectively, not individually.

---

## Links

**Foundations**
- [[01_foundations/05_statistical_learning_theory/generalization_bounds|Generalization Bounds]] — why model explanations matter: what a model learns vs. what it generalises to

**Modeling**
- [[03_modeling/07_evaluation_and_model_selection/shap_and_feature_attribution|SHAP and Feature Attribution]] — Shapley value theory and explainer variants
- [[03_modeling/07_evaluation_and_model_selection/partial_dependence_ice|Partial Dependence and ALE]] — marginal effect visualisation theory

**Model Implementations**
- [[tree_ensembles_implementation|Tree Ensembles Implementation]] — XGBoost, LightGBM, RF implementation (models to explain)
- [[linear_models_implementation|Linear Models Implementation]] — logistic regression coefficients as a natural baseline for attribution

**Applications**
- [[tabular_classification_pipeline|Tabular Classification Pipeline]] — end-to-end system integrating SHAP explanations post-training
