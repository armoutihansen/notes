---
layer: 01_foundations
type: index
status: evergreen
tags: []
created: 2026-03-02
---

# Deep Learning Theory Index

Navigation hub.

## Notes

**Foundations**

- [[neural_network_notation|Neural Network Notation]] — layers, activations, forward pass notation
- [[activation_functions|Activation Functions]] — sigmoid, ReLU, GELU, Swish, universal approximation
- [[weight_initialization|Weight Initialization]] — Xavier, Kaiming, symmetry breaking

**Training**

- [[backpropagation|Backpropagation]] — computational graph, chain rule, Jacobian chain
- [[backpropagation_through_time|Backpropagation Through Time]] — unrolled RNNs, gradient truncation
- [[gradient_descent|Gradient Descent]] — SGD, mini-batch, learning rate schedules
- [[adaptive_optimizers|Adaptive Optimizers]] — Momentum, RMSProp, Adam, AdamW
- [[gradient_checking|Gradient Checking]] — numerical gradient verification

**Regularization and normalization**

- [[dropout|Dropout]] — inverted dropout, MC dropout, ensemble interpretation
- [[batch_normalization|Batch Normalization]] — normalise over batch, scale/shift, covariate shift
- [[layer_normalization|Layer Normalization]] — normalise over features, Pre-LN, RMSNorm
- [[residual_connections|Residual Connections]] — skip connections, gradient highway, ensemble theory

**Loss functions**

- [[cross_entropy_loss|Cross-Entropy Loss]] — categorical and binary cross-entropy
- [[triplet_loss|Triplet Loss]] — metric learning, anchor/positive/negative

**Efficiency**

- [[vectorization|Vectorization and Broadcasting]] — batch operations, NumPy/PyTorch broadcasting

## Links
- [[01_foundations/index|Foundations]]
- [[01_foundations/05_statistical_learning_theory/supervised_learning|Supervised Learning]] (→ 05_statistical_learning_theory)
- [[01_foundations/03_probability_and_statistics/logistic_regression|Logistic Regression]] (→ 03_probability_and_statistics)

---

**Navigation:** ← [[01_foundations/05_statistical_learning_theory/index|Statistical Learning Theory]] | [[01_foundations/index|Foundations Index]]
