---
layer: 03_modeling
type: application
status: growing
tags: [algorithm, clustering, anomaly-detection]
created: 2026-03-06
---

# Unsupervised Learning — Implementation

## Goal

Implement clustering, dimensionality reduction, and anomaly detection with scikit-learn and UMAP.

## Conceptual Counterpart

- [[03_modeling/02_unsupervised_learning/index|Unsupervised Learning]] — clustering algorithms, PCA, anomaly detection methods
- [[07_applications/03_detection_and_monitoring/anomaly_detection_operations|Anomaly Detection — Operations]] — production context for unsupervised anomaly detection
- [[07_applications/08_domain_verticals/06_operations/quality_control_vision|Quality Control Vision]] — manufacturing anomaly detection application

## Purpose

Practical implementation of clustering, dimensionality reduction, and anomaly detection with scikit-learn and UMAP.

### Examples

- K-means clustering with elbow and silhouette selection
- DBSCAN for density-based clustering
- PCA for dimensionality reduction and explained variance
- t-SNE and UMAP for 2D visualisation
- Isolation Forest for anomaly detection

## Architecture

```
Raw features → StandardScaler
             → Dimensionality reduction (optional) → PCA / UMAP
             → Clustering / anomaly detection algorithm
             → Cluster labels / anomaly scores
```

## Implementation

### Setup

```bash
pip install scikit-learn umap-learn
```

### K-Means Clustering

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import silhouette_score
from sklearn.datasets import make_blobs

X, _ = make_blobs(n_samples=500, centers=4, random_state=42)
X = StandardScaler().fit_transform(X)

# Elbow method
inertias = []
for k in range(2, 11):
    km = KMeans(n_clusters=k, n_init=10, random_state=42)
    km.fit(X)
    inertias.append(km.inertia_)

# Silhouette score
sil_scores = [
    silhouette_score(X, KMeans(n_clusters=k, n_init=10, random_state=42).fit_predict(X))
    for k in range(2, 11)
]
best_k = np.argmax(sil_scores) + 2
print(f"Best k by silhouette: {best_k}")

km_final = KMeans(n_clusters=best_k, n_init=10, random_state=42)
labels = km_final.fit_predict(X)
```

### DBSCAN

```python
from sklearn.cluster import DBSCAN
from sklearn.neighbors import NearestNeighbors

# Estimate eps using k-distance graph
nbrs = NearestNeighbors(n_neighbors=5).fit(X)
dists, _ = nbrs.kneighbors(X)
kth_dist = np.sort(dists[:, -1])
# Plot kth_dist — choose eps at the "elbow"

db = DBSCAN(eps=0.5, min_samples=5)
labels = db.fit_predict(X)
n_clusters = len(set(labels)) - (1 if -1 in labels else 0)
n_noise    = (labels == -1).sum()
print(f"Clusters: {n_clusters}, Noise points: {n_noise}")
```

### PCA

```python
from sklearn.decomposition import PCA

pca = PCA()
pca.fit(X)

# Cumulative explained variance
cum_var = np.cumsum(pca.explained_variance_ratio_)
n_components_95 = np.argmax(cum_var >= 0.95) + 1
print(f"Components for 95% variance: {n_components_95}")

# Reduce
pca_final = PCA(n_components=n_components_95)
X_reduced = pca_final.fit_transform(X)
```

### t-SNE (2D Visualisation)

```python
from sklearn.manifold import TSNE

tsne = TSNE(
    n_components=2,
    perplexity=30,      # 5–50; higher → more global structure
    learning_rate="auto",
    n_iter=1000,
    random_state=42
)
X_2d = tsne.fit_transform(X)

plt.figure(figsize=(8, 6))
plt.scatter(X_2d[:, 0], X_2d[:, 1], c=labels, cmap="tab10", s=5, alpha=0.7)
plt.title("t-SNE")
```

### UMAP (faster and preserves global structure better)

```python
import umap

reducer = umap.UMAP(
    n_components=2,
    n_neighbors=15,    # local vs global balance (smaller → local)
    min_dist=0.1,      # compactness of clusters
    random_state=42
)
X_umap = reducer.fit_transform(X)
```

### Isolation Forest (Anomaly Detection)

```python
from sklearn.ensemble import IsolationForest

iso = IsolationForest(
    n_estimators=100,
    contamination=0.05,   # expected fraction of anomalies
    random_state=42
)
scores = iso.fit_predict(X)    # -1 = anomaly, 1 = inlier
anomaly_scores = iso.score_samples(X)  # lower → more anomalous
print(f"Anomalies found: {(scores == -1).sum()}")
```

## Trade-offs

- K-means assumes spherical clusters of equal size; use DBSCAN for arbitrary shapes or when the number of clusters is unknown.
- t-SNE preserves local structure but distances are not meaningful across distant clusters; UMAP is faster and better for large datasets.
- PCA is deterministic and invertible; non-linear methods (t-SNE, UMAP) are not.
- Isolation Forest is efficient for high-dimensional anomaly detection; Local Outlier Factor is better for density-based anomalies.

## Links

- [[03_modeling/02_unsupervised_learning/01_clustering/unsupervised_learning|Unsupervised Learning (theory)]]
- [[02_data_science/03_exploratory_data_analysis/exploratory_data_analysis|EDA (initial data profiling)]]
- [[03_modeling/07_evaluation_and_model_selection/evaluation_and_validation|Evaluation: silhouette, metrics]]
- [[01_foundations/01_linear_algebra/03_eigenvalues_and_decompositions/index|SVD / PCA foundations]]
