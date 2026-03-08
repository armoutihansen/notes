---
layer: 03_modeling
type: concept
status: seed
tags: [algorithm, forecasting]
created: 2026-05-31
---

# State-Space Models

## Core Idea

State-space models represent a time series as the noisy observation of a hidden state that evolves according to a Markov process. This separates the noise in observations from the underlying dynamics, enabling principled uncertainty quantification, interpolation of missing values, and smooth trend extraction.

The canonical form is the **Linear Gaussian State-Space Model** (LGSSM), solvable exactly via the Kalman filter.

## Mathematical Formulation

### Linear Gaussian State-Space Model

**State transition:**
$$
\mathbf{z}_t = F \mathbf{z}_{t-1} + \mathbf{q}_t, \qquad \mathbf{q}_t \sim \mathcal{N}(\mathbf{0}, Q)
$$

**Observation model:**
$$
\mathbf{x}_t = H \mathbf{z}_t + \mathbf{r}_t, \qquad \mathbf{r}_t \sim \mathcal{N}(\mathbf{0}, R)
$$

where:
- $\mathbf{z}_t \in \mathbb{R}^m$ is the hidden state
- $\mathbf{x}_t \in \mathbb{R}^d$ is the observed measurement
- $F$ is the transition matrix, $H$ is the observation matrix
- $Q$ is process noise covariance, $R$ is observation noise covariance

### Kalman Filter (Forward Pass)

The Kalman filter computes the posterior $p(\mathbf{z}_t \mid \mathbf{x}_{1:t})$ recursively.

**Predict step:**
$$
\mathbf{z}_{t|t-1} = F \mathbf{z}_{t-1|t-1}
$$
$$
P_{t|t-1} = F P_{t-1|t-1} F^\top + Q
$$

**Update step (incorporate observation $\mathbf{x}_t$):**
$$
K_t = P_{t|t-1} H^\top (H P_{t|t-1} H^\top + R)^{-1} \qquad \text{(Kalman gain)}
$$
$$
\mathbf{z}_{t|t} = \mathbf{z}_{t|t-1} + K_t (\mathbf{x}_t - H \mathbf{z}_{t|t-1})
$$
$$
P_{t|t} = (I - K_t H) P_{t|t-1}
$$

The Kalman filter is the optimal linear estimator under Gaussian noise.

### Kalman Smoother (Backward Pass)

Computes the full posterior $p(\mathbf{z}_t \mid \mathbf{x}_{1:T})$ using the **RTS smoother** (Rauch-Tung-Striebel), running a backward pass after the Kalman filter.

### Structural Time Series Models

STMs decompose a time series into interpretable components using the SSM framework:

$$
y_t = \mu_t + \gamma_t + \varepsilon_t
$$

- $\mu_t$: **trend** (local level or local linear trend)
- $\gamma_t$: **seasonality** (trigonometric or dummy encoding)
- $\varepsilon_t \sim \mathcal{N}(0, \sigma^2)$: observation noise

Each component is modelled as a state with its own transition and noise equations. Parameters are estimated by maximum likelihood via the Kalman filter's prediction error decomposition:

$$
\log L = -\frac{T}{2}\log 2\pi - \frac{1}{2}\sum_{t=1}^T \left(\log |S_t| + v_t^\top S_t^{-1} v_t\right)
$$

where $v_t = x_t - H\mathbf{z}_{t|t-1}$ are the innovations.

## Inductive Bias

- Linear dynamics + Gaussian noise → exact inference via Kalman filter.
- Structural decomposition → interpretable trend and seasonal components.
- Markov property: the future depends on the present state only, not the full history.

## Training Objective

Maximise the log-likelihood of observations under the SSM using the Kalman filter's prediction error decomposition. In Python, use `statsmodels.tsa.statespace` or `PyKalman`.

```python
from statsmodels.tsa.statespace.structural import UnobservedComponents

model = UnobservedComponents(
    y_train,
    level='local linear trend',
    seasonal=12,
    stochastic_seasonal=True
)
result = model.fit(disp=False)
forecast = result.forecast(steps=12)
```

## Strengths

- Handles missing values naturally (just skip the update step).
- Provides calibrated uncertainty intervals.
- Interpretable components (trend, seasonality).
- Real-time filtering: online updates as new data arrives.

## Weaknesses

- Assumes linear dynamics and Gaussian noise; fails for highly non-linear or heavy-tailed systems.
- Parameter estimation (Q, R matrices) can be poorly identified with short series.
- Extended Kalman Filter and Unscented Kalman Filter handle non-linearities but lose optimality guarantees.

## Variants

- **Extended Kalman Filter (EKF)**: linearises non-linear dynamics; approximate.
- **Unscented Kalman Filter (UKF)**: uses sigma points instead of linearisation; better for moderate non-linearities.
- **Particle filter**: fully non-parametric; handles arbitrary non-linear, non-Gaussian systems; $O(n_\text{particles})$.
- **Dynamic Linear Models (DLMs)**: flexible SSM framework; supports time-varying $F_t$, $H_t$, $Q_t$, $R_t$.

## References

- Harvey, A.C. (1990). *Forecasting, Structural Time Series Models and the Kalman Filter*. Cambridge University Press.
- Durbin, J. & Koopman, S.J. (2012). *Time Series Analysis by State Space Methods* (2nd ed.). Oxford University Press.
- West, M. & Harrison, J. (1997). *Bayesian Forecasting and Dynamic Models*. Springer.

## Links

- [[03_modeling/05_time_series/01_classical_forecasting/time_series_models|Time Series Models — ARIMA and classical approach]]
- [[03_modeling/03_probabilistic_models/02_latent_variable_models/latent_variable_models|Latent Variable Models — HMM as discrete-state SSM]]
- [[01_foundations/03_probability_and_statistics/probability_distributions|Probability Distributions — Gaussian conditionals]]
- [[01_foundations/04_optimization/index|Optimization — MLE parameter estimation]]
