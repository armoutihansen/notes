---
layer: 03_modeling
type: concept
status: growing
tags: [algorithm, clustering]
created: 2026-03-06
---

# Clustering

## Definition

Clustering partitions an unlabelled dataset into groups (clusters) such that points within a cluster are more similar to each other than to points in other clusters. It is the primary task in unsupervised learning when the goal is to discover discrete structure in data.

## Intuition

Given $n$ data points with no labels, clustering surfaces natural groupings. The notion of "similar" is algorithm-specific: K-Means uses Euclidean distance to centroid; DBSCAN uses local density; GMM uses probabilistic mixture components. The right algorithm depends on expected cluster shape, the need to handle noise, and whether $K$ is known a priori.

## Formal Description

### K-Means

Partitions $n$ points into $K$ clusters by minimising within-cluster sum of squares (WCSS):

$$
\min_{\{C_k\}} \sum_{k=1}^K \sum_{\mathbf{x} \in C_k} \|\mathbf{x} - \boldsymbol{\mu}_k\|^2
$$

**Algorithm (Lloyd's):**
1. Initialise $K$ centroids — k-means++ samples the first centroid uniformly, then picks each subsequent centroid with probability proportional to $d(\mathbf{x}, \text{nearest centroid})^2$, dramatically reducing the chance of a bad initialisation.
2. **Assign:** $z_i = \arg\min_k \|\mathbf{x}_i - \boldsymbol{\mu}_k\|^2$
3. **Update:** $\boldsymbol{\mu}_k = \frac{1}{|C_k|}\sum_{\mathbf{x} \in C_k}\mathbf{x}$
4. Repeat 2–3 until convergence.

Guaranteed to converge but only to a local minimum; set `n_init ≥ 10` in sklearn to run multiple restarts and keep the best.

**Choosing K:**
- *Elbow method:* plot WCSS vs $K$; pick the elbow where marginal gains flatten.
- *Silhouette score:* pick $K$ maximising mean silhouette; more principled than the elbow.
- *Gap statistic:* compare within-cluster dispersion to that of a reference random distribution.

**Silhouette coefficient** for point $i$:
$$
s_i = \frac{b_i - a_i}{\max(a_i, b_i)}
$$
where $a_i$ = mean intra-cluster distance, $b_i$ = mean distance to nearest other cluster. Range: $[-1, 1]$; values near 1 indicate well-separated clusters.

**Limitations:** assumes spherical, equal-variance clusters; sensitive to outliers and feature scaling; $K$ must be specified.

### DBSCAN (Density-Based Spatial Clustering of Applications with Noise)

Groups points that are **density-connected**: reachable through chains of core points (points with $\geq$ `min_samples` neighbours within radius `eps`).

**Point classification:**
- **Core point:** has $\geq$ `min_samples` points within `eps`.
- **Border point:** within `eps` of a core point but below density threshold itself.
- **Noise point:** not within `eps` of any core point; assigned label $-1$.

**Hyperparameter selection:**
- `eps`: use the *k-distance graph* — sort the $k$-th nearest-neighbour distances and pick the knee.
- `min_samples`: rule of thumb $\geq d + 1$ where $d$ is the feature dimension; larger values suppress noise.

**Properties:**
- Discovers arbitrary-shaped clusters with no need to specify $K$.
- Explicitly labels outliers — useful for anomaly detection.
- Sensitive to varying densities across clusters; HDBSCAN addresses this by building a hierarchy over multiple `eps` values.
- Struggles in high dimensions due to the curse of dimensionality making distance measures less discriminative.

### HDBSCAN (Hierarchical DBSCAN)

Extends DBSCAN by building a cluster hierarchy across all density levels and extracting the most stable clusters. Key advantage: no need to choose `eps`; only `min_cluster_size` is required. Available in sklearn ≥ 1.3 as `sklearn.cluster.HDBSCAN`.

### Hierarchical Clustering (Agglomerative)

Builds a nested sequence of partitions by merging the closest pair of clusters bottom-up. The result is a **dendrogram** — cut at a chosen height to obtain $K$ clusters.

**Linkage criteria** (determines "distance between clusters"):

| Linkage | Merges based on | Tendency |
|---|---|---|
| `ward` | Minimise increase in total WCSS | Compact, equally-sized clusters |
| `complete` | Maximum pairwise distance | Avoids elongated clusters |
| `average` | Mean pairwise distance | Balanced |
| `single` | Minimum pairwise distance | Chaining — sensitive to outliers |

Ward linkage is the default in sklearn and usually gives the most interpretable results.

**Complexity:** $O(n^2 \log n)$ time; impractical for $n > 10^4$. Use `n_clusters` to cut the dendrogram, or `distance_threshold` to cut by height.

### GMM as Soft Clustering

A **Gaussian Mixture Model** assigns each point a *soft* probability of belonging to each of $K$ Gaussian components, rather than a hard label. K-Means is a degenerate GMM in the limit of zero covariance within clusters.

$$
p(\mathbf{x}) = \sum_{k=1}^K \pi_k \, \mathcal{N}(\mathbf{x}; \boldsymbol{\mu}_k, \Sigma_k)
$$

GMM handles ellipsoidal clusters (non-spherical covariance) and produces probabilistic assignments via `predict_proba()`. Select $K$ via BIC/AIC. Full theory is in [[03_modeling/03_probabilistic_models/03_bayesian_modeling/probabilistic_models|Probabilistic Models]].

## Applications

- **K-Means:** customer segmentation, image compression (vector quantisation), codebook generation for bag-of-words models.
- **DBSCAN / HDBSCAN:** geospatial clustering of GPS traces, noise-robust document clustering, satellite imagery analysis.
- **Hierarchical:** taxonomic/biological classification, document taxonomy, exploratory analysis where the full dendrogram is informative.
- **GMM:** speaker identification, background modelling in computer vision, initialisation for EM in HMMs.

## Trade-offs

| Algorithm | Cluster shape | Handles noise | $K$ required | Scalability | Soft assignments |
|---|---|---|---|---|---|
| K-Means | Spherical | No | Yes | ✅ $O(nKd)$ | No |
| DBSCAN | Arbitrary | Yes (explicit) | No | $O(n \log n)$ | No |
| HDBSCAN | Arbitrary | Yes (explicit) | No (min_size only) | $O(n \log n)$ | Soft available |
| Agglomerative | Arbitrary (linkage-dep.) | No | Yes (cut height) | ❌ $O(n^2 \log n)$ | No |
| GMM | Ellipsoidal | No | Yes | $O(nKd^2)$ | Yes |

## References

- Arthur, D. & Vassilvitskii, S. (2007). "k-means++: The advantages of careful seeding." *SODA*.
- Ester, M. et al. (1996). "A density-based algorithm for discovering clusters in large spatial databases with noise." *KDD*.
- Campello, R.J.G.B. et al. (2013). "Density-based clustering based on hierarchical density estimates." *PAKDD* (HDBSCAN).
- sklearn clustering user guide: https://scikit-learn.org/stable/modules/clustering.html

## Links

- [[03_modeling/02_unsupervised_learning/01_clustering/index|Clustering Sublayer Index]]
- [[03_modeling/02_unsupervised_learning/01_clustering/clustering_implementation|Clustering — Implementation]]
- [[01_foundations/03_probability_and_statistics/index|Probability and Statistics — GMM foundations]]
- [[03_modeling/03_probabilistic_models/03_bayesian_modeling/probabilistic_models|Probabilistic Models — GMM full derivation]]
