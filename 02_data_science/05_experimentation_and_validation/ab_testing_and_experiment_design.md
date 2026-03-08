---
layer: 02_data_science
type: workflow
status: growing
tags: [evaluation, workflow]
created: 2026-03-07
---

# A/B Testing and Experiment Design

## Problem Context

Online controlled experiments (A/B tests) are the gold standard for measuring the causal effect of a product or model change. The goal is to isolate a single intervention and measure its effect on a metric of business interest, with enough statistical confidence to make a decision. Errors in design (under-powered tests, mis-specified metrics) or analysis (peeking, multiple comparisons) silently corrupt that causal inference.

Typical contexts: ranking model changes, pricing algorithms, UI treatments, recommendation system variants, risk model thresholds.

---

## Core Concepts

### Null and Alternative Hypotheses

- **H₀ (null):** the treatment has no effect; observed differences are due to chance.
- **H₁ (alternative):** the treatment has a nonzero (or directional) effect.

The test never "accepts H₀" — it either rejects it (p < α) or fails to reject it.

### Type I and Type II Errors

| | H₀ true | H₀ false |
|---|---|---|
| **Reject H₀** | Type I error (false positive, rate = α) | Correct (power = 1 − β) |
| **Fail to reject H₀** | Correct | Type II error (false negative, rate = β) |

Standard settings: α = 0.05, β = 0.20 (power = 80 %). Reducing both requires a larger sample.

### Statistical Significance vs Practical Significance

