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

# Linear Models and GLMs — Implementation

## Purpose

Practical implementation of linear and generalised linear models with scikit-learn and statsmodels. Synthesised from [[02_modeling/03_model_families/01_linear_and_glm/linear_and_glm|Linear Models & GLMs]].

### Examples

- OLS, Ridge, Lasso, ElasticNet regression
- Logistic regression (binary and multi-class)
- Tweedie/Gamma GLM for insurance-style count/severity targets

## Architecture

```
Raw features → ColumnTransformer (scale / encode)
             → Linear estimator (sklearn or statsmodels)
             → Threshold / inverse-link → predictions
```

## Implementation

### Setup

```bash
pip install scikit-learn statsmodels
```

### OLS and Regularised Regression

```python
import numpy as np
from sklearn.linear_model import Ridge, Lasso, ElasticNet
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.datasets import fetch_california_housing
from sklearn.model_selection import cross_val_score

X, y = fetch_california_housing(return_X_y=True)

# Ridge (L2) — automatically handles multicollinearity
ridge = Pipeline([("scaler", StandardScaler()), ("model", Ridge(alpha=1.0))])
cv_r2 = cross_val_score(ridge, X, y, cv=5, scoring="r2")
print(f"Ridge CV R²: {cv_r2.mean():.3f} ± {cv_r2.std():.3f}")

# Lasso (L1) — sparse solutions
lasso = Pipeline([("scaler", StandardScaler()), ("model", Lasso(alpha=0.01))])

# ElasticNet — combined
enet = Pipeline([
    ("scaler", StandardScaler()),
    ("model", ElasticNet(alpha=0.01, l1_ratio=0.5))
])
```

**Selecting alpha with cross-validation:**

```python
from sklearn.linear_model import RidgeCV, LassoCV

lasso_cv = Pipeline([
    ("scaler", StandardScaler()),
    ("model", LassoCV(cv=5, alphas=np.logspace(-4, 2, 50)))
])
lasso_cv.fit(X, y)
print("Best alpha:", lasso_cv["model"].alpha_)
print("Non-zero coefficients:", np.sum(lasso_cv["model"].coef_ != 0))
```

### Logistic Regression

```python
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_breast_cancer
from sklearn.metrics import classification_report, roc_auc_score

X, y = load_breast_cancer(return_X_y=True)

clf = Pipeline([
    ("scaler", StandardScaler()),
    ("model", LogisticRegression(
        C=1.0,          # inverse regularisation strength
        penalty="l2",   # or "l1", "elasticnet"
        solver="lbfgs",
        max_iter=200
    ))
])
clf.fit(X_train, y_train)
proba = clf.predict_proba(X_test)[:, 1]
print(f"AUC: {roc_auc_score(y_test, proba):.3f}")
print(classification_report(y_test, clf.predict(X_test)))
```

**Multi-class (multinomial):**

```python
LogisticRegression(multi_class="multinomial", solver="lbfgs")
```

### GLMs with statsmodels (Tweedie / Gamma)

```python
import statsmodels.api as sm

# Gamma GLM for continuous positive target (e.g., claim severity)
X_sm = sm.add_constant(X_train)
glm = sm.GLM(y_train, X_sm, family=sm.families.Gamma(sm.families.links.Log()))
res = glm.fit()
print(res.summary())
y_pred = res.predict(sm.add_constant(X_test))

# Tweedie (p=1.5 → compound Poisson-Gamma for pure premium)
glm_tw = sm.GLM(
    y_train, X_sm,
    family=sm.families.Tweedie(var_power=1.5, link=sm.families.links.Log())
)
res_tw = glm_tw.fit()
```

### Coefficient Inspection

```python
import pandas as pd

coef_df = pd.DataFrame({
    "feature": feature_names,
    "coefficient": clf["model"].coef_[0],
    "abs": np.abs(clf["model"].coef_[0])
}).sort_values("abs", ascending=False)
print(coef_df.head(10))
```

## Trade-offs

- Ridge never produces exactly zero coefficients; Lasso does — prefer Lasso when feature selection matters.
- Logistic regression is fast, interpretable, well-calibrated with Platt scaling; use as a strong baseline.
- Statsmodels GLM provides inference (p-values, CIs) that sklearn does not — important for regulated use.

## Links

- [[02_modeling/03_model_families/01_linear_and_glm/linear_and_glm|Linear Models & GLMs (theory)]]
- [[02_modeling/02_data_science/data_preprocessing|Data Preprocessing (pipelines, scaling)]]
- [[02_modeling/05_evaluation_and_validation/evaluation_and_validation|Evaluation and Validation]]
- [[01_foundations/04_optimization/gradient_descent_optimization|Gradient Descent (solver internals)]]
