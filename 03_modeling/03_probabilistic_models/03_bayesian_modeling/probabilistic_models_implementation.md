---
layer: 03_modeling
type: application
status: growing
tags: [algorithm, tabular, clustering, classification]
created: 2026-03-06
---

# Probabilistic Models — Implementation

## Goal

Implement Naive Bayes classifiers, Gaussian Mixture Models, and Bayesian regression with scikit-learn and PyMC.

## Conceptual Counterpart

- [[03_modeling/03_probabilistic_models/index|Probabilistic Models]] — generative modelling, Bayesian inference, EM algorithm for GMMs
- [[01_foundations/03_probability_and_statistics/index|Probability and Statistics]] — Bayesian updating, conjugate priors, marginal likelihood
- [[07_applications/03_detection_and_monitoring/fraud_detection|Fraud Detection]] — Naive Bayes as baseline classifier in fraud context

## Purpose

Practical implementation of Naive Bayes classifiers, Gaussian Mixture Models, and Bayesian linear regression with scikit-learn and PyMC.

### Examples

- GaussianNB, MultinomialNB, BernoulliNB
- GMM with model selection (BIC)
- Bayesian ridge regression

## Architecture

```
Raw features → (count vectoriser for NB, StandardScaler for GMM)
             → Probabilistic estimator
             → Posterior probabilities / soft cluster assignments
```

## Implementation

### Setup

```bash
pip install scikit-learn
```

### Naive Bayes Variants

```python
from sklearn.naive_bayes import GaussianNB, MultinomialNB, BernoulliNB
from sklearn.datasets import load_iris, fetch_20newsgroups
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.pipeline import Pipeline
from sklearn.metrics import accuracy_score

# GaussianNB — continuous features
X, y = load_iris(return_X_y=True)
gnb = GaussianNB()
gnb.fit(X_train, y_train)
print(f"GaussianNB accuracy: {accuracy_score(y_test, gnb.predict(X_test)):.3f}")

# MultinomialNB — text / count data
categories = ["sci.space", "talk.politics.guns", "comp.graphics"]
data = fetch_20newsgroups(subset="train", categories=categories)
clf_text = Pipeline([
    ("tfidf", TfidfVectorizer(max_features=5000)),
    ("nb", MultinomialNB(alpha=1.0))   # Laplace smoothing
])
clf_text.fit(data.data, data.target)

# BernoulliNB — binary features
BernoulliNB(alpha=1.0, binarize=0.5)  # threshold for binary conversion
```

### Gaussian Mixture Model (GMM)

```python
import numpy as np
from sklearn.mixture import GaussianMixture
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Select number of components by BIC
bics = []
for k in range(1, 11):
    gm = GaussianMixture(n_components=k, random_state=42, n_init=3)
    gm.fit(X_scaled)
    bics.append(gm.bic(X_scaled))

best_k = np.argmin(bics) + 1
print(f"Best k (BIC): {best_k}")

gm_final = GaussianMixture(
    n_components=best_k,
    covariance_type="full",   # "tied", "diag", "spherical"
    n_init=5,
    random_state=42
)
gm_final.fit(X_scaled)

# Soft cluster assignments
probabilities = gm_final.predict_proba(X_scaled)    # (n, k)
labels        = gm_final.predict(X_scaled)

# Log-likelihood
print(f"Log-likelihood: {gm_final.score(X_scaled):.3f}")

# Anomaly detection via low-density regions
log_probs = gm_final.score_samples(X_scaled)
threshold = np.percentile(log_probs, 5)
anomalies = X_scaled[log_probs < threshold]
```

### Bayesian Ridge Regression

```python
from sklearn.linear_model import BayesianRidge
from sklearn.datasets import fetch_california_housing

X, y = fetch_california_housing(return_X_y=True)

br = BayesianRidge(
    n_iter=300,
    alpha_1=1e-6,   # Gamma prior on noise precision
    alpha_2=1e-6,
    lambda_1=1e-6,  # Gamma prior on weight precision
    lambda_2=1e-6,
    compute_score=True
)
br.fit(X_train, y_train)

# Predictions with uncertainty
y_pred, y_std = br.predict(X_test, return_std=True)
print(f"Pred: {y_pred[:3]}, Std: {y_std[:3]}")
```

## Trade-offs

- GaussianNB is fast, works with few samples, but the conditional independence assumption is rarely met — it can still work well in practice.
- GMM is more expressive than k-means (soft assignments, covariance structure) but requires choosing $k$ and covariance type; BIC automates this.
- Bayesian Ridge adds principled uncertainty; use when calibrated confidence intervals on predictions are needed.

## Links

- [[03_modeling/03_probabilistic_models/03_bayesian_modeling/probabilistic_models|Probabilistic Models (theory)]]
- [[01_foundations/03_probability_and_statistics/bayesian_inference|Bayesian Inference (MAP, conjugate priors)]]
- [[03_modeling/07_evaluation_and_model_selection/evaluation_and_validation|Calibration and evaluation]]
- [[03_modeling/02_unsupervised_learning/01_clustering/unsupervised_learning_implementation|Unsupervised Learning Implementation (GMM vs k-means)]]