A result can be statistically significant (p < 0.05) but practically meaningless — a 0.001 % lift in conversion is detectable with millions of users but not worth engineering effort. Always report an **effect size** (Cohen's d, relative lift, MDE) alongside the p-value.

**Minimum Detectable Effect (MDE):** the smallest true effect for which the test has the pre-specified power. Set the MDE based on the *business threshold* for action, not on what happens to be detectable with available traffic.

### p-values

The p-value is the probability of observing a test statistic at least as extreme as the one computed, **assuming H₀ is true**. It is not the probability that H₀ is true, and it is not the probability that the result is a false positive. Misreading p-values is a primary source of incorrect decisions.

### A/B vs Multivariate Testing

| Method | When to use | Trade-off |
|---|---|---|
| A/B (2 arms) | Single change, clear hypothesis | Simple; loses signal about interactions |
| A/B/n (multi-arm) | Several variants of the same change | Requires corrections for multiple comparisons |
| Multivariate (MVT) | Testing combinations of features simultaneously | Requires factorial traffic splits; hard to interpret interactions |
| Bandit (MAB) | When exploration-exploitation matters in production | Adapts allocation but invalidates classical inference |

---

## Experiment Design Checklist

1. **Define the primary metric** before running. A single north-star metric with a pre-specified MDE and direction (one-tailed vs two-tailed).
2. **Define guardrail metrics** (e.g., latency, error rate, revenue) that must not regress; failure on a guardrail is a veto.
3. **Randomization unit:** user-level (long-term effects), session-level (short-term, risk of cross-contamination), request-level (high variance).
4. **Power calculation:** determine required sample size before starting (see §Power Analysis).
5. **Pre-experiment AA test:** run an A/A test with the same randomization logic. A statistically significant difference in an A/A test signals instrumentation or assignment bugs.
6. **Holdout period and pre-period:** measure baseline metrics for the same period in the prior cycle to detect instrumentation drift.
7. **Novelty effect window:** new UI treatments often show a short-lived engagement spike. Plan a run duration that extends beyond it (typically ≥ 2 weeks for consumer products).
8. **Unit of diversion matches unit of analysis:** if users are the unit of diversion, the analysis must aggregate to users, not to events (variance inflation if mis-matched).
9. **Ship criteria written before launch:** what p-value threshold and absolute lift are required to ship? Decide in advance.

---

## Statistical Framework

### Two-sample z-test for proportions (conversion metrics)

Let $p_c$, $p_t$ be control and treatment conversion rates, $n$ total sample size (equal split assumed).

$$
z = \frac{\hat{p}_t - \hat{p}_c}{\sqrt{\hat{p}(1-\hat{p})\left(\frac{2}{n/2}\right)}}
$$

where $\hat{p} = (\hat{p}_c + \hat{p}_t)/2$ is the pooled estimate.

Reject H₀ when $|z| > z_{\alpha/2}$ (two-tailed) or $z > z_\alpha$ (one-tailed).

```python
from scipy.stats import proportions_ztest
import numpy as np

conversions = np.array([control_conversions, treatment_conversions])
nobs        = np.array([n_control, n_treatment])

stat, p_value = proportions_ztest(conversions, nobs, alternative='two-sided')
print(f"z = {stat:.3f}, p = {p_value:.4f}")
```

### Two-sample t-test for continuous metrics (revenue, engagement time)

```python
from scipy.stats import ttest_ind

stat, p_value = ttest_ind(control_values, treatment_values, equal_var=False)  # Welch's t-test
```

### Multiple comparisons correction

Testing $k$ hypotheses simultaneously inflates the family-wise error rate. Use:

- **Bonferroni:** $\alpha' = \alpha / k$ — conservative, controls FWER.
- **Benjamini-Hochberg:** controls False Discovery Rate (FDR) at level $q$; less conservative, preferred when many metrics are tested.

```python
from statsmodels.stats.multitest import multipletests

reject, pvals_corrected, _, _ = multipletests(p_values, alpha=0.05, method='fdr_bh')
```

---

## Power Analysis

### Sample size for a two-proportion test

$$
n = \frac{(z_{\alpha/2} + z_\beta)^2 \cdot 2p(1-p)}{\delta^2}
$$

where:
- $p$ = baseline conversion rate
- $\delta$ = MDE (absolute difference, e.g., 0.01 for a 1 pp lift)
- $z_{\alpha/2}$ = 1.96 (α = 0.05, two-tailed)
- $z_\beta$ = 0.84 (power = 80 %) or 1.28 (power = 90 %)

```python
from statsmodels.stats.power import NormalIndPower, zt_ind_solve_power

# For proportions
n_per_group = zt_ind_solve_power(
    effect_size=mde / np.sqrt(p_baseline * (1 - p_baseline)),
    alpha=0.05,
    power=0.80,
    alternative='two-sided',
)
print(f"Required per group: {n_per_group:.0f}")
```

For continuous metrics, use Cohen's d:

```python
from statsmodels.stats.power import TTestIndPower

analysis = TTestIndPower()
n = analysis.solve_power(
    effect_size=cohens_d,   # (mu_t - mu_c) / pooled_std
    alpha=0.05,
    power=0.80,
    alternative='two-sided',
)
```

### Effect size calibration

Set the MDE from historical data and business thresholds:
- Compute the metric's standard deviation from the past 30 days of control traffic.
- Set $\delta$ to the smallest lift that would change the product decision (not the smallest detectable lift).
- Higher baseline variance → larger required n → longer runtime or reduced power.

### Runtime estimation

$$
\text{days} = \frac{2n}{\text{daily traffic per arm}}
$$

Accounts for both arms. Traffic allocation other than 50/50 increases required total n.

---

## Sequential Testing

Classical tests assume a fixed sample size decided before peeking. Repeated peeking and stopping early on significance inflates the actual Type I error rate far above α.

**Alpha spending (Lan-DeMets):** pre-allocate the Type I error budget across pre-planned looks. At look $k$ out of $K$:
$$
\alpha(t_k) = 2\left(1 - \Phi\left(\frac{z_{\alpha/2}}{\sqrt{t_k}}\right)\right)
$$
where $t_k = n_k/N$ is the information fraction.

**Sequential Probability Ratio Test (SPRT):** continuously computes a likelihood ratio; stops when it crosses an upper bound (reject H₀) or lower bound (accept H₀). Fully sequential, no pre-planned maximum n, but requires specifying H₁ explicitly.

**Always-Valid Inference (mSPRT):** confidence sequences that are valid at any stopping time. Used in platforms like Statsig and Optimizely Stats Engine; allows continuous monitoring without inflation.

```python
# Simple alpha-spending illustration (O'Brien-Fleming boundary)
from scipy.stats import norm

def obrien_fleming_boundary(t, alpha=0.05):
    """Critical value at information fraction t ∈ (0,1]."""
    return norm.ppf(1 - alpha / 2) / np.sqrt(t)
```

---

## Common Pitfalls

### P-hacking / data dredging
Stopping when p < 0.05 is reached, testing multiple metrics without correction, or running multiple segments and reporting only the significant one — all inflate the Type I error rate dramatically. Enforce pre-registration: metric, MDE, α, and runtime must be documented before the experiment starts.

### Novelty and primacy effects
Users behave differently when they encounter a change for the first time (novelty) or resist changing habitual behaviour (primacy). Run experiments long enough to observe steady-state behaviour — typically ≥ 2 full weekly cycles for consumer products.

### SUTVA violations (Stable Unit Treatment Value Assumption)
Assumes the treatment of one unit does not affect outcomes for other units. Violated by:
- **Network effects:** social platforms where treated users communicate with control users.
- **Marketplace interference:** pricing experiments that shift demand and affect control-arm prices.
- **Shared resources:** two experiment arms competing for the same cache or recommendation pool.

Mitigations: cluster randomization (randomize at the group/market level), holdout clusters, ego-network disjoint splits.

### Survivorship and selection bias
Analysing only users who completed a funnel step post-randomization (e.g., only users who made a purchase) excludes users who were deterred by the treatment — the excluded population is non-random across arms.

### Cookie churn / re-randomization
If the randomization ID (cookie) is volatile, users may switch arms mid-experiment, contaminating both arms. Use stable identifiers (user ID) wherever possible.

### Metric mis-specification
Using a proxy metric that is easy to move but does not causally predict long-term business value. Validate proxy metrics against long-term outcomes periodically.

### Underpowered tests
Declaring a result "not significant" after an under-powered test does not mean the treatment has no effect. Always report the MDE and achieved power alongside a null result.

---

## References

- Kohavi, R., Tang, D., & Xu, Y. (2020). *Trustworthy Online Controlled Experiments: A Practical Guide to A/B Testing*. Cambridge University Press. [Book page](https://www.cambridge.org/core/books/trustworthy-online-controlled-experiments/D97B26382EB0EB2DC2019A7A7B518F59)
- Kohavi, R. et al. (2009). Controlled experiments on the web: Survey and practical guide. *Data Mining and Knowledge Discovery*, 18, 140–181. [DOI](https://doi.org/10.1007/s10618-008-0114-1)
- Johari, R. et al. (2015). Always valid inference: Bringing sequential analysis to A/B testing. [arXiv:1512.04922](https://arxiv.org/abs/1512.04922)
- Deng, A. et al. (2013). Improving the sensitivity of online controlled experiments by utilizing pre-experiment data. *WSDM 2013*. [ACM](https://dl.acm.org/doi/10.1145/2433396.2433413)
- [statsmodels power analysis docs](https://www.statsmodels.org/stable/stats.html#power-and-sample-size-calculations)

---

## Links

- [[02_data_science/05_experimentation_and_validation/index|05 — Experimentation and Validation]]
- [[01_foundations/03_probability_and_statistics/index|Probability and Statistics]]
- [[03_modeling/07_evaluation_and_model_selection/index|Evaluation and Model Selection]]
- [[02_data_science/05_experimentation_and_validation/data_validation|Data Validation]] — complementary data quality gates
- [[01_foundations/03_probability_and_statistics/hypothesis_testing|Hypothesis Testing]] — formal test theory underlying A/B tests
