---
layer: 01_foundations
type: concept
status: growing
tags: [probability, sample-space, conditional-probability, independence, bayes-theorem]
created: 2026-03-06
---

# Probability Theory

## Definition

A mathematical framework for quantifying uncertainty. Assigns a number in $[0,1]$ to events drawn from a sample space, obeying Kolmogorov's axioms.

## Intuition

Probability formalises the intuition that some outcomes are more likely than others. The axioms ensure probabilities are consistent: they don't go negative, they add up to 1 across all possibilities, and disjoint events combine additively.

## Formal Description

**Sample space and events**

- **Sample space** $\Omega$: the set of all possible outcomes.
- **Event** $A \subseteq \Omega$: a subset of outcomes.
- **Probability measure** $P: \mathcal{F} \to [0,1]$ on a $\sigma$-algebra $\mathcal{F}$ satisfying:
  1. $P(A) \geq 0$ for all $A$
  2. $P(\Omega) = 1$
  3. Countable additivity: $P\!\left(\bigcup_{i=1}^\infty A_i\right) = \sum_{i=1}^\infty P(A_i)$ for pairwise disjoint $A_i$

**Derived rules**

$$P(A^c) = 1 - P(A)$$

$$P(A \cup B) = P(A) + P(B) - P(A \cap B)$$

**Conditional probability**

$$P(A \mid B) = \frac{P(A \cap B)}{P(B)}, \qquad P(B) > 0$$

**Chain rule** (product rule):

$$P(A \cap B) = P(A \mid B)\,P(B) = P(B \mid A)\,P(A)$$

For $n$ events: $P(A_1 \cap \cdots \cap A_n) = P(A_1)\,P(A_2 \mid A_1)\cdots P(A_n \mid A_1,\ldots,A_{n-1})$

**Total probability**

Given a partition $\{B_i\}$ of $\Omega$:

$$P(A) = \sum_i P(A \mid B_i)\,P(B_i)$$

**Bayes' theorem**

$$P(B_i \mid A) = \frac{P(A \mid B_i)\,P(B_i)}{\sum_j P(A \mid B_j)\,P(B_j)}$$

This is the engine of Bayesian inference: $P(B_i \mid A)$ is the **posterior**, $P(B_i)$ is the **prior**, $P(A \mid B_i)$ is the **likelihood**.

**Independence**

$A$ and $B$ are **independent** iff $P(A \cap B) = P(A)\,P(B)$, equivalently $P(A \mid B) = P(A)$.

$A$ and $B$ are **conditionally independent** given $C$ iff $P(A \cap B \mid C) = P(A \mid C)\,P(B \mid C)$.

**Random variables**

A random variable $X: \Omega \to \mathbb{R}$ maps outcomes to real numbers.

- **CDF:** $F_X(x) = P(X \leq x)$
- **PMF (discrete):** $p_X(x) = P(X = x)$
- **PDF (continuous):** $f_X(x)$ such that $P(X \in [a,b]) = \int_a^b f_X(x)\,dx$

**Expected value and variance**

$$\mathbb{E}[X] = \sum_x x\,p(x) \quad \text{or} \quad \int x\,f(x)\,dx$$

$$\operatorname{Var}(X) = \mathbb{E}[(X - \mathbb{E}[X])^2] = \mathbb{E}[X^2] - (\mathbb{E}[X])^2$$

**Linearity of expectation:** $\mathbb{E}[aX + bY] = a\,\mathbb{E}[X] + b\,\mathbb{E}[Y]$ (no independence required).

**Covariance:** $\operatorname{Cov}(X,Y) = \mathbb{E}[(X-\mu_X)(Y-\mu_Y)]$.

Correlation: $\rho(X,Y) = \operatorname{Cov}(X,Y) / (\sigma_X \sigma_Y) \in [-1, 1]$.

## Applications

- Bayesian inference: updating beliefs given new data
- Machine learning: every probabilistic model (naive Bayes, GMMs, neural nets with cross-entropy) is built on these foundations
- Statistical hypothesis testing (p-values rely on conditional probability)
- Markov chains, hidden Markov models, graphical models

## Trade-offs

- The frequentist interpretation ($P$ = long-run frequency) and the Bayesian interpretation ($P$ = degree of belief) are philosophically distinct but mathematically identical at the level of Kolmogorov's axioms.
- Conditional probability is undefined when $P(B) = 0$; care is needed at measure-zero events.

## Links

- [[probability_distributions|Probability Distributions]]
- [[bayesian_inference|Bayesian Inference]]
- [[hypothesis_testing|Hypothesis Testing]]
- [[01_foundations/05_statistical_learning_theory/bias_variance_analysis|Bias–Variance Analysis]]
- [[02_modeling/03_model_families/03_probabilistic_models/index|Probabilistic Models]]
