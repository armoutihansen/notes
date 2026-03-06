---
layer: 08_reference_implementations
type: application
domain: supervised-learning
stakeholders: []
regulatory: []
status: growing
tags: [algorithm, tabular, classification, regression]
created: 2026-03-06
---

# Tree Ensembles — Implementation

## Goal

Implement Random Forest, XGBoost, and LightGBM end-to-end with tuning, evaluation, and SHAP explainability.

## Conceptual Counterpart

- [[03_modeling/01_supervised_learning/02_tree_based_models/gradient_boosting|Gradient Boosting]] — boosting theory, XGBoost/LightGBM algorithm details
- [[07_applications/04_classification_and_decisioning/credit_scoring|Credit Scoring]] — typical decision tree ensemble deployment context
- [[07_applications/01_prediction_and_forecasting/churn_prediction|Churn Prediction]] — classification application using tree ensembles

## Purpose

Practical implementation of tree ensembles using scikit-learn, XGBoost, and LightGBM. Synthesised from [[03_modeling/01_supervised_learning/02_tree_based_models/gradient_boosting|Tree Ensembles]].

### Examples

- Random Forest classification/regression
- XGBoost with early stopping and custom evaluation
- LightGBM for large datasets
- Hyperparameter tuning with Optuna

## Architecture

```
Raw features → ColumnTransformer (encode categoricals)
             → Tree ensemble estimator
             → Prediction (probability / score)
```

Tree ensembles handle mixed types, missing values (XGBoost/LGBM), and non-linear interactions natively.

## Implementation

### Setup

```bash
pip install scikit-learn xgboost lightgbm optuna
```

### Random Forest (sklearn)

```python
from sklearn.ensemble import RandomForestClassifier, RandomForestRegressor
from sklearn.model_selection import cross_val_score
from sklearn.datasets import load_breast_cancer

X, y = load_breast_cancer(return_X_y=True)

rf = RandomForestClassifier(
    n_estimators=200,
    max_depth=None,          # grow full trees
    min_samples_leaf=1,
    max_features="sqrt",     # controls variance
    n_jobs=-1,
    random_state=42
)
print(f"RF AUC: {cross_val_score(rf, X, y, cv=5, scoring='roc_auc').mean():.3f}")
```

### XGBoost with Early Stopping

```python
import xgboost as xgb
from sklearn.model_selection import train_test_split

X_tr, X_val, y_tr, y_val = train_test_split(X, y, test_size=0.2, random_state=42)

model = xgb.XGBClassifier(
    n_estimators=1000,
    learning_rate=0.05,
    max_depth=6,
    subsample=0.8,
    colsample_bytree=0.8,
    min_child_weight=1,
    gamma=0,
    use_label_encoder=False,
    eval_metric="logloss",
    early_stopping_rounds=50,
    random_state=42
)
model.fit(
    X_tr, y_tr,
    eval_set=[(X_val, y_val)],
    verbose=False
)
print(f"Best iteration: {model.best_iteration}")
print(f"Val logloss: {model.best_score:.4f}")
```

**Native API (for custom objectives/metrics):**

```python
dtrain = xgb.DMatrix(X_tr, label=y_tr)
dval   = xgb.DMatrix(X_val, label=y_val)

params = {
    "max_depth": 6, "eta": 0.05,
    "objective": "binary:logistic",
    "eval_metric": "auc"
}
bst = xgb.train(
    params, dtrain, num_boost_round=1000,
    evals=[(dval, "val")],
    early_stopping_rounds=50,
    verbose_eval=100
)
```

### LightGBM

```python
import lightgbm as lgb

lgb_model = lgb.LGBMClassifier(
    n_estimators=1000,
    learning_rate=0.05,
    num_leaves=31,        # main complexity control
    max_depth=-1,
    min_child_samples=20,
    subsample=0.8,
    colsample_bytree=0.8,
    random_state=42
)
lgb_model.fit(
    X_tr, y_tr,
    eval_set=[(X_val, y_val)],
    callbacks=[lgb.early_stopping(50), lgb.log_evaluation(0)]
)
```

### Hyperparameter Tuning with Optuna

```python
import optuna
from sklearn.metrics import roc_auc_score

def objective(trial):
    params = {
        "n_estimators": 500,
        "learning_rate": trial.suggest_float("learning_rate", 1e-3, 0.3, log=True),
        "max_depth": trial.suggest_int("max_depth", 3, 10),
        "num_leaves": trial.suggest_int("num_leaves", 15, 127),
        "min_child_samples": trial.suggest_int("min_child_samples", 5, 100),
        "subsample": trial.suggest_float("subsample", 0.5, 1.0),
        "colsample_bytree": trial.suggest_float("colsample_bytree", 0.5, 1.0),
    }
    m = lgb.LGBMClassifier(**params, random_state=42, verbose=-1)
    m.fit(X_tr, y_tr, eval_set=[(X_val, y_val)],
          callbacks=[lgb.early_stopping(30), lgb.log_evaluation(-1)])
    return roc_auc_score(y_val, m.predict_proba(X_val)[:, 1])

study = optuna.create_study(direction="maximize")
study.optimize(objective, n_trials=50)
print("Best params:", study.best_params)
```

### Feature Importance

```python
import pandas as pd, matplotlib.pyplot as plt

fi = pd.Series(lgb_model.feature_importances_, index=feature_names)
fi.sort_values().tail(20).plot(kind="barh", figsize=(8, 6))
plt.title("LightGBM Feature Importance (split)")
plt.tight_layout()
```

## Trade-offs

- XGBoost second-order gradients are more accurate; LightGBM trains faster on large data (leaf-wise growth).
- Random forests are parallelisable and more robust to hyperparameter choice; ensembles of shallow trees (GBM) generally outperform.
- Tree ensembles do not extrapolate — use neural models when the test distribution may differ significantly from training.

## Links

- [[03_modeling/01_supervised_learning/02_tree_based_models/tree_ensembles|Tree Ensembles (theory)]]
- [[02_data_science/04_feature_engineering/feature_engineering|Feature Engineering]]
- [[03_modeling/07_evaluation_and_model_selection/evaluation_and_validation|Evaluation and Validation]]
- [[03_modeling/07_evaluation_and_model_selection/shap_and_feature_attribution|SHAP Feature Attribution]]
