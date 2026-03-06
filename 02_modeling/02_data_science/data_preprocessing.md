---
layer: 02_modeling
type: concept
status: growing
tags: [data, workflow]
created: 2026-03-06
---

# Data Preprocessing

## Definition

Transformations applied to raw data to make it suitable for modelling: handling missing values, encoding categorical variables, scaling numerical features, and structuring data into model-ready arrays.

## Intuition

Most ML algorithms assume clean, numeric, fixed-dimensional input. Preprocessing bridges the gap between raw real-world data and model expectations. The choices made here directly affect model performance and must be fitted only on training data to prevent data leakage.

## Formal Description

### Missing Value Imputation

**Simple strategies:**

```python
from sklearn.impute import SimpleImputer

# Numerical: mean, median, or constant
imp_num = SimpleImputer(strategy='median')

# Categorical: most frequent or constant
imp_cat = SimpleImputer(strategy='most_frequent')
```

**Iterative / model-based imputation:**

```python
from sklearn.experimental import enable_iterative_imputer  # noqa
from sklearn.impute import IterativeImputer

imp = IterativeImputer(max_iter=10, random_state=0)
X_imputed = imp.fit_transform(X_train)
```

`IterativeImputer` models each feature as a function of all others. More accurate but slower.

**Missingness indicator:** add a binary flag column for MNAR features:

```python
from sklearn.impute import MissingIndicator
ind = MissingIndicator()
X_miss_flags = ind.fit_transform(X_train)
```

### Scaling Numerical Features

Scale before algorithms that use distances or gradient-based optimization (SVMs, KNNs, neural networks, regularised regression). Tree-based models are scale-invariant.

```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler, RobustScaler

# Standardisation: (x - mean) / std — assumes approx Gaussian
scaler = StandardScaler()

# Min-Max scaling: rescales to [0, 1]
scaler = MinMaxScaler()

# Robust scaling: uses median and IQR — less sensitive to outliers
scaler = RobustScaler()

X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled  = scaler.transform(X_test)   # use training statistics
```

**Critical:** always call `fit_transform` on training data and `transform` only on validation/test.

### Encoding Categorical Features

```python
from sklearn.preprocessing import OrdinalEncoder, OneHotEncoder

# Ordinal: preserves order (e.g., low < medium < high)
enc = OrdinalEncoder(categories=[['low', 'medium', 'high']])

# One-hot: creates K binary columns (or K-1 with drop='first')
enc = OneHotEncoder(drop='first', sparse_output=False)
```

**High-cardinality categories:** use target encoding, frequency encoding, or embedding layers.

```python
# Target encoding (mean target per category, computed on train only)
target_means = X_train.groupby(cat_col)['target'].mean()
X_train[cat_col + '_enc'] = X_train[cat_col].map(target_means)
X_test[cat_col + '_enc']  = X_test[cat_col].map(target_means).fillna(global_mean)
```

Target encoding risks leakage: compute within cross-validation folds.

### Sklearn Pipelines

Pipelines chain transformers and an estimator, ensuring correct fit/transform discipline:

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer

num_pipeline = Pipeline([
    ('impute', SimpleImputer(strategy='median')),
    ('scale',  StandardScaler()),
])

cat_pipeline = Pipeline([
    ('impute', SimpleImputer(strategy='most_frequent')),
    ('encode', OneHotEncoder(handle_unknown='ignore')),
])

preprocessor = ColumnTransformer([
    ('num', num_pipeline, num_cols),
    ('cat', cat_pipeline, cat_cols),
])

full_pipeline = Pipeline([
    ('prep',  preprocessor),
    ('model', LinearRegression()),
])

full_pipeline.fit(X_train, y_train)
full_pipeline.score(X_test, y_test)
```

### Handling Skewed Distributions

```python
import numpy as np

# Log transform for right-skewed, strictly positive features
X_train['col_log'] = np.log1p(X_train['col'])  # log(1+x)

# Box-Cox: finds optimal power transform (requires positive values)
from sklearn.preprocessing import PowerTransformer
pt = PowerTransformer(method='box-cox')

# Yeo-Johnson: works with negative values
pt = PowerTransformer(method='yeo-johnson')
```

### Outlier Handling

Options: cap (Winsorize), transform, or flag and let the model handle them.

```python
# Winsorize: cap at 1st and 99th percentile
lower, upper = df[col].quantile([0.01, 0.99])
df[col] = df[col].clip(lower, upper)
```

## Applications

- Insurance GLMs require log-transformed claim amounts and one-hot-encoded categorical risk factors.
- Neural networks for tabular data benefit from StandardScaler on continuous features and embedding layers for high-cardinality categoricals.

## Trade-offs

- **Mean vs median imputation**: mean is distorted by outliers; median is safer.
- **Standardisation vs min-max**: StandardScaler is preferred unless the algorithm specifically requires $[0,1]$ inputs (e.g., certain neural net architectures with sigmoid inputs).
- **One-hot vs ordinal**: one-hot is preferred for unordered categories with low cardinality; ordinal encoding can introduce spurious ordinal relationships.

## Links

- [[exploratory_data_analysis|Exploratory Data Analysis]]
- [[feature_engineering|Feature Engineering]]
- [[data_validation|Data Validation]]
- [[04_ml_engineering/03_feature_engineering/index|Feature Engineering (Production)]]
- [[01_foundations/03_probability_and_statistics/probability_distributions|Probability Distributions]]
