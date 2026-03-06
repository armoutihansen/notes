---
layer: 02_modeling
type: concept
status: growing
tags: [feature-engineering, tabular]
created: 2026-03-06
---

# Feature Engineering

## Definition

The process of constructing new input features from raw data to improve model performance by making relevant patterns more explicit and accessible to the learning algorithm.

## Intuition

Features are the language in which your model thinks. A poor feature set forces the model to "discover" structure that could have been provided directly; good features encode domain knowledge and reduce the hypothesis space. Tree-based models can learn non-linear interactions, but encoding them as features speeds up learning and reduces the need for large trees.

## Formal Description

### Numerical Feature Transformations

**Polynomial and interaction features:**

```python
from sklearn.preprocessing import PolynomialFeatures

# Adds x1², x2², x1*x2 (degree=2, no bias term)
poly = PolynomialFeatures(degree=2, include_bias=False)
X_poly = poly.fit_transform(X)
```

Use with regularised linear models (ridge, lasso) to capture non-linear effects.

**Binning / discretisation:**

```python
from sklearn.preprocessing import KBinsDiscretizer

# Equal-frequency bins (quantile-based)
kbd = KBinsDiscretizer(n_bins=5, encode='ordinal', strategy='quantile')
X_binned = kbd.fit_transform(X[['age']])
```

Useful when the relationship is highly non-linear, or to create monotonic features for tree models.

**Ratio and difference features:**

```python
# E.g., for insurance: premium per unit of exposure
df['premium_per_vehicle'] = df['total_premium'] / df['n_vehicles']
df['age_diff'] = df['policy_end_year'] - df['policy_start_year']
```

**Log, sqrt, and power transforms:** applied to right-skewed count/amount features (see [[data_preprocessing]]).

### Temporal Features

```python
df['day_of_week']  = df['date'].dt.dayofweek
df['month']        = df['date'].dt.month
df['is_weekend']   = df['day_of_week'].isin([5, 6]).astype(int)
df['days_since']   = (pd.Timestamp.today() - df['date']).dt.days
```

Cyclical encoding (preserves periodicity for sine/cosine):

```python
import numpy as np
df['month_sin'] = np.sin(2 * np.pi * df['month'] / 12)
df['month_cos'] = np.cos(2 * np.pi * df['month'] / 12)
```

### Categorical Feature Engineering

**Frequency encoding:**

```python
freq = X_train[col].value_counts(normalize=True)
X_train[col + '_freq'] = X_train[col].map(freq)
```

**Target encoding** (with cross-validation to avoid leakage):

```python
from sklearn.model_selection import KFold
import numpy as np

X_train['target_enc'] = np.nan
kf = KFold(n_splits=5, shuffle=True, random_state=42)
for tr_idx, val_idx in kf.split(X_train):
    means = y_train.iloc[tr_idx].groupby(X_train.iloc[tr_idx][col]).mean()
    X_train.iloc[val_idx, X_train.columns.get_loc('target_enc')] = \
        X_train.iloc[val_idx][col].map(means)
```

**Groupby aggregates:**

```python
# Statistics of claim amounts per policyholder
agg = df.groupby('policy_id')['claim_amount'].agg(['mean', 'max', 'count'])
df = df.merge(agg, on='policy_id', how='left')
```

### Text Features

```python
from sklearn.feature_extraction.text import TfidfVectorizer

tfidf = TfidfVectorizer(max_features=10000, ngram_range=(1,2), sublinear_tf=True)
X_text = tfidf.fit_transform(df['description'])
```

For deep learning: use pre-trained embeddings (BERT, sentence-transformers) to encode semantic content.

### Feature Selection

**Filter methods:**

```python
from sklearn.feature_selection import mutual_info_classif, SelectKBest
selector = SelectKBest(mutual_info_classif, k=20)
X_selected = selector.fit_transform(X_train, y_train)
```

**Embedded methods:** lasso (L1 regularisation) shrinks unimportant coefficients to zero; random forest feature importance via mean impurity decrease.

**Variance threshold:** remove near-constant features:

```python
from sklearn.feature_selection import VarianceThreshold
sel = VarianceThreshold(threshold=0.01)
X_filtered = sel.fit_transform(X)
```

### Feature Leakage

A feature is **leaked** if it encodes information available only after the prediction time:

- **Target leakage**: a feature derived from the target (e.g., "claim indicator" in a model predicting claim amount — only exists after the claim is filed).
- **Temporal leakage**: using future data to predict past events (e.g., including a field updated at claim settlement when the prediction is at policy inception).

**Detection:** unexplainably high feature importance or correlation with target; performance collapses on live data.

**Prevention:** define a precise cutoff time and ensure all features are derived only from information available at that point.

## Applications

- GLMs for insurance pricing: log(exposure), interaction between vehicle age × driver age
- Gradient boosting on tabular data: ratio features, target encoding, lag features
- Fraud detection: velocity features (n_transactions in last 24h)

## Trade-offs

- More features increase model complexity and can cause overfitting; apply feature selection or regularisation.
- Leakage is the most dangerous failure mode; always validate features against the timeline.
- Automated feature engineering tools (featuretools, tsfresh) can generate thousands of features; filtering is essential.

## Links

- [[exploratory_data_analysis|Exploratory Data Analysis]]
- [[data_preprocessing|Data Preprocessing]]
- [[data_validation|Data Validation]]
- [[04_ml_engineering/03_feature_engineering/index|Feature Engineering (Production)]]
- [[02_modeling/03_model_families/01_linear_and_glm/index|Linear Models and GLMs]]
- [[01_foundations/01_linear_algebra/01_vector_spaces/vector_space|Vector Space — feature space geometry, basis transformations]]
- [[01_foundations/03_probability_and_statistics/probability_distributions|Probability Distributions — normalization, encoding categorical features]]
