---
layer: 02_modeling
type: concept
status: growing
tags: [algorithm, clustering, anomaly-detection]
created: 2026-03-06
---

# Unsupervised Learning

## Definition

Learning structure from unlabelled data: discovering clusters, reducing dimensionality, or modelling the data distribution without a target variable.

## Intuition

When you have data but no labels, you can still find structure: natural groupings, low-dimensional representations, or anomalous points. Unsupervised methods are used as standalone models, as preprocessing for downstream supervised learning, or as tools for data exploration.

## Formal Description

### K-Means Clustering

Partitions $n$ points into $K$ clusters by alternating assignment and centroid update.

**Objective:** minimise within-cluster sum of squares (WCSS):

$$\min_{\{C_k\}} \sum_{k=1}^K \sum_{\mathbf{x} \in C_k} \|\mathbf{x} - \boldsymbol{\mu}_k\|^2$$

**Algorithm (Lloyd's):**
1. Initialise $K$ centroids (k-means++ for better initialisation).
2. **Assign:** $z_i = \arg\min_k \|\mathbf{x}_i - \boldsymbol{\mu}_k\|^2$
3. **Update:** $\boldsymbol{\mu}_k = \frac{1}{|C_k|}\sum_{\mathbf{x} \in C_k}\mathbf{x}$
4. Repeat 2–3 until convergence.

Guaranteed to converge but to a local minimum; run multiple times (`n_init` in sklearn).

**Choosing K:** elbow plot (WCSS vs K), silhouette score, or gap statistic.

**Silhouette coefficient** for point $i$:
$$s_i = \frac{b_i - a_i}{\max(a_i, b_i)}$$
where $a_i$ = mean intra-cluster distance, $b_i$ = mean distance to nearest other cluster. Range: $[-1, 1]$; higher is better.

**Limitations:** assumes spherical, equal-variance clusters; sensitive to outliers; $K$ must be specified.

### DBSCAN (Density-Based Spatial Clustering)

Groups points that are **density-connected**: connected by chains of core points (points with $\geq$ `min_samples` within radius `eps`).

**Point types:**
- **Core point:** $\geq$ `min_samples` points within `eps`.
- **Border point:** within `eps` of a core point but not itself a core point.
- **Noise (outlier):** not within `eps` of any core point.

**Properties:**
- Discovers arbitrary cluster shapes (not just spherical).
- No need to specify $K$; automatically identifies outliers.
- Requires tuning `eps` and `min_samples`; use k-distance plot to choose `eps`.
- Struggles in high dimensions (curse of dimensionality).

### PCA (Principal Component Analysis)

Finds the orthogonal directions of maximum variance and projects data onto them.

**Formulation:** find $W \in \mathbb{R}^{d \times k}$ with orthonormal columns that maximise $\mathrm{Var}[W^\top X]$:

$$W = \text{top-}k\text{ eigenvectors of }\Sigma = \frac{1}{n}X^\top X$$

Equivalently, the top-$k$ right singular vectors of $X$ from SVD: $X = U\Sigma V^\top$.

**Explained variance ratio:** $\lambda_j / \sum_i \lambda_i$ for eigenvalue $\lambda_j$; choose $k$ such that cumulative variance ≥ 95%.

**Whitening PCA:** additionally scales to unit variance: $Z = \Lambda^{-1/2}W^\top X$.

**Use cases:** visualisation (2D/3D PCA plots), noise reduction, preprocessing before clustering, feature compression.

**Limitations:** captures only linear structure; highly non-linear manifolds require t-SNE, UMAP, or autoencoders.

### t-SNE and UMAP (Visualisation)

Non-linear dimensionality reduction for 2D/3D visualisation.

**t-SNE:** preserves local neighbourhood structure; non-deterministic; use for visualisation only (not as features).

**UMAP:** faster than t-SNE, better preserves global structure, can be used as a preprocessing step.

### Isolation Forest (Anomaly Detection)

Anomalies are isolated more quickly by random partitioning. The **anomaly score** is the average path length to isolation across trees:

- Short path length → anomalous (easy to isolate).
- Long path length → normal (requires many splits to isolate).

**Contamination parameter:** expected fraction of anomalies; controls the decision threshold.

**Alternatives:**
- **One-Class SVM:** learns a tight boundary around normal data.
- **Local Outlier Factor (LOF):** density-based; flags points in low-density regions relative to neighbours.
- **Autoencoder reconstruction error:** points that reconstruct poorly are anomalous.

## Applications

- K-means: customer segmentation, image compression (vector quantisation), feature codebooks
- DBSCAN: geospatial clustering (delivery zones), noise-robust customer segmentation
- PCA: visualise high-dimensional embeddings, remove correlated features before linear models
- Isolation Forest: fraud detection, intrusion detection, data quality checks

## Trade-offs

| Algorithm | Cluster shape | Noise robust | K required | Scalability |
|---|---|---|---|---|
| K-means | Spherical | No | Yes | ✅ $O(nKd)$ |
| DBSCAN | Arbitrary | Yes | No | Moderate $O(n\log n)$ |
| GMM | Ellipsoidal | No | Yes | Moderate |
| Spectral | Arbitrary | No | Yes | ❌ $O(n^3)$ |

## Links

- [[01_foundations/01_linear_algebra/03_eigenvalues/eigenvalues_and_eigenvectors|Eigenvalues and Eigenvectors (PCA)]]
- [[01_foundations/01_linear_algebra/04_linear_systems/lu_decomposition|LU Decomposition (SVD relationship)]]
- [[02_modeling/03_model_families/03_probabilistic_models/probabilistic_models|Probabilistic Models (GMM)]]
- [[02_modeling/02_data_science/exploratory_data_analysis|Exploratory Data Analysis]]
- [[02_modeling/06_interpretability/interpretability_overview|Interpretability Overview]]
