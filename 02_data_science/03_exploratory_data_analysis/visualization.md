---
layer: 02_data_science
type: concept
status: growing
tags: [data, workflow]
created: 2026-03-06
---

# Data Visualization for Modelling

## Definition

The use of graphical representations to explore data distributions, relationships, model diagnostics, and results during the modelling workflow.

## Intuition

Humans process visual patterns far faster than numeric tables. Visualisation accelerates EDA, exposes model failure modes (e.g., residuals clustering by a feature), and communicates results to non-technical stakeholders. The right chart type depends on what you want to compare or reveal.

## Formal Description

### Chart Selection Guide

| Goal | Chart type |
|---|---|
| Distribution of one numeric variable | Histogram, KDE, box plot |
| Distribution of one categorical variable | Bar chart |
| Relationship between two numeric variables | Scatter plot |
| Relationship between numeric and categorical | Box plot, violin plot, strip plot |
| Correlation among many variables | Correlation heatmap |
| Change over time | Line plot |
| Proportion of a whole | Stacked bar, pie (sparingly) |
| Model calibration | Calibration curve |
| Classification performance | ROC curve, precision–recall curve, confusion matrix |
| Regression diagnostics | Residuals vs fitted, Q–Q plot |

### Distribution Plots

```python
import matplotlib.pyplot as plt
import seaborn as sns

# Histogram + KDE
fig, axes = plt.subplots(1, 2, figsize=(12, 4))
sns.histplot(df['age'], kde=True, ax=axes[0])
sns.boxplot(x='target', y='age', data=df, ax=axes[1])
plt.tight_layout()
```

**Q–Q plot** to check normality:

```python
from scipy.stats import probplot
probplot(df['residuals'], plot=plt)
plt.title('Q–Q Plot of Residuals')
```

Deviations from the diagonal indicate non-normality (heavy tails, skewness).

### Correlation Heatmap

```python
corr = df[num_cols].corr()
mask = np.triu(np.ones_like(corr, dtype=bool))  # upper triangle

sns.heatmap(corr, mask=mask, cmap='coolwarm', vmin=-1, vmax=1,
            annot=True, fmt='.2f', square=True)
plt.title('Feature Correlation Matrix')
```

### Scatter Plots with Regression Line

```python
sns.regplot(x='age', y='claim_amount', data=df, scatter_kws={'alpha': 0.3})
# Or: pairplot for multiple features
sns.pairplot(df[num_cols + ['target']], hue='target', corner=True)
```

### Classification Diagnostics

```python
from sklearn.metrics import RocCurveDisplay, PrecisionRecallDisplay, ConfusionMatrixDisplay

# ROC curve
RocCurveDisplay.from_estimator(model, X_test, y_test)

# Precision-Recall curve (better for imbalanced data)
PrecisionRecallDisplay.from_estimator(model, X_test, y_test)

# Confusion matrix
ConfusionMatrixDisplay.from_estimator(model, X_test, y_test, cmap='Blues')
```

### Regression Diagnostics

```python
y_pred = model.predict(X_test)
residuals = y_test - y_pred

fig, axes = plt.subplots(1, 2, figsize=(12, 5))

# Residuals vs fitted
axes[0].scatter(y_pred, residuals, alpha=0.3)
axes[0].axhline(0, color='red', linestyle='--')
axes[0].set_xlabel('Fitted values'); axes[0].set_ylabel('Residuals')
axes[0].set_title('Residuals vs Fitted')

# Residuals distribution
sns.histplot(residuals, kde=True, ax=axes[1])
axes[1].set_title('Residual Distribution')
```

A random scatter around 0 in the residuals-vs-fitted plot indicates a good fit. Patterns suggest heteroscedasticity or a missing feature.

### Calibration Plots

```python
from sklearn.calibration import CalibrationDisplay
CalibrationDisplay.from_estimator(model, X_test, y_test, n_bins=10)
```

A well-calibrated model lies on the diagonal: if it predicts 0.7 probability, about 70% of such cases are positive.

### Feature Importance Visualisation

```python
import pandas as pd

importance = pd.Series(model.feature_importances_, index=feature_names)
importance.sort_values().tail(20).plot(kind='barh', figsize=(8, 8))
plt.title('Top 20 Feature Importances')
```

### Learning Curves

Learning curves plot training and validation performance as a function of training set size — the primary visual tool for diagnosing the bias-variance tradeoff.

