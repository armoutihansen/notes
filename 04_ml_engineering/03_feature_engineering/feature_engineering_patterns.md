---
layer: 04_ml_engineering
type: engineering
tool: general
status: growing
tags: [feature-engineering, preprocessing, transformations, embeddings]
created: 2026-03-05
---

# Feature Engineering Patterns

## Purpose

Feature engineering is the process of transforming raw data into representations that better capture the underlying structure for a given learning algorithm. Well-engineered features can dramatically improve model performance, reduce the need for large amounts of data, and make models more interpretable. Even with the rise of deep learning and automated feature learning, deliberate feature engineering remains valuable for tabular data, time-series, and domains where domain knowledge is available.

## Architecture

### Numeric Features

**Scaling** is applied to align feature magnitudes and prevent gradient-based learners from being dominated by high-variance columns.

- **Min-max normalization**: $x' = (x - x_{min}) / (x_{max} - x_{min})$ — maps values to [0, 1]; sensitive to outliers.
- **Standardization (z-score)**: $x' = (x - \mu) / \sigma$ — zero mean, unit variance; preferred for most linear models and neural networks.
- **Log transform**: $x' = \log(1 + x)$ — compresses right-skewed distributions (e.g., income, counts); use `log1p` for zero-safe behavior.
- **Robust scaling**: uses median and IQR instead of mean/std; more resistant to outliers.
- **Power transforms** (Box-Cox, Yeo-Johnson): parametric family that maps data toward normality; Yeo-Johnson handles negative values.

### Categorical Features

- **Ordinal encoding**: assigns integers according to a meaningful order (e.g., Low=0, Medium=1, High=2). Only valid when order is semantically meaningful.
- **One-hot encoding**: creates a binary column per category. Suitable for low-cardinality features; can explode dimensionality for high-cardinality columns.
- **Target encoding**: replaces each category with the mean of the target variable within that category. Powerful for high-cardinality features but prone to target leakage — must be computed within each cross-validation fold.
- **Hashing trick**: maps category strings to a fixed-size vector via a hash function. Handles unseen categories gracefully; collisions introduce noise but are often acceptable.
- **Embeddings**: learned dense representations; can be shared across multiple features (e.g., entity embeddings in neural networks for tabular data).

### Feature Crosses and Interaction Terms

Feature crosses explicitly capture non-linear relationships between pairs (or higher-order combinations) of features. In linear models, crossing features `A` and `B` creates a new feature `A × B` that allows the model to learn an interaction effect. In tree-based models, interactions are learned automatically, so manual crosses are less impactful. In neural networks, explicit crosses (as in Wide & Deep) improve memorization of specific co-occurrence patterns.

### Embedding Features

- **Pre-trained embeddings**: leverage representations learned on large corpora (e.g., word2vec, GloVe, BERT token embeddings, OpenAI text-embedding-ada). Useful when training data is small.
- **Learned embeddings**: trained end-to-end on the task. Entity embeddings for categorical IDs (user IDs, product IDs) are a common pattern in recommender systems and click-through rate models.
- Embeddings reduce the curse of dimensionality compared to one-hot encoding and capture semantic similarity in the embedding space.

### Temporal Features

Raw timestamps are rarely useful directly. Common transformations include:

- **Lag features**: the value of a variable at time $t - k$. Captures auto-correlation and recent history.
- **Rolling window statistics**: mean, std, min, max over a trailing window (e.g., 7-day average sales). Must be computed without look-ahead to avoid leakage.
- **Time-of-day / day-of-week encodings**: cyclical encoding using $\sin(2\pi x / T)$ and $\cos(2\pi x / T)$ preserves the circular nature of time (e.g., 23:00 and 00:00 are adjacent).
- **Time since event**: age since last purchase, account creation date, etc.

## Implementation Notes

- All transformations that depend on statistics computed from training data (mean, std, min, max, target means) must be fit on training data only, then applied to validation and test sets via a `Pipeline` (scikit-learn) to prevent leakage.
- Use `sklearn.pipeline.Pipeline` and `ColumnTransformer` to bundle preprocessing with model training as a single artifact.
- Feature stores (Feast, Tecton, Hopsworks) centralize feature computation and serve precomputed features at low latency in production, ensuring training-serving consistency.

## Trade-offs

**Learned vs. engineered features**: Deep neural networks can learn complex feature representations from raw data, reducing the need for manual engineering. However, tabular data with limited samples often benefits more from domain-driven feature engineering than from learned representations. The cost of manual feature engineering is development time and potential for overfitting to historical patterns; the cost of relying solely on learned features is data inefficiency and opacity.

**Feature selection**: Too many features increases training time, memory usage, and risk of overfitting.
- **Filter methods**: rank features by statistical association with the target (mutual information, chi-squared, correlation). Fast but ignores feature interactions.
- **Wrapper methods**: use model performance to evaluate feature subsets (recursive feature elimination, forward/backward selection). More accurate but expensive.
- **Embedded methods**: regularization (L1/Lasso) drives feature weights to zero during training, performing implicit selection.

## References

- Zheng & Casari, *Feature Engineering for Machine Learning* (O'Reilly)
- Guo & Berkhahn, "Entity Embeddings of Categorical Variables" (2016)
- scikit-learn: `sklearn.preprocessing`, `sklearn.pipeline.Pipeline`

## Links
- [[data_leakage|Data Leakage]]
- [[offline_evaluation|Offline Evaluation]]
- [[experiment_tracking|Experiment Tracking]]
- [[regularization|Regularization]]
