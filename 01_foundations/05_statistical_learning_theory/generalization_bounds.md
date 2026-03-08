---
layer: 01_foundations
type: concept
status: growing
tags: [theory]
created: 2026-03-06
---

# Generalization Bounds and Rademacher Complexity

## Definition

**Generalization bound:** a probabilistic upper bound on the true risk $L_\mathcal{D}(h)$ of a learned hypothesis $h$, given only the empirical risk $L_S(h)$ on training data.

**Rademacher complexity** of a hypothesis class $\mathcal{H}$ with respect to a sample $S = (x_1,\ldots,x_m)$:

$$
\hat{\mathcal{R}}_S(\mathcal{H}) = \mathbb{E}_{\boldsymbol{\sigma}}\left[\sup_{h \in \mathcal{H}} \frac{1}{m}\sum_{i=1}^m \sigma_i h(x_i)\right]
$$

where $\sigma_i \in \{-1, +1\}$ are i.i.d. Rademacher (uniform ±1) random variables.

$\mathcal{R}_m(\mathcal{H}) = \mathbb{E}_S[\hat{\mathcal{R}}_S(\mathcal{H})]$ is the expected Rademacher complexity.

## Intuition

The generalization gap ($L_\mathcal{D}(h) - L_S(h)$) is bounded by how well the hypothesis class can "fit random noise". Rademacher complexity measures exactly this: how much correlation can the best $h \in \mathcal{H}$ have with a random $\pm 1$ labelling? If the class is so expressive that it can fit any labelling, the Rademacher complexity is 1 — the class is memorising noise and will not generalise.

This is a data-dependent, distribution-sensitive bound — tighter than worst-case VC bounds.

## Formal Description

**Empirical risk minimization (ERM) bound:** for ERM on a finite class $|\mathcal{H}| = N$ with $m$ samples, with probability $\geq 1-\delta$:

$$
L_\mathcal{D}(h_\text{ERM}) \leq L_S(h_\text{ERM}) + \sqrt{\frac{\ln N + \ln(2/\delta)}{2m}}
$$

**Rademacher generalisation bound:** for any $h \in \mathcal{H}$, with probability $\geq 1-\delta$:

$$
L_\mathcal{D}(h) \leq L_S(h) + 2\mathcal{R}_m(\mathcal{H}) + \sqrt{\frac{\ln(1/\delta)}{2m}}
$$

**Uniform convergence:** $\mathcal{H}$ satisfies uniform convergence if for all $\epsilon, \delta > 0$:

$$
\Pr_S\left[\sup_{h \in \mathcal{H}} |L_\mathcal{D}(h) - L_S(h)| > \epsilon\right] < \delta
$$

when $m \geq m_\mathcal{H}(\epsilon, \delta)$. This is the key condition for ERM to work.

**McDiarmid's inequality (stability):** if replacing any single sample changes the empirical risk by at most $c$, then by concentration:

$$
\Pr[L_S(h) - \mathbb{E}[L_S(h)] > t] \leq e^{-2t^2/(mc^2)}
$$

**Rademacher complexity of linear classes:** for $\mathcal{H} = \{\mathbf{x} \mapsto \mathbf{w}^\top\mathbf{x} : \|\mathbf{w}\|_2 \leq B\}$ with $\|\mathbf{x}_i\|_2 \leq C$:

$$
\mathcal{R}_m(\mathcal{H}) \leq \frac{BC}{\sqrt{m}}
$$

This shows that larger weight norms or inputs inflate the complexity, while more data shrinks it.

**Structural risk minimization (SRM):** instead of fixing $\mathcal{H}$, consider a hierarchy $\mathcal{H}_1 \subseteq \mathcal{H}_2 \subseteq \ldots$. For each level $k$, add a complexity penalty $\text{pen}(k)$ proportional to $\sqrt{d_k/m}$. Minimise the penalised objective:

$$
\hat{k} = \arg\min_k \left[L_{S}(h_k) + \text{pen}(k)\right]
$$

This formalises the bias-variance tradeoff: too simple (high bias, low complexity) vs too complex (low bias, high variance).

**PAC-Bayes bounds:** for a posterior $Q$ over $\mathcal{H}$ and prior $P$, with probability $\geq 1-\delta$:

$$
\mathbb{E}_{h \sim Q}[L_\mathcal{D}(h)] \leq \mathbb{E}_{h \sim Q}[L_S(h)] + \sqrt{\frac{\text{KL}(Q \| P) + \ln(m/\delta)}{2(m-1)}}
$$

PAC-Bayes bounds are often the tightest available for deep networks (when $P$ is chosen carefully).

## Applications

| Concept | How bounds apply |
|---|---|
| Regularization ($L_2$, $L_1$) | Constrains weight norms → smaller Rademacher complexity |
| Dropout | Effectively constrains hypothesis class capacity at training time |
| Early stopping | Limits effective complexity of gradient descent trajectory |
| Cross-validation | Empirical proxy for true risk; formal bound via $m$-splits |
| Deep network theory | PAC-Bayes and margin bounds partially explain generalisation |

## Trade-offs

- Classical VC and Rademacher bounds are often vacuous for deep networks (the bound > 1); they serve as theoretical insight rather than practical guidance.
- Data-dependent Rademacher bounds are tighter than VC bounds but still require knowing $\mathcal{H}$ exactly.
- PAC-Bayes bounds require choosing a prior $P$; with a good prior (matching inductive bias), they can be quite tight.
- The i.i.d. assumption is fundamental; distribution shift invalidates all standard generalisation bounds.

## Links

- [[pac_learning|PAC Learning]] — sample complexity is derived from generalisation bounds
- [[vc_dimension|VC Dimension]] — VC bound is a special case; Rademacher is tighter
- [[bias_variance_analysis|Bias-Variance Analysis]] — practical interpretation of approximation + estimation error
- [[01_foundations/03_probability_and_statistics/probability_theory|Probability Theory]] — McDiarmid, Hoeffding inequalities underpin the proofs
- [[01_foundations/04_optimization/convex_optimization|Convex Optimization]] — SRM requires solving a regularised objective
