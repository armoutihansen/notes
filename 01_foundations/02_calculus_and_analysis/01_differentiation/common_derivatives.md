---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Common Derivatives

## Definition

Derivatives of the standard elementary functions: exponentials, logarithms, trigonometric functions, and inverse trigonometric functions.

## Intuition

These derivatives are the leaves of the differentiation tree — once known, any combination of elementary functions can be differentiated by applying the algebraic rules. The exponential $e^x$ is special because it is its own derivative; logarithm and inverse trig derivatives emerge from implicit differentiation of the defining identities.

## Formal Description

### Exponential and Logarithm

**General exponential:** For base $a > 0$,
$$
\frac{d}{dx}a^{x} = \lim_{h \to 0} \frac{a^{x+h} - a^{x}}{h} = a^{x} \lim_{h \to 0} \frac{a^{h}-1}{h}.
$$
The limit $\displaystyle\lim_{h \to 0} \frac{a^{h}-1}{h}$ equals $1$ precisely when $a = e$.

**Proof for $a = e$:** Using $e = \lim_{m \to 0}(1+m)^{1/m}$ and the binomial expansion,
$$
\frac{d}{dx}e^{x}\bigg|_{x=0} = \lim_{h \to 0} \frac{e^{h}-1}{h} = \lim_{h,m \to 0} \frac{(1+m)^{h/m}-1}{h} = \lim_{h,m \to 0} \frac{1+h+\cdots - 1}{h} = 1.
$$
Therefore:
$$
\frac{d}{dx}e^{x} = e^{x}.
$$

**Natural logarithm:** Since $\ln x$ is the inverse of $e^x$, differentiate both sides of $x = \exp(\ln x)$ with respect to $x$ and apply the chain rule:
$$
1 = \exp(\ln x) \cdot \frac{d}{dx}\ln x = x \cdot \frac{d}{dx}\ln x \implies \frac{d}{dx}\ln x = \frac{1}{x}.
$$

For $x < 0$, applying the chain rule to $\ln(-x)$:
$$
\frac{d}{dx}\ln(-x) = \frac{1}{-x} \cdot (-1) = \frac{1}{x}.
$$

$$
\frac{d}{dx}\ln x = \frac{1}{x} \quad (x > 0), \qquad \frac{d}{dx}\ln|x| = \frac{1}{x} \quad (x \neq 0).
$$

Note: $x^{-1} = \frac{1}{x}$ is the only power $x^n$ not obtainable as the derivative of another power law (since $\frac{d}{dx}x^0 = 0$).

### Trigonometric Functions

**Sine and cosine** are derived from the limit definition using angle-addition formulas:
$$
\frac{d}{dx}\sin x = \cos x \left(\lim_{h \to 0}\frac{\sin h}{h}\right) + \sin x \left(\lim_{h \to 0}\frac{\cos h - 1}{h}\right),
$$

$$
\frac{d}{dx}\cos x = -\sin x \left(\lim_{h \to 0}\frac{\sin h}{h}\right) + \cos x \left(\lim_{h \to 0}\frac{\cos h - 1}{h}\right).
$$
Using $\lim_{h\to 0}\frac{\sin h}{h} = 1$ and $\lim_{h\to 0}\frac{\cos h - 1}{h} = 0$:
$$
\frac{d}{dx}\sin x = \cos x, \qquad \frac{d}{dx}\cos x = -\sin x.
$$

The remaining four functions follow from the quotient rule applied to $\tan x = \frac{\sin x}{\cos x}$, etc.:
$$
\frac{d}{dx}\tan x = \sec^{2}x, \qquad \frac{d}{dx}\cot x = -\csc^{2}x,
$$

$$
\frac{d}{dx}\sec x = \tan x \sec x, \qquad \frac{d}{dx}\csc x = -\cot x \csc x.
$$

### Inverse Trigonometric Functions

Derivatives are obtained via implicit differentiation and the chain rule.

**Arctangent:** From $\tan(\arctan x) = x$,
$$
\frac{1}{\cos^{2}(\arctan x)}\cdot\frac{d}{dx}(\arctan x) = 1.
$$
Using $\cos(\arctan x) = \frac{1}{\sqrt{1+x^{2}}}$ gives:
$$
\frac{d}{dx}\arctan x = \frac{1}{1+x^{2}} \qquad (\text{all real } x).
$$

**Arcsine and Arccosine:** From $\sin(\arcsin x) = x$ and $\cos(\arccos x) = x$, implicit differentiation yields:
$$
\frac{d}{dx}(\arcsin x) = \frac{1}{\cos(\arcsin x)}, \quad \frac{d}{dx}(\arccos x) = -\frac{1}{\sin(\arccos x)}.
$$
Using $\cos(\arcsin x) = \sin(\arccos x) = \sqrt{1 - x^{2}}$:
$$
\frac{d}{dx}\arcsin x = \frac{1}{\sqrt{1-x^{2}}}, \qquad \frac{d}{dx}\arccos x = -\frac{1}{\sqrt{1-x^{2}}}.
$$

## Applications

- Differentiating any elementary function via the chain rule and algebraic rules.
- $\frac{d}{dx}e^x = e^x$ is fundamental to solving differential equations and defining exponential growth.
- Inverse trig derivatives appear in integration formulas (e.g., $\int \frac{dx}{1+x^2} = \arctan x + C$).

## Trade-offs

- $\ln x$ is only defined (as a real function) for $x > 0$; the formula $\frac{d}{dx}\ln|x| = \frac{1}{x}$ extends it to $x \neq 0$.
- Arcsine and arccosine are only defined for $-1 \le x \le 1$, and their derivatives require $-1 < x < 1$ (strict inequality).
- The limit $\lim_{h\to 0}\frac{\sin h}{h} = 1$ holds only when angles are measured in radians.

## Links

- [[differentiation_rules|Differentiation Rules]]
- [[chain_rule|Chain Rule]]
- [[definition_of_derivative|Definition of the Derivative]]
