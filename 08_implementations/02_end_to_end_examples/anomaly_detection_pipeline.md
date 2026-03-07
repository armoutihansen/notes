---
layer: 08_implementations
type: application
domain: anomaly-detection
stakeholders: [data-scientist, ml-engineer]
regulatory: []
status: growing
tags: [workflow, tabular, anomaly-detection]
created: 2026-03-06
---

# Anomaly Detection Pipeline

## Purpose

A complete anomaly detection system combining unsupervised learning (Isolation Forest, GMM), dimensionality reduction, and systematic evaluation. Applies to fraud detection, infrastructure monitoring, data quality, and industrial defect detection.

### Examples

- Isolation Forest on tabular sensor / transaction data
- PCA for high-dimensional data compression before detection
- GMM density-based anomaly scoring
- Evaluation in the absence of ground-truth labels

## Architecture

```
┌────────────────────────────┐
│  Raw Feature Matrix         │  Tabular, potentially high-dim
└──────────────┬─────────────┘
               │
      ┌────────▼────────┐
      │  EDA + Cleaning │  Missingness, outlier profiling
      └────────┬────────┘
               │
      ┌────────▼────────┐
      │  Preprocessing  │  StandardScaler, PCA (optional)
      └────────┬────────┘
               │
    ┌──────────┴──────────┐
    │                     │
┌───▼───────────┐  ┌──────▼──────────┐
│ Isolation     │  │  GMM Density    │
│ Forest        │  │  Scoring        │
└───┬───────────┘  └──────┬──────────┘
    │                     │
    └──────────┬──────────┘
               │
      ┌────────▼────────┐
      │ Score Ensemble  │  Rank / threshold
      └────────┬────────┘
               │
      ┌────────▼────────┐
      │ Evaluation      │  Precision@k, PR-AUC if labels exist
      │ + Alerts        │
      └─────────────────┘
```

## Step-by-Step Implementation

### 1. EDA and Preprocessing

```python
import numpy as np
import pandas as pd
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA

df = pd.read_csv("transactions.csv")

# Quick profiling
print(df.describe())
print(f"Missing: {df.isnull().mean().sort_values(ascending=False).head()}")

# Scale
scaler = StandardScaler()
X = scaler.fit_transform(df.select_dtypes(include="number"))

# Optional: PCA to 95% variance for high-dimensional data
pca = PCA(n_components=0.95, random_state=42)
X_pca = pca.fit_transform(X)
print(f"PCA reduced to {X_pca.shape[1]} components")
```

### 2. Isolation Forest

```python
from sklearn.ensemble import IsolationForest

iso = IsolationForest(
    n_estimators=200,
    contamination=0.05,   # set based on expected anomaly rate
    max_samples="auto",
    random_state=42,
    n_jobs=-1
)
iso.fit(X_pca)

scores_iso = iso.score_samples(X_pca)   # lower = more anomalous
labels_iso = iso.predict(X_pca)         # -1 = anomaly
print(f"Anomalies detected: {(labels_iso == -1).sum()}")
```

### 3. GMM Density Scoring

```python
from sklearn.mixture import GaussianMixture

# Select k by BIC
bics = [GaussianMixture(k, n_init=3, random_state=42).fit(X_pca).bic(X_pca)
        for k in range(2, 10)]
best_k = np.argmin(bics) + 2

gm = GaussianMixture(best_k, n_init=5, covariance_type="full", random_state=42)
gm.fit(X_pca)

log_probs = gm.score_samples(X_pca)   # log-likelihood; low = anomaly
threshold = np.percentile(log_probs, 5)
labels_gmm = (log_probs < threshold).astype(int)
```

### 4. Score Ensemble

```python
from scipy.stats import rankdata

# Normalise scores to [0, 1] (higher = more anomalous)
rank_iso = rankdata(-scores_iso) / len(scores_iso)
rank_gmm = rankdata(-log_probs)  / len(log_probs)

ensemble_score = 0.5 * rank_iso + 0.5 * rank_gmm

# Flag top-k as anomalies
k = 100
top_k_idx = np.argsort(ensemble_score)[-k:]
```

### 5. Evaluation

**If ground-truth labels are available (semi-supervised setting):**

```python
from sklearn.metrics import roc_auc_score, average_precision_score

y_true = df["is_fraud"].values  # 0/1

print(f"IF  ROC-AUC: {roc_auc_score(y_true, -scores_iso):.3f}")
print(f"GMM ROC-AUC: {roc_auc_score(y_true, -log_probs):.3f}")
print(f"Ens ROC-AUC: {roc_auc_score(y_true, ensemble_score):.3f}")

# Average Precision (PR-AUC) — better for imbalanced
print(f"Ens AP: {average_precision_score(y_true, ensemble_score):.3f}")
```

**Without labels (unsupervised evaluation):**

```python
# Visual inspection: project anomalies via UMAP
import umap

reducer = umap.UMAP(n_components=2, random_state=42)
X_2d = reducer.fit_transform(X_pca)

import matplotlib.pyplot as plt
plt.scatter(X_2d[:, 0], X_2d[:, 1], c=ensemble_score, cmap="Reds", s=2, alpha=0.5)
plt.colorbar(label="Anomaly score")
plt.title("UMAP — anomaly score overlay")
plt.show()
```

### 6. Operational Deployment

```python
import joblib

joblib.dump({
    "scaler": scaler,
    "pca": pca,
    "iso": iso,
    "gm": gm
}, "anomaly_detector.pkl")

def score_new_batch(X_new: np.ndarray, artefacts: dict) -> np.ndarray:
    X_s = artefacts["scaler"].transform(X_new)
    X_p = artefacts["pca"].transform(X_s)
    r_iso = -artefacts["iso"].score_samples(X_p)
    r_gmm = -artefacts["gm"].score_samples(X_p)
    return 0.5 * (rankdata(r_iso) / len(r_iso)) + \
           0.5 * (rankdata(r_gmm) / len(r_gmm))
```

## Trade-offs

- Isolation Forest: fast, scalable, tree-based; works well in high dimensions; not differentiable.
- GMM: provides probability estimates; sensitive to $k$ choice; struggles in very high dimensions.
- Ensemble: more robust to individual model failure; harder to explain to stakeholders.
- When ground-truth labels become available, switch to a supervised model (XGBoost) trained on confirmed positives.

## Links

- [[03_modeling/02_unsupervised_learning/01_clustering/unsupervised_learning|Unsupervised Learning (theory)]]
- [[03_modeling/03_probabilistic_models/03_bayesian_modeling/probabilistic_models|Probabilistic Models (GMM)]]
- [[02_data_science/03_exploratory_data_analysis/exploratory_data_analysis|EDA]]
- [[03_modeling/07_evaluation_and_model_selection/evaluation_and_validation|Evaluation (PR-AUC, ROC-AUC)]]
- [[03_modeling/02_unsupervised_learning/01_clustering/unsupervised_learning_implementation|Unsupervised Learning Implementation]]
- [[03_modeling/03_probabilistic_models/03_bayesian_modeling/probabilistic_models_implementation|Probabilistic Models Implementation]]
- [[08_implementations/02_end_to_end_examples/batch_ml_prediction_pipeline|Batch ML Prediction Pipeline]]
