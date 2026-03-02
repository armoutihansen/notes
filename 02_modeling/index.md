# Modeling

## Purpose of This Layer

This layer captures the conceptual discipline of model design and selection.

It focuses on:

- Framing predictive and inferential problems
- Designing features and representations
- Choosing appropriate model families
- Training and tuning models
- Designing evaluation protocols
- Reasoning about interpretability and fairness
- Applying modeling techniques to specific domains

Guiding question:

> How should we model this problem, and how do we justify that choice?

This layer does NOT cover:

- Mathematical proofs and derivations (→ 01_foundations)
- Production pipelines, deployment, and monitoring (→ 04_ml_engineering)
- Foundation-model system design (→ 05_ai_engineering)

---
## Structure

### 00_modeling_principles/

Core conceptual foundations of modeling.

Includes:
- Modeling workflow
- Problem types and objectives
- Bias–variance tradeoff
- Overfitting and underfitting
- Error analysis

Focus:
Building intuition for how models behave and fail.

---
### 01_problem_framing/

Formalizing real-world problems as ML tasks.

Includes:
- Prediction vs inference
- Target definition
- Data leakage
- Train/validation/test splitting
- Time-series splitting

Focus:
Translating business or research questions into well-posed modeling tasks.

---
### 02_data_and_features/

Representation design.

Includes:
- Feature engineering overview
- Handling missing values
- Categorical encoding
- Scaling and normalization
- Text vectorization
- Embeddings
- Class imbalance handling
- Data augmentation strategies

Focus:
Designing inputs that make learning possible and robust.

---
### 03_model_families/

Algorithmic toolkits.

Includes:

### Linear and GLM
- Linear regression
- Logistic regression
- Poisson, Gamma, Tweedie GLMs

### Trees and Boosting
- Random forest
- Gradient boosting
- XGBoost, LightGBM, CatBoost

### Probabilistic Models
- Naive Bayes
- Bayesian regression
- Hidden Markov models

### Deep Learning
- MLP
- CNN
- RNN / LSTM / GRU
- Transformer architectures (modeling perspective)

### Unsupervised Learning
- Clustering (K-means, GMM)
- Dimensionality reduction (PCA, UMAP)
- Anomaly detection

### Time Series Models
- ARIMA / SARIMA
- State-space / Kalman models
- ML-based forecasting

Focus:
Understanding strengths, weaknesses, assumptions, and trade-offs of model classes.

---
### 04_training_and_tuning/

Optimization and generalization control.

Includes:
- Loss functions
- Regularization
- Optimization algorithms
- Hyperparameter tuning
- Early stopping

Focus:
Controlling model capacity and convergence behavior.

---
### 05_evaluation/

Designing reliable offline evaluation.

Includes:
- Offline evaluation design
- Classification metrics
- Regression metrics
- Ranking metrics
- Calibration
- Uncertainty quantification
- Cross-validation
- Backtesting for time series

Focus:
Measuring performance before deployment.

---
### 06_interpretability_and_fairness/

Model transparency and risk.

Includes:
- Interpretability overview
- SHAP and feature attribution
- Partial dependence and ICE
- Fairness metrics
- Model risk considerations

Focus:
Understanding model behavior and societal implications.

---
### 07_domain_modeling/

Applying modeling principles to specific domains.

### Insurance Models
- Claims frequency modeling
- Claims severity modeling
- Pricing with GLMs and boosting
- Reserving overview
- Fraud modeling

### Experimentation
- A/B testing basics
- Uplift modeling
- Causal inference overview

Focus:
Contextualizing general modeling techniques within applied settings.

---

## Relationship to Other Layers

01_foundations:
Provides mathematical justification for modeling techniques.

02_modeling:
Defines which models, features, and evaluation strategies to use.

04_ml_engineering:
Implements, deploys, and operates chosen models in production.

05_ai_engineering:
Engineers systems built around pretrained foundation models.

This layer treats models as conceptual decision objects — not production artifacts.

---
## Explicit Structure

```
02_modeling/
├── index.md
├── 00_modeling_principles/
│   ├── modeling_workflow.md
│   ├── problem_types_and_objectives.md
│   ├── bias_variance.md
│   ├── overfitting_underfitting.md
│   └── error_analysis.md
├── 01_problem_framing/
│   ├── prediction_vs_inference.md
│   ├── target_definition.md
│   ├── leakage_definition_and_examples.md
│   ├── train_val_test_splitting.md
│   └── time_series_splitting.md
├── 02_data_and_features/
│   ├── feature_engineering_overview.md
│   ├── missing_values.md
│   ├── encoding_categorical.md
│   ├── scaling_normalization.md
│   ├── text_vectorization.md
│   ├── embeddings.md
│   ├── imbalance_and_reweighting.md
│   └── augmentation_strategies.md
├── 03_model_families/
│   ├── linear_and_glm/
│   │   ├── linear_regression.md
│   │   ├── logistic_regression.md
│   │   └── poisson_gamma_tweedie_glm.md
│   ├── trees_and_boosting/
│   │   ├── random_forest.md
│   │   ├── gradient_boosting.md
│   │   └── xgboost_lightgbm_catboost.md
│   ├── probabilistic_models/
│   │   ├── naive_bayes.md
│   │   ├── bayesian_regression.md
│   │   └── hidden_markov_models.md
│   ├── deep_learning/
│   │   ├── mlp.md
│   │   ├── cnn.md
│   │   ├── rnn_lstm_gru.md
│   │   └── transformers_overview.md
│   ├── unsupervised_learning/
│   │   ├── clustering_kmeans_gmm.md
│   │   ├── dimensionality_reduction_pca_umap.md
│   │   └── anomaly_detection.md
│   └── time_series_models/
│       ├── arima_sarima.md
│       ├── state_space_kalman.md
│       └── forecasting_with_ml.md
├── 04_training_and_tuning/
│   ├── loss_functions.md
│   ├── regularization.md
│   ├── optimization_algorithms.md
│   ├── hyperparameter_tuning.md
│   └── early_stopping.md
├── 05_evaluation/
│   ├── offline_evaluation_design.md
│   ├── classification_metrics.md
│   ├── regression_metrics.md
│   ├── ranking_metrics.md
│   ├── calibration.md
│   ├── uncertainty_quantification.md
│   ├── cross_validation.md
│   └── backtesting_time_series.md
├── 06_interpretability_and_fairness/
│   ├── interpretability_overview.md
│   ├── shap_and_feature_attribution.md
│   ├── partial_dependence_ice.md
│   ├── fairness_metrics.md
│   └── model_risk_considerations.md
└── 07_domain_modeling/
    ├── insurance_models/
    │   ├── claims_frequency_modeling.md
    │   ├── claims_severity_modeling.md
    │   ├── pricing_glms_and_boosting.md
    │   ├── reserving_overview.md
    │   └── fraud_modeling.md
    └── experimentation/
        ├── ab_testing_basics.md
        ├── uplift_modeling.md
        └── causal_inference_overview.md
```