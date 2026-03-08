---
layer: 01_foundations
type: concept
status: growing
tags: [theory]
created: 2026-03-06
---

# VC Dimension

## Definition

The **Vapnik-Chervonenkis (VC) dimension** of a hypothesis class $\mathcal{H}$ over input space $\mathcal{X}$ is the size of the largest set $S \subseteq \mathcal{X}$ that $\mathcal{H}$ **shatters**:

$$
\text{VCdim}(\mathcal{H}) = \max\{|S| : S \subseteq \mathcal{X},\ \mathcal{H} \text{ shatters } S\}
$$

A set $S$ is **shattered** by $\mathcal{H}$ if every possible binary labelling of $S$ is realised by some $h \in \mathcal{H}$.

## Intuition

VC dimension measures the "expressive capacity" of a hypothesis class — how complex a binary pattern can it learn? If you can find a set of $d$ points that $\mathcal{H}$ labels in all $2^d$ possible ways, then $\text{VCdim}(\mathcal{H}) \geq d$. But if for every set of $d+1$ points there's some labelling $\mathcal{H}$ can't produce, then $\text{VCdim}(\mathcal{H}) \leq d$.

Higher VC dimension = more expressive = needs more data to learn without overfitting. The VC dimension is the right notion of "degrees of freedom" for a hypothesis class.

## Formal Description

**Growth function $\Pi_\mathcal{H}(m)$:** the maximum number of distinct labellings that $\mathcal{H}$ can produce on any $m$ points:

$$
\Pi_\mathcal{H}(m) = \max_{S \subseteq \mathcal{X},|S|=m} |\{(h(x_1),\ldots,h(x_m)) : h \in \mathcal{H}\}|
$$

**Sauer-Shelah lemma:** if $\text{VCdim}(\mathcal{H}) = d < \infty$, then:

$$
\Pi_\mathcal{H}(m) \leq \sum_{i=0}^d \binom{m}{i} = O(m^d)
$$

This transitions from exponential growth ($2^m$, when $m \leq d$) to polynomial growth. The key insight: once you have more data than the VC dimension, the class is "effectively finite", enabling generalisation.

**Fundamental theorem of statistical learning:** $\mathcal{H}$ is agnostic PAC learnable if and only if $\text{VCdim}(\mathcal{H}) < \infty$. The sample complexity is:

$$
m \asymp \frac{d + \log(1/\delta)}{\epsilon^2} \quad \text{(agnostic)} \qquad m \asymp \frac{d + \log(1/\delta)}{\epsilon} \quad \text{(realizable)}
$$

where $d = \text{VCdim}(\mathcal{H})$.

**Standard VC dimensions:**

| Hypothesis class | VC dimension |
|---|---|
| Halfspaces in $\mathbb{R}^d$ ($\{x : w^\top x + b \geq 0\}$) | $d+1$ |
| Axis-aligned rectangles in $\mathbb{R}^d$ | $2d$ |
| Polynomials of degree $\leq k$ in $\mathbb{R}^d$ | $\binom{d+k}{k}$ |
| Linear classifiers (neural net, one hidden layer, $p$ parameters) | $O(p \log p)$ |
| Finite set $|\mathcal{H}| = N$ | $\leq \log_2 N$ |

**Infinite VC dimension:** if no finite $d$ exists, $\mathcal{H}$ is not PAC learnable (e.g., all functions $\{0,1\}^\mathcal{X}$).

**VC bound:** with $m$ samples and $\mathcal{H}$ with $\text{VCdim}(\mathcal{H}) = d$, with probability $\geq 1-\delta$:

$$
L_\mathcal{D}(h) \leq L_S(h) + \sqrt{\frac{8d\ln(2m/d) + 8\ln(4/\delta)}{m}}
$$

## Applications

| Application | Role of VC dimension |
|---|---|
| Model selection | Higher-capacity models (large $d$) require more data |
| SVM generalisation | Support vector margin theory bounds generalisation via a normalised VC measure |
| Neural network theory | Networks with $p$ parameters have $\tilde{O}(p)$ VC dimension — but in practice generalise much better than this bound implies |
| Regularization theory | Constraining model complexity ↔ reducing effective VC dimension |

## Trade-offs

- VC dimension is a worst-case measure; it ignores the actual data distribution. Rademacher complexity and PAC-Bayes give tighter distribution-dependent bounds.
- Modern deep networks have enormous VC dimension yet generalise well — classical VC theory doesn't fully explain this; double-descent and implicit regularisation are active research areas.
- Computing VC dimension exactly is often NP-hard; it is generally used as a theoretical tool rather than a practical quantity.

## Links

- [[pac_learning|PAC Learning]] — PAC learnability ↔ finite VC dimension
- [[generalization_bounds|Generalization Bounds and Rademacher Complexity]] — tighter, data-dependent bounds
- [[bias_variance_analysis|Bias-Variance Analysis]] — practical analogue; VC dimension ↔ model complexity
- [[01_foundations/03_probability_and_statistics/probability_theory|Probability Theory]] — Sauer-Shelah lemma uses combinatorial probability
