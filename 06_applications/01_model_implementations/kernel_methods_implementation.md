---
layer: 06_applications
type: application
domain: supervised-learning
stakeholders: []
regulatory: []
status: growing
tags: [algorithm, tabular, classification, regression]
created: 2026-03-06
---

# Kernel Methods — Implementation

## Purpose

Practical implementation of SVMs and SVRs with scikit-learn. Synthesised from [[02_modeling/03_model_families/05_kernel_methods/kernel_methods|Kernel Methods]].

### Examples

- SVC for binary and multi-class classification
- SVR for regression
- GridSearchCV for C/gamma tuning
- Kernel selection guide

## Architecture

```
Raw features → StandardScaler (critical for SVMs)
             → SVC / SVR with kernel
             → Decision boundary / predicted value
```

SVMs are sensitive to feature scale; always standardise. Computational cost is $O(n^2)$ to $O(n^3)$ in the number of training samples — use on datasets with $n < 100k$.

## Implementation

### Setup

```bash
pip install scikit-learn
```

### SVC (Classification)

```python
from sklearn.svm import SVC
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import GridSearchCV, cross_val_score
from sklearn.datasets import load_breast_cancer

X, y = load_breast_cancer(return_X_y=True)

svc = Pipeline([
    ("scaler", StandardScaler()),
    ("model", SVC(
        kernel="rbf",    # "linear", "poly", "rbf", "sigmoid"
        C=1.0,           # regularisation
        gamma="scale",   # 1 / (n_features * X.var()); or float
        probability=True # enable predict_proba
    ))
])
print(f"SVC AUC: {cross_val_score(svc, X, y, cv=5, scoring='roc_auc').mean():.3f}")
```

### Hyperparameter Tuning (C, gamma)

```python
param_grid = {
    "model__C":     [0.01, 0.1, 1, 10, 100],
    "model__gamma": ["scale", "auto", 0.001, 0.01, 0.1],
}
grid = GridSearchCV(svc, param_grid, cv=5, scoring="roc_auc", n_jobs=-1)
grid.fit(X, y)
print("Best params:", grid.best_params_)
print(f"Best AUC:   {grid.best_score_:.3f}")
```

### Kernel Selection Guide

| Kernel | When to use |
|--------|-------------|
| Linear | High-dimensional sparse data (text), linearly separable |
| RBF (Gaussian) | Default; low to moderate dimensions |
| Polynomial `degree=3` | Image classification, polynomial interactions |
| Sigmoid | Rarely; equivalent to shallow neural net |

### Multi-class SVC

```python
from sklearn.svm import SVC
# sklearn handles multi-class with OvR or OvO automatically
svc_mc = SVC(decision_function_shape="ovr")   # one-vs-rest
```

### SVR (Regression)

```python
from sklearn.svm import SVR
from sklearn.datasets import fetch_california_housing

X, y = fetch_california_housing(return_X_y=True)

svr = Pipeline([
    ("scaler", StandardScaler()),
    ("model", SVR(kernel="rbf", C=100.0, gamma=0.1, epsilon=0.1))
])
svr.fit(X_train, y_train)
print(f"SVR R²: {svr.score(X_test, y_test):.3f}")
```

### Decision Boundary Visualisation (2D)

```python
import numpy as np
import matplotlib.pyplot as plt

def plot_svm_boundary(clf, X, y, ax):
    h = 0.02
    x_min, x_max = X[:, 0].min() - 1, X[:, 0].max() + 1
    y_min, y_max = X[:, 1].min() - 1, X[:, 1].max() + 1
    xx, yy = np.meshgrid(np.arange(x_min, x_max, h),
                          np.arange(y_min, y_max, h))
    Z = clf.predict(np.c_[xx.ravel(), yy.ravel()])
    Z = Z.reshape(xx.shape)
    ax.contourf(xx, yy, Z, alpha=0.3)
    ax.scatter(X[:, 0], X[:, 1], c=y, edgecolors="k", s=20)
```

## Trade-offs

- SVMs excel in high-dimensional, low-sample settings; slow on $n > 100k$.
- Kernel choice and C/gamma together determine the decision boundary — use nested CV to tune.
- For large tabular data, gradient boosting (XGBoost/LightGBM) typically outperforms SVM with less tuning.

## Links

- [[02_modeling/03_model_families/05_kernel_methods/kernel_methods|Kernel Methods (theory)]]
- [[01_foundations/04_optimization/lagrangian_and_constrained_optimization|KKT / SVM Dual Derivation]]
- [[02_modeling/05_evaluation_and_validation/evaluation_and_validation|Evaluation and Validation]]
- [[06_applications/01_model_implementations/linear_models_implementation|Linear Models Implementation]]
