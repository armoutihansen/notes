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

- [[supervised_learning|Supervised Learning]] — task setup, loss functions, generalization
- [[neural_network_notation|Neural Network Notation]] — layers, activations, forward pass notation
- [[logistic_regression|Logistic Regression]] — binary classification, sigmoid, cross-entropy
- [[activation_functions|Activation Functions]] — sigmoid, ReLU, GELU, Swish, universal approximation
- [[weight_initialization|Weight Initialization]] — Xavier, Kaiming, symmetry breaking

**Training**

- [[backpropagation|Backpropagation]] — computational graph, chain rule, Jacobian chain
- [[backpropagation_through_time|Backpropagation Through Time]] — unrolled RNNs, gradient truncation
- [[gradient_descent|Gradient Descent]] — SGD, mini-batch, learning rate schedules
- [[adaptive_optimizers|Adaptive Optimizers]] — Momentum, RMSProp, Adam, AdamW
- [[gradient_checking|Gradient Checking]] — numerical gradient verification

**Regularisation and normalisation**

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
