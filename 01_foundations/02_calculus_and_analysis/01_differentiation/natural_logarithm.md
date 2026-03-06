---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Natural Logarithm

## Definition

The **natural logarithm** $\ln x$ is the inverse of the exponential function $\exp(x)$. It is the logarithm with base $e$:
$$
\ln x = \log_{e} x.
$$

**Domain:** positive real numbers $x>0$ (occasionally $\ln|x|$ is used to include negative reals). $\ln 0$ is undefined.

## Intuition

$\ln x$ measures how many times $e \approx 2.718$ must be raised to a power to reach $x$. Geometrically, $\ln x$ equals the area under the curve $1/t$ from $t=1$ to $t=x$. It converts multiplication into addition, which makes it the natural tool for reasoning about ratios and exponential growth rates.

## Formal Description

Standard logarithm rules:
$$
\ln(xy)=\ln x+\ln y,\quad \ln\!\left(\frac{x}{y}\right)=\ln x-\ln y,\quad \ln(x^{r})=r\ln x.
$$

Special values:
$$
\ln 1=0,\quad \ln e=1.
$$

**Power law via logarithm:** For all real $p$,
$$
x^{p}=\exp(p\ln x).
$$

**Mutual inverses:**
$$
e^{\ln x}=x\quad(x>0),\qquad \ln(e^{x})=x.
$$

## Applications

Used to solve exponential equations (e.g. $e^{rt}=2 \Rightarrow t=\frac{\ln 2}{r}$), to define entropy and information content ($-\ln p$), and to linearise power-law relationships in data analysis via log-log plots.

## Trade-offs

$\ln x$ is only defined for $x>0$; extending to negative reals or zero requires either $\ln|x|$ (losing injectivity) or complex logarithms (introducing multi-valuedness). Numerically, $\ln x$ is ill-conditioned near $x=0$.

## Links

- [[growth_decay_oscillation|Growth, Decay and Oscillation]] — exponential growth/decay