```python
from sklearn.model_selection import LearningCurveDisplay

LearningCurveDisplay.from_estimator(
    model, X_train, y_train,
    train_sizes=np.linspace(0.1, 1.0, 10),
    scoring='roc_auc',
    cv=5,
    n_jobs=-1
)
plt.title('Learning Curve')
```

**Interpretation:**
- **High bias (underfitting)**: both training and validation scores are low and converge to a similar low value. Adding more data won't help — increase model complexity.
- **High variance (overfitting)**: training score is high, validation score is much lower, and a gap persists. Adding more data may help; also try regularisation.
- **Good fit**: training and validation scores are close and both high.

### Embedding Visualisation (t-SNE / UMAP)

High-dimensional embeddings (e.g., from neural network layers, sentence transformers, or feature matrices) can be projected to 2-D for visual inspection.

```python
from sklearn.manifold import TSNE
import umap
import matplotlib.pyplot as plt

# t-SNE (better for local cluster structure)
reducer = TSNE(n_components=2, perplexity=30, random_state=42, init='pca')
Z = reducer.fit_transform(X)  # X: (n_samples, d)

# UMAP (faster, preserves global structure better)
reducer = umap.UMAP(n_components=2, random_state=42)
Z = reducer.fit_transform(X)

# Plot coloured by label
plt.figure(figsize=(8, 6))
scatter = plt.scatter(Z[:, 0], Z[:, 1], c=y, cmap='tab10', alpha=0.6, s=10)
plt.colorbar(scatter, label='Class')
plt.title('UMAP Embedding')
plt.tight_layout()
```

Use cases: checking whether classes are linearly separable, visualising cluster quality, debugging representations before a downstream task.
Caveat: distances in t-SNE output are not interpretable; only cluster membership is meaningful.

### SHAP Summary Plot

SHAP (SHapley Additive exPlanations) summary plots show feature impact across the entire dataset — a global explanation built from local attributions.

```python
import shap

explainer = shap.TreeExplainer(model)   # or shap.Explainer(model, X_train)
shap_values = explainer.shap_values(X_test)

# Beeswarm plot: feature importance + direction of effect
shap.summary_plot(shap_values, X_test, feature_names=feature_names)

# Bar chart: mean absolute SHAP values (magnitude only)
shap.summary_plot(shap_values, X_test, feature_names=feature_names, plot_type='bar')
```

Each dot represents one prediction; colour indicates feature value (red = high). Points to the right increase the prediction; points to the left decrease it.

See [[03_modeling/07_evaluation_and_model_selection/partial_dependence_ice|PDP, ICE, and ALE]] for marginal effect alternatives.

### Interactive Visualisation with Plotly

```python
import plotly.express as px

fig = px.scatter(df, x='age', y='claim_amount', color='target',
                 hover_data=['policy_id'], opacity=0.4)
fig.show()
```

Plotly is preferred for exploratory reports and dashboards; matplotlib/seaborn for publication-quality static figures.

## Applications

- Identifying that residuals increase with fitted value → heteroscedasticity → consider log-transforming target.
- Showing stakeholders that the model is well-calibrated at a steering committee.
- Spotting a bimodal age distribution → investigate whether two distinct customer segments should be modelled separately.

## Trade-offs

- Interactive visualisations (Plotly, Bokeh) are better for exploration but harder to embed in PDFs; use matplotlib for reports.
- Pair plots are $O(d^2)$ — intractable for many features; use correlation heatmap or PCA visualisation instead.

## References

- Tufte, E. R. (2001). *The Visual Display of Quantitative Information* (2nd ed.). Graphics Press.
- Wickham, H. (2010). "A Layered Grammar of Graphics." *Journal of Computational and Graphical Statistics*.
- Lundberg & Lee (2017). "A Unified Approach to Interpreting Model Predictions." NeurIPS. (SHAP)

## Links

- [[02_data_science/03_exploratory_data_analysis/exploratory_data_analysis|Exploratory Data Analysis]]
- [[02_data_science/04_feature_engineering/feature_engineering|Feature Engineering]]
- [[03_modeling/07_evaluation_and_model_selection/partial_dependence_ice|PDP, ICE, and ALE]]
- [[03_modeling/07_evaluation_and_model_selection/index|Evaluation and Validation]]
- [[01_foundations/03_probability_and_statistics/probability_distributions|Probability Distributions — density plots, histograms, Q-Q plots]]
- [[01_foundations/05_statistical_learning_theory/bias_variance_analysis|Bias–Variance Analysis — learning curves, error decomposition]]
