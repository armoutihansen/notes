---
layer: 02_modeling
type: concept
status: growing
tags: [data, workflow]
created: 2026-03-06
---

# Exploratory Data Analysis

## Definition

The initial, open-ended investigation of a dataset to understand its structure, distributions, missing values, outliers, and relationships before modelling.

## Intuition

EDA is how you build intuition about data before committing to any model. It catches data quality issues early, suggests feature engineering ideas, reveals class imbalance, and uncovers surprising relationships that can reshape the problem formulation.

## Formal Description

### Univariate Analysis

Examine each feature in isolation.

**For numerical features:**

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

df.describe()             # count, mean, std, min, quartiles, max
df[col].hist(bins=50)     # distribution shape
df[col].skew()            # > 1 or < -1: consider log transform
df[col].kurtosis()        # excess kurtosis vs Gaussian
```

**For categorical features:**

```python
df[col].value_counts()                       # frequency table
df[col].value_counts(normalize=True)         # proportions
df[col].nunique()                            # cardinality
```

High cardinality (> 50 unique values) flags potential encoding challenges.

### Missing Value Analysis

```python
missing = df.isnull().mean().sort_values(ascending=False)
# Threshold: > 40–50% missing → consider dropping the column
# Inspect whether missingness is MCAR, MAR, or MNAR
```

- **MCAR** (Missing Completely At Random): safe to ignore (but lose samples).
- **MAR** (Missing At Random): conditional on observed variables; imputable.
- **MNAR** (Missing Not At Random): missingness carries information; missingness indicator as a feature is often useful.

### Bivariate and Multivariate Analysis

**Numerical vs numerical:**

```python
# Correlation matrix
corr = df.corr(method='pearson')   # linear relationship
sns.heatmap(corr, annot=True, cmap='coolwarm')

# Scatter matrix
pd.plotting.scatter_matrix(df[num_cols], alpha=0.2, figsize=(12,12))
```

Note: correlation ≠ causation; check for non-linear relationships with scatter plots.

**Numerical vs target:**

```python
# For regression target
df.groupby('target')[num_col].describe()

# For classification target
df.boxplot(column=num_col, by='target')
```

**Categorical vs target:**

```python
df.groupby(cat_col)['target'].mean()    # mean target per category
pd.crosstab(df[cat_col], df['target'], normalize='index')
```

### Outlier Detection

**IQR method:**

```python
Q1, Q3 = df[col].quantile([0.25, 0.75])
IQR = Q3 - Q1
outliers = df[(df[col] < Q1 - 1.5*IQR) | (df[col] > Q3 + 1.5*IQR)]
```

**Z-score method:** flag observations with $|z| > 3$.

Outliers should be investigated, not automatically removed — they may be errors, or genuine extreme cases (e.g., large insurance claims).

### Class Imbalance

```python
df['target'].value_counts(normalize=True)
```

**Imbalanced classes (< 10% minority):** evaluate with PR-AUC or F1, not accuracy. Consider stratified sampling, SMOTE, class weights.

### Distribution Shape

| Characteristic | Detection | Action |
|---|---|---|
| Right skew | `skew() > 1` | Log or square-root transform |
| Heavy tails | High kurtosis, QQ plot vs Normal | Robust estimators, quantile regression |
| Multimodality | Histogram with multiple peaks | Consider subgroup modelling |
| Truncation | Values cut at hard boundary | Be aware of censoring |

## Applications

- Discovering that a numeric feature is actually a categorical code (all integers 0-4).
- Finding that a date feature has anomalous spikes (data collection errors).
- Identifying target leakage: a feature with suspiciously high correlation with the target.

## Trade-offs

- EDA is open-ended — time-box it. Use automated tools (e.g., `ydata-profiling`) for initial scans, then dig deeper into anomalies manually.
- Do not use the test set during EDA; restrict to training data only.

## Links

- [[feature_engineering|Feature Engineering]]
- [[data_preprocessing|Data Preprocessing]]
- [[02_modeling/01_problem_formulation/problem_formulation|Problem Formulation]]
- [[01_foundations/03_probability_and_statistics/probability_distributions|Probability Distributions]]
- [[04_ml_engineering/01_data_engineering/index|Data Engineering]]
