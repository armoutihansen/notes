---
layer: 03_modeling
type: concept
status: growing
tags: [algorithm, theory]
created: 2026-03-06
---

# Graphical Models

## Definition

Probabilistic graphical models (PGMs) represent joint distributions over many variables using a graph, where nodes are variables and edges encode conditional (in)dependencies. They provide a compact, interpretable representation of complex multivariate distributions.

## Intuition

A joint distribution over $d$ binary variables requires $2^d - 1$ parameters — intractable for large $d$. Graphical models exploit conditional independence structure to represent the same distribution with far fewer parameters, and to perform efficient inference.

## Formal Description

### Directed Graphical Models (Bayesian Networks)

A **Bayesian network** (Belief Network) is a DAG where each node $X_i$ has a conditional probability table (CPT): $P(X_i | \text{Pa}(X_i))$.

**Joint distribution factorises as:**

$$
P(X_1, \ldots, X_d) = \prod_{i=1}^d P(X_i \mid \text{Pa}(X_i))
$$

**D-separation:** a criterion for reading off conditional independence from the graph structure. Key patterns:

| Structure | Relationship |
|---|---|
| Chain: $A \to B \to C$ | $A \perp C \mid B$ (B blocks the path) |
| Fork: $A \leftarrow B \to C$ | $A \perp C \mid B$ (common cause blocked by B) |
| Collider: $A \to B \leftarrow C$ | $A \not\perp C \mid B$ (conditioning on B opens the path) |

**Parameter learning (MLE):** for complete data, each CPT is estimated independently by counting.

**Structure learning:** score-based (maximise BIC/BDe over DAG structures) or constraint-based (PC algorithm: test conditional independence).

### Undirected Graphical Models (Markov Random Fields)

Nodes connected by edges encoding potential functions:

$$
P(\mathbf{x}) = \frac{1}{Z}\prod_{C \in \mathcal{C}} \psi_C(\mathbf{x}_C)
$$

where $\mathcal{C}$ are cliques and $Z = \sum_\mathbf{x}\prod_C \psi_C$ is the partition function.

**Markov property:** $X_i \perp X_j \mid \mathbf{x}_{\setminus\{i,j\}}$ for non-adjacent $i,j$.

**Applications:** image segmentation (each pixel is a node), social networks.

**Gibbs random fields / Ising model:** pairwise potentials on a grid; extensively studied in physics and CV.

### Inference

**Exact inference:**
- **Variable elimination:** sum out variables in an ordering. Complexity depends on the treewidth.
- **Belief propagation (sum-product):** message-passing on trees (exact) or graphs (loopy BP, approximate).
- **Junction tree algorithm:** exact inference via converting the graph to a junction tree.

**Approximate inference:**
- **MCMC (Gibbs sampling):** iteratively sample each variable from its conditional given current values.
- **Variational inference (mean field):** minimise $\mathrm{KL}(q(\mathbf{x})\|p(\mathbf{x}))$ over a factorised family $q$.

### Hidden Markov Model (HMM)

A sequential Bayesian network with latent states $z_1,\ldots,z_T$ and observations $x_1,\ldots,x_T$:

$$
P(\mathbf{x},\mathbf{z}) = P(z_1)\prod_{t=2}^T P(z_t|z_{t-1})\prod_{t=1}^T P(x_t|z_t)
$$

**Key algorithms:**
- **Forward-backward (Baum-Welch):** E-step for EM; computes $P(z_t|\mathbf{x})$.
- **Viterbi:** finds the most likely state sequence $\hat{\mathbf{z}} = \arg\max_\mathbf{z} P(\mathbf{z}|\mathbf{x})$ via dynamic programming.
- **EM (Baum-Welch):** learns transition and emission parameters from unlabelled sequences.

## Applications

- **Bayesian networks:** clinical decision support, root cause analysis, causal reasoning
- **MRF:** image segmentation, medical image analysis
- **HMM:** speech recognition, CRF-based NER, gene finding, financial regime detection

## Trade-offs

- Exact inference is intractable for general graphs (NP-hard); tree-structured graphs are tractable.
- Bayesian networks require careful structure specification or expensive structure learning.
- Modern deep learning has largely replaced graphical models for perception tasks but PGMs remain valuable for causal modelling and interpretability.

## Links

- [[01_foundations/03_probability_and_statistics/bayesian_inference|Bayesian Inference]]
- [[01_foundations/03_probability_and_statistics/probability_theory|Conditional Independence]]
- [[03_modeling/03_probabilistic_models/03_bayesian_modeling/probabilistic_models|Probabilistic Models (GMM, Naive Bayes)]]
- [[03_modeling/05_time_series/01_classical_forecasting/time_series_models|Time Series Models (HMM)]]
