---
layer: 01_foundations
type: concept
status: growing
tags: [theory, evaluation]
created: 2026-03-06
---

# Statistical Inference and Hypothesis Testing

## Definition

Statistical inference draws conclusions about population parameters from sample data. Hypothesis testing is a formal decision procedure that evaluates whether observed data is consistent with a null hypothesis $H_0$.

## Intuition

Hypothesis testing asks: "if the null hypothesis were true, how surprising is the data I observed?" The p-value answers this: a small p-value means the data would be very rare under $H_0$, providing evidence against it. A confidence interval gives a plausible range of values for the parameter, consistent with the data at a given confidence level.

## Formal Description

### Estimators and Properties

An **estimator** $\hat{\theta}(X_1,\ldots,X_n)$ is a function of the sample.

- **Bias:** $\text{Bias}(\hat{\theta}) = \mathbb{E}[\hat{\theta}] - \theta$
- **Variance:** $\operatorname{Var}(\hat{\theta})$
- **MSE:** $\text{MSE}(\hat{\theta}) = \text{Bias}^2 + \operatorname{Var}$

The **sample mean** $\bar{X} = \frac{1}{n}\sum_i X_i$ is an unbiased estimator of $\mu$ with variance $\sigma^2/n$.

### Hypothesis Testing Framework

1. State $H_0$ (null) and $H_1$ (alternative).
2. Choose a test statistic $T(X_1,\ldots,X_n)$ whose distribution under $H_0$ is known.
3. Compute the observed test statistic $t_{\text{obs}}$.
4. Compute the **p-value**: $P(|T| \geq |t_{\text{obs}}| \mid H_0)$ (two-sided).
5. Reject $H_0$ if $p < \alpha$ (significance level, typically 0.05).

**Error types:**

| | $H_0$ true | $H_1$ true |
|---|---|---|
| Reject $H_0$ | Type I error (FP), prob. $\alpha$ | Correct (power $1-\beta$) |
| Fail to reject | Correct | Type II error (FN), prob. $\beta$ |

**Power** $= 1 - \beta$: the probability of correctly rejecting a false $H_0$.

### Common Tests

**One-sample $t$-test** — test whether $\mu = \mu_0$ when $\sigma^2$ is unknown:

$$
t = \frac{\bar{X} - \mu_0}{s/\sqrt{n}} \sim t_{n-1} \text{ under } H_0
$$

**Two-sample $t$-test (Welch)** — compare means of two independent groups:

$$
t = \frac{\bar{X}_1 - \bar{X}_2}{\sqrt{s_1^2/n_1 + s_2^2/n_2}}
$$

**Paired $t$-test** — for paired observations, test differences $D_i = X_{1i} - X_{2i}$ as a one-sample test.

**Chi-squared test of independence** — test whether two categorical variables are independent:

$$
\chi^2 = \sum_{i,j} \frac{(O_{ij} - E_{ij})^2}{E_{ij}} \sim \chi^2_{(r-1)(c-1)} \text{ under } H_0
$$

where $E_{ij} = (\text{row total}_i)(\text{col total}_j)/n$.

**Z-test for proportions** — comparing observed proportion to $p_0$ with large $n$.

### Confidence Intervals

A $95\%$ CI for $\mu$ with unknown $\sigma$:

$$
\bar{X} \pm t_{n-1,\,0.025} \cdot \frac{s}{\sqrt{n}}
$$

**Interpretation:** if the procedure were repeated many times, 95% of such intervals would contain the true $\mu$. It does **not** mean there is a 95% probability that $\mu$ lies in any one specific interval (a common misconception).

### Multiple Testing

Running $m$ tests at $\alpha = 0.05$ yields $\approx 0.05m$ false positives. Corrections:

- **Bonferroni:** use $\alpha/m$ per test; controls family-wise error rate (FWER), very conservative.
- **Benjamini-Hochberg:** controls false discovery rate (FDR) at level $\alpha$; more powerful than Bonferroni when $m$ is large.

## Applications

- A/B testing in product and ML model comparisons
- Feature selection (test each feature's association with target)
- Clinical trials, regulatory submissions
- Checking model residuals for normality or heteroscedasticity

## Trade-offs

- **Statistical vs practical significance:** a large enough sample makes tiny effects significant; always report effect sizes alongside p-values.
- **p-values do not give the probability that $H_0$ is true** — only the probability of the data given $H_0$.
- For online experimentation at scale, consider sequential testing (e.g., mSPRT) to avoid inflated error rates from early stopping.

## Links

- [[probability_theory|Probability Theory]]
- [[probability_distributions|Probability Distributions]]
- [[bayesian_inference|Bayesian Inference]]
- [[01_foundations/05_statistical_learning_theory/evaluation_metrics|Evaluation Metrics]]
- [[01_foundations/05_statistical_learning_theory/data_splits_and_distribution|Data Splits and Distribution]]
