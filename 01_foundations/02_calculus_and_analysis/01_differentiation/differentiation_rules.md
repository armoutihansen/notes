---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Differentiation Rules

## Definition

The core algebraic rules for computing derivatives: the power rule, linearity (sum and constant-multiple), the product rule, and the quotient rule.

## Intuition

Each rule reduces differentiation of a compound expression to differentiation of its simpler parts, making it possible to differentiate any combination of elementary functions by breaking it down step by step.

## Formal Description

### Power Rule

For a positive integer $n$, applying the binomial expansion to the limit definition yields:
$$
\frac{d}{dx}x^{n} = \lim_{h \to 0} \frac{(x+h)^{n} - x^{n}}{h} = \lim_{h \to 0} \frac{x^{n} + nhx^{n-1} + \cdots - x^{n}}{h} = nx^{n-1}.
$$

**Extension to all real exponents:** For any real $p$ and $x > 0$, write $x^{p} = \exp(p \ln x)$ and apply the chain rule:
$$
\frac{d}{dx}x^{p} = \exp(p \ln x) \cdot \frac{p}{x} = x^{p} \cdot \frac{p}{x} = px^{p-1}.
$$

$$
\boxed{\frac{d}{dx}x^{n} = nx^{n-1}}
$$

**Example — square root:**
$$
\frac{d}{dx}\sqrt{x} = \lim_{h \to 0} \frac{\sqrt{x+h} - \sqrt{x}}{h} = \lim_{h \to 0} \frac{1}{\sqrt{x+h} + \sqrt{x}} = \frac{1}{2\sqrt{x}} = \frac{1}{2}x^{-1/2}.
$$

### Sum and Constant-Multiple Rules

Differentiation is linear: the derivative of a sum (or difference) equals the sum (or difference) of the derivatives, and the derivative of a constant multiple equals that constant times the derivative.

**Sum rule:** For differentiable functions $f(x)$ and $g(x)$,
$$
\bigl(f(x) + g(x)\bigr)' = \lim_{h \to 0} \frac{[f(x+h)+g(x+h)] - [f(x)+g(x)]}{h} = f'(x) + g'(x).
$$

**Constant-multiple rule:**
$$
\bigl(cf(x)\bigr)' = \lim_{h \to 0} \frac{cf(x+h) - cf(x)}{h} = c \lim_{h \to 0} \frac{f(x+h)-f(x)}{h} = cf'(x).
$$

$$
\bigl(f \pm g\bigr)' = f' \pm g', \qquad \bigl(cf\bigr)' = cf'.
$$

The derivative of any constant function is zero.

### Product Rule

The product rule gives the derivative of a product of two differentiable functions. Starting from the limit definition, adding and subtracting $f(x)g(x+h)$ in the numerator:
$$
\begin{align}
\bigl[f(x)g(x)\bigr]' &= \lim_{h \to 0} \frac{f(x+h)g(x+h) - f(x)g(x)}{h} \\
&= \lim_{h \to 0} \frac{[f(x+h)-f(x)]\,g(x+h) + f(x)[g(x+h)-g(x)]}{h} \\
&= f'(x)g(x) + f(x)g'(x).
\end{align}
$$

$$
\bigl[f(x)g(x)\bigr]' = f'(x)g(x) + f(x)g'(x).
$$

### Quotient Rule

The quotient rule gives the derivative of the ratio of two differentiable functions. Starting from the limit definition and adding and subtracting $f(x)g(x)$ in the numerator:
$$
\begin{align}
\left(\frac{f(x)}{g(x)}\right)' &= \lim_{h \to 0} \frac{f(x+h)g(x) - f(x)g(x+h)}{h\,g(x+h)g(x)} \\
&= \frac{1}{g(x)^{2}}\left[f'(x)g(x) - f(x)g'(x)\right].
\end{align}
$$

$$
\left(\frac{f(x)}{g(x)}\right)' = \frac{f'(x)g(x) - f(x)g'(x)}{g(x)^{2}}.
$$

## Applications

- Computing derivatives of polynomials, rational functions, and algebraic expressions.
- Building blocks for differentiating more complex functions together with the chain rule.
- The quotient rule underlies the derivatives of $\tan x$, $\cot x$, $\sec x$, $\csc x$.

## Trade-offs

- The power rule in its basic form requires integer (or later rational/real) exponents; for negative or fractional powers, the general form via $\exp(p \ln x)$ requires $x > 0$.
- The quotient rule requires $g(x) \neq 0$ at the point of differentiation.
- The product and quotient rules assume both $f$ and $g$ are differentiable at the point.

## Links

- [[definition_of_derivative|Definition of the Derivative]]
- [[chain_rule|Chain Rule]]
- [[common_derivatives|Common Derivatives]]
