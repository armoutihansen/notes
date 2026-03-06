---
layer: 01_foundations
type: concept
status: growing
tags: [theory]
created: 2026-03-06
---

# PAC Learning

## Definition

A concept class $\mathcal{C}$ is **probably approximately correct (PAC) learnable** if there exists an algorithm $A$ and a polynomial $p(\cdot, \cdot, \cdot, \cdot)$ such that for all $\epsilon, \delta \in (0, 1)$, every concept $c \in \mathcal{C}$, and every distribution $\mathcal{D}$ over the input space, if $A$ receives at least $m \geq p(1/\epsilon, 1/\delta, n, |\mathcal{C}|)$ i.i.d. samples from $\mathcal{D}$ labelled by $c$, then with probability at least $1 - \delta$, $A$ outputs a hypothesis $h$ with true error $\text{err}(h) \leq \epsilon$.

- **$\epsilon$:** accuracy parameter (maximum tolerated error)
- **$\delta$:** confidence parameter (allowed failure probability)
- **Probably:** succeeds with probability $\geq 1 - \delta$
- **Approximately correct:** error $\leq \epsilon$

## Intuition

PAC learning formalises "what does it mean to learn from data?" Instead of demanding a perfect classifier, it asks: can you, with enough data, produce a classifier that is almost certainly good enough? The framework cleanly separates the statistical question (how many samples?) from the computational question (how fast can you find the hypothesis?).

Think of it as a contract: the learner doesn't know the true concept $c$ or the data distribution $\mathcal{D}$, but given enough examples, it commits to outputting something close to $c$ with high confidence.

## Formal Description

**Setup:**
- Input space $\mathcal{X}$, label space $\mathcal{Y} = \{0,1\}$
- Concept class $\mathcal{C}$ (all learnable target functions)
- Hypothesis class $\mathcal{H}$ (all functions the learner can output; may differ from $\mathcal{C}$)
- True risk: $L_\mathcal{D}(h) = \Pr_{\mathbf{x}\sim\mathcal{D}}[h(\mathbf{x}) \neq c(\mathbf{x})]$
- Empirical risk: $L_S(h) = \frac{1}{m} \sum_{i=1}^m \mathbf{1}[h(\mathbf{x}_i) \neq y_i]$

**Realizable PAC learning:** assumes $\exists h^* \in \mathcal{H}$ with $L_\mathcal{D}(h^*) = 0$ (target in the class). Standard result: for finite $|\mathcal{H}|$, empirical risk minimization (ERM) satisfies the PAC bound with

$$m \geq \frac{1}{\epsilon}\left(\ln|\mathcal{H}| + \ln\frac{1}{\delta}\right)$$

**Agnostic PAC learning:** no assumption that $\mathcal{H}$ contains a perfect hypothesis. Goal: find $h \in \mathcal{H}$ satisfying

$$L_\mathcal{D}(h) \leq \min_{h' \in \mathcal{H}} L_\mathcal{D}(h') + \epsilon$$

Agnostic PAC is strictly harder; sample complexity depends on VC dimension (see [[vc_dimension]]).

**Consistent learner:** an algorithm that always returns $h \in \mathcal{H}$ with zero empirical error on the training set. For realizable PAC, any consistent learner is a valid PAC learner.

**Occam's razor / compression:** simpler hypotheses (shorter description length $\log|\mathcal{H}|$) require fewer samples. This is the formal justification for preferring simple models.

**Computational vs statistical learnability:** PAC is a statistical framework; even if a class is statistically PAC learnable, finding the optimal $h$ may be NP-hard (e.g., learning intersections of halfspaces).

## Applications

| Concept | Connection to PAC |
|---|---|
| Sample complexity bounds | Lower bound on $m$ to achieve $(\epsilon,\delta)$-PAC |
| Regularization | Constraining $\mathcal{H}$ (smaller size) reduces required $m$ |
| Model selection | Simpler models ↔ smaller $|\mathcal{H}|$ ↔ tighter PAC bounds |
| Cross-validation | Empirical substitute for estimating true risk |

## Trade-offs

- PAC bounds are often loose (vacuous for realistic $m$ and $n$); non-uniform bounds (Rademacher complexity) are tighter in practice.
- The framework assumes i.i.d. data; distribution shift invalidates PAC guarantees.
- PAC learning ignores computational cost; a class can be PAC learnable but computationally hard to learn.

## Links

- [[vc_dimension|VC Dimension]] — infinite hypothesis class version of PAC sample complexity
- [[generalization_bounds|Generalization Bounds and Rademacher Complexity]] — tighter, data-dependent bounds
- [[bias_variance_analysis|Bias-Variance Analysis]] — practical interpretation of approximation-estimation tradeoff
- [[01_foundations/03_probability_and_statistics/bayesian_inference|Bayesian Inference]] — alternative framework for quantifying learning uncertainty
