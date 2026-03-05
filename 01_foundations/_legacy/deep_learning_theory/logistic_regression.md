---
layer: 01_foundations
type: concept
status: seed
tags: [logistic-regression, classification, sigmoid, cross-entropy, gradient-descent]
created: 2026-03-02
---

# Logistic Regression

## Definition

A linear binary classifier that models $P(y=1 \mid x)$ via the sigmoid of a linear combination of inputs; the building block of a neural network neuron.

## Intuition

The sigmoid "squashes" the linear score into a probability in $(0, 1)$. Training maximizes the log-likelihood of the labels, which is equivalent to minimizing cross-entropy. Despite the name, logistic regression is a classifier, not a regressor.

## Formal Description

**Model:**

$$z = w^\top x + b, \qquad \hat y = \sigma(z) = \frac{1}{1+e^{-z}}$$

**Loss per example (binary cross-entropy):**

$$\ell(\hat y, y) = -\bigl[y\log\hat y + (1-y)\log(1-\hat y)\bigr]$$

**Gradients:**

$$\frac{\partial\ell}{\partial w} = (\hat y - y)\,x, \qquad \frac{\partial\ell}{\partial b} = \hat y - y$$

The gradient has a clean form: residual × input.

**Vectorized batch form** (columns are examples):

$$Z = W^\top X + b, \quad A = \sigma(Z)$$

$$dW = \frac{1}{m}X(A-Y)^\top, \qquad db = \frac{1}{m}\mathbf{1}^\top(A-Y)$$

## Applications

- Baseline binary classifier for any task
- Direct interpretation as calibrated probabilities
- Each neuron in a sigmoid-activation network
- Output layer for binary classification problems

## Trade-offs

- Linear decision boundary — underfits non-linear problems
- Assumes features are individually informative; correlated features can cause instability
- Sensitive to class imbalance (consider reweighting or focal loss)
- Easily extended to multi-class via softmax regression

## Links

- [[01_foundations/_legacy/deep_learning_theory/cross_entropy_loss]]
- [[01_foundations/_legacy/deep_learning_theory/gradient_descent]]
- [[01_foundations/_legacy/deep_learning_theory/backpropagation]]
