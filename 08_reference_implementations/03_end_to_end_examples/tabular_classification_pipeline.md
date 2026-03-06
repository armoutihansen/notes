---
layer: 08_reference_implementations
type: application
status: growing
tags: [workflow, tabular, classification, mlops]
created: 2026-05-10
---

# Tabular Classification Pipeline

## Purpose

A complete reference architecture for a tabular supervised ML system, from raw CSV to monitored production API. This example synthesizes EDA, feature engineering, XGBoost training with MLflow tracking, SHAP explanations, FastAPI serving, and Evidently drift monitoring into a single coherent system.

### Examples

**Customer churn prediction**: Score all customers daily; flag high-risk segments for retention campaigns; monitor feature distributions weekly for drift.

**Credit risk scoring**: Retrain gradient-boosted model monthly on latest transaction data; explain individual credit decisions via SHAP waterfall plots for regulatory audit.

---

## Architecture

```
Raw data (CSV / DWH table)
    │
    ├──[1] EDA → data quality report (nulls, distributions, correlations)
    │
    ├──[2] Feature pipeline (sklearn Pipeline: impute → encode → scale)
    │
    ├──[3] XGBoost training + MLflow experiment tracking
    │       ├── hyperparameter tuning (Optuna)
    │       └── evaluation gate (AUC vs. champion)
    │
    ├──[4] SHAP explanations (TreeExplainer → beeswarm + waterfall)
    │
    ├──[5] FastAPI inference endpoint
    │       ├── /predict — return probability + top SHAP contributors
    │       └── /health  — liveness probe
    │
    └──[6] Evidently drift monitoring
            ├── weekly data drift report
            └── automated retraining trigger
```

---

## Step 1: EDA

```python
import pandas as pd, numpy as np, matplotlib.pyplot as plt, seaborn as sns

df = pd.read_csv("data/customers.csv")

# Data quality summary
print(df.dtypes)
print(df.isnull().sum().sort_values(ascending=False).head(10))
print(df.describe())

# Target distribution
df["churn"].value_counts(normalize=True).plot(kind="bar", title="Churn distribution")
plt.tight_layout(); plt.savefig("reports/target_dist.png")

# Correlation heatmap (numeric features)
corr = df.select_dtypes("number").corr()
sns.heatmap(corr, cmap="coolwarm", vmax=1, vmin=-1, center=0, square=True)
plt.savefig("reports/correlation_heatmap.png")
```

---

## Step 2: Feature Pipeline

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OrdinalEncoder
from sklearn.impute import SimpleImputer

NUMERIC_FEATS   = ["tenure", "monthly_charges", "total_charges", "txn_count_30d"]
CATEGORICAL_FEATS = ["contract_type", "payment_method", "internet_service"]

numeric_pipeline = Pipeline([
    ("impute", SimpleImputer(strategy="median")),
    ("scale",  StandardScaler()),
])
categorical_pipeline = Pipeline([
    ("impute",  SimpleImputer(strategy="most_frequent")),
    ("encode",  OrdinalEncoder(handle_unknown="use_encoded_value", unknown_value=-1)),
])

preprocessor = ColumnTransformer([
    ("num",  numeric_pipeline,      NUMERIC_FEATS),
    ("cat",  categorical_pipeline,  CATEGORICAL_FEATS),
])
```

---

## Step 3: XGBoost Training + MLflow

```python
import xgboost as xgb, mlflow, mlflow.xgboost, optuna
from sklearn.model_selection import train_test_split
from sklearn.metrics import roc_auc_score

X, y = df[NUMERIC_FEATS + CATEGORICAL_FEATS], df["churn"]
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)
X_train_p, X_test_p = preprocessor.fit_transform(X_train), preprocessor.transform(X_test)

mlflow.set_experiment("churn-xgboost")

def objective(trial):
    params = {
        "n_estimators":  trial.suggest_int("n_estimators", 100, 500),
        "max_depth":     trial.suggest_int("max_depth", 3, 7),
        "learning_rate": trial.suggest_float("learning_rate", 0.01, 0.2, log=True),
        "subsample":     trial.suggest_float("subsample", 0.6, 1.0),
        "colsample_bytree": trial.suggest_float("colsample_bytree", 0.6, 1.0),
    }
    model = xgb.XGBClassifier(**params, use_label_encoder=False, eval_metric="logloss", random_state=42)
    model.fit(X_train_p, y_train, eval_set=[(X_test_p, y_test)], verbose=False)
    return roc_auc_score(y_test, model.predict_proba(X_test_p)[:, 1])

study = optuna.create_study(direction="maximize")
study.optimize(objective, n_trials=50, show_progress_bar=True)

# Final training run with best params
with mlflow.start_run(run_name="challenger_best"):
    best_params = study.best_params
    model = xgb.XGBClassifier(**best_params, use_label_encoder=False, eval_metric="logloss", random_state=42)
    model.fit(X_train_p, y_train)
    auc = roc_auc_score(y_test, model.predict_proba(X_test_p)[:, 1])
    mlflow.log_params(best_params)
    mlflow.log_metric("val_auc", auc)
    mlflow.xgboost.log_model(model, "model", registered_model_name="churn_xgboost")
    print(f"AUC: {auc:.4f}")
```

---

## Step 4: SHAP Explanations

```python
import shap, joblib

explainer   = shap.TreeExplainer(model)
shap_values = explainer(X_test_p)

# Global: beeswarm plot
shap.plots.beeswarm(shap_values, max_display=15)
plt.savefig("reports/shap_beeswarm.png", bbox_inches="tight")

# Local: waterfall for the highest-risk customer
top_risk_idx = model.predict_proba(X_test_p)[:, 1].argmax()
shap.plots.waterfall(shap_values[top_risk_idx])
plt.savefig("reports/shap_waterfall_top_risk.png", bbox_inches="tight")

# Persist artefacts for serving
joblib.dump(preprocessor, "artefacts/preprocessor.pkl")
model.save_model("artefacts/model.json")
joblib.dump(explainer, "artefacts/explainer.pkl")
```

---

## Step 5: FastAPI Serving

```python
# api.py
import joblib, xgboost as xgb, shap, numpy as np
from fastapi import FastAPI
from pydantic import BaseModel

app   = FastAPI(title="Churn Prediction API")
pp    = joblib.load("artefacts/preprocessor.pkl")
model = xgb.XGBClassifier(); model.load_model("artefacts/model.json")
exp   = joblib.load("artefacts/explainer.pkl")
FEAT_NAMES = ["tenure", "monthly_charges", "total_charges", "txn_count_30d",
              "contract_type", "payment_method", "internet_service"]

class Customer(BaseModel):
    tenure: float
    monthly_charges: float
    total_charges: float
    txn_count_30d: int
    contract_type: str
    payment_method: str
    internet_service: str

@app.post("/predict")
def predict(customer: Customer):
    import pandas as pd
    row    = pd.DataFrame([customer.dict()])
    X_p    = pp.transform(row)
    prob   = float(model.predict_proba(X_p)[0, 1])
    sv     = exp(X_p).values[0]
    top3   = sorted(zip(FEAT_NAMES, sv), key=lambda x: abs(x[1]), reverse=True)[:3]
    return {"churn_probability": prob, "top_contributors": [{"feature": f, "shap": s} for f, s in top3]}

@app.get("/health")
def health(): return {"status": "ok"}
```

```bash
uvicorn api:app --host 0.0.0.0 --port 8080 --workers 2
```

---

## Step 6: Drift Monitoring

```python
from evidently.report import Report
from evidently.metric_preset import DataDriftPreset

reference = pd.read_parquet("data/training_features.parquet")[FEAT_NAMES]
current   = pd.read_parquet("data/production_features_last_week.parquet")[FEAT_NAMES]

report = Report(metrics=[DataDriftPreset()])
report.run(reference_data=reference, current_data=current)
report.save_html("reports/weekly_drift.html")

n_drifted = report.as_dict()["metrics"][0]["result"]["number_of_drifted_columns"]
if n_drifted > 2:
    print(f"WARNING: {n_drifted} features drifted — trigger retraining")
```

---

## Key Trade-offs

| Decision | Choice | Alternative |
|---|---|---|
| Model | XGBoost (fast, explainable) | LightGBM (faster on large data), CatBoost (native categoricals) |
| Serving | FastAPI (flexible, fast dev) | BentoML (built-in batching, model versioning) |
| Explainability | SHAP TreeExplainer (exact) | Partial dependence (global only) |
| Monitoring | Evidently (open-source) | Fiddler, Arize (managed) |

---

## Links

**Data Science & Modeling**
- [[02_data_science/03_exploratory_data_analysis/exploratory_data_analysis|Exploratory Data Analysis]] — EDA patterns and diagnostic plots
- [[02_data_science/04_feature_engineering/feature_engineering|Feature Engineering]] — encoding, scaling, imputation strategies
- [[03_modeling/07_evaluation_and_model_selection/shap_and_feature_attribution|SHAP and Feature Attribution]] — Shapley value theory

**Model Implementations**
- [[tree_ensembles_implementation|Tree Ensembles Implementation]] — XGBoost, LightGBM, Optuna tuning code
- [[interpretability_implementation|Interpretability Implementation]] — SHAP TreeExplainer, PDP, permutation importance

**System Patterns**
- [[model_serving_with_fastapi|Model Serving with FastAPI]] — production API patterns
- [[training_pipeline_pattern|Training Pipeline Pattern]] — MLflow + DVC training pipeline
- [[model_monitoring_system|Model Monitoring System]] — Evidently + Prometheus + alerting
