---
layer: 01_foundations
type: concept
status: growing
tags: [dropout, regularization, stochastic-depth, mc-dropout, ensemble]
created: 2026-03-06
---

# Dropout

## Definition

**Dropout** is a regularisation technique that randomly sets each neuron's output to zero with probability $p$ during training, independently for each forward pass. At test time, all neurons are active but their outputs are scaled by $(1-p)$ to match the expected activation magnitude.

## Intuition

Dropout prevents neurons from co-adapting — relying on specific other neurons being present. Every training step produces a "thinned" network (a random sub-network), forcing each neuron to be useful on its own. At test time, using all neurons with scaled outputs approximates averaging over all $2^n$ possible thinned networks — an implicit **ensemble**.

Think of it as training a team where no member knows who else will show up: every team member must be able to pull their weight independently.

## Formal Description

**Inverted dropout (standard implementation):**

During training, for each layer activation $\mathbf{a}$:

$$\tilde{a}_i = \begin{cases} a_i / (1-p) & \text{with probability } 1-p \\ 0 & \text{with probability } p \end{cases}$$

The $1/(1-p)$ scaling keeps the expected value of $\tilde{a}_i$ equal to $a_i$, so no test-time correction is needed.

**Effect on gradient:** the backward pass zeroes out the same units that were zeroed in the forward pass. Only the surviving units receive gradient updates.

```python
# PyTorch: nn.Dropout(p=0.5) applied inside forward()
import torch.nn as nn

class MLP(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc1 = nn.Linear(128, 64)
        self.drop = nn.Dropout(p=0.5)
        self.fc2 = nn.Linear(64, 10)

    def forward(self, x):
        return self.fc2(self.drop(torch.relu(self.fc1(x))))
        # drop is a no-op at eval time (model.eval())
```

**Typical dropout rates:**
- Fully connected layers: $p = 0.5$
- Input layer: $p = 0.1$–$0.2$
- Convolutional layers: lower values or no dropout (spatial dropout instead)
- Transformers: $p = 0.1$ applied to attention weights and FFN activations

**Dropout as regularisation:** the expected loss with dropout is approximately the $L_2$-regularised loss; specifically, for linear models, dropout is equivalent to an adaptive $L_2$ penalty that is larger for features with higher variance.

**MC Dropout (Monte Carlo Dropout):** leaving dropout active at test time and running $T$ forward passes:

$$p(y|\mathbf{x}) \approx \frac{1}{T} \sum_{t=1}^T p(y|\mathbf{x}, \hat{\theta}_t)$$

The variance across passes estimates **epistemic uncertainty** (uncertainty about model parameters). Used in Bayesian deep learning for uncertainty quantification.

**Spatial Dropout / DropBlock:** drops entire feature map channels (Spatial Dropout) or contiguous spatial regions (DropBlock) — more effective for convolutional networks where adjacent activations are highly correlated.

**Stochastic Depth:** randomly drops entire residual blocks during training (used in EfficientNet, DeiT). Reduces training time and acts as regulariser.

## Applications

| Use case | Notes |
|---|---|
| MLP regularisation | Classic use; $p = 0.5$ for large hidden layers |
| Transformer pre-training | Light dropout on attention/FFN; often $p = 0.1$ |
| Uncertainty estimation | MC Dropout — cheap Bayesian approximation |
| Small datasets | High dropout rate compensates for limited data |
| Convolutional nets | Prefer DropBlock or Stochastic Depth over elementwise dropout |

## Trade-offs

- Slows convergence: training effectively requires 2×–3× more steps to compensate for dropped neurons.
- Less effective for batch-normalised networks: BN already provides regularisation; dropout and BN can interact poorly (the variance shift problem).
- Modern large models (transformers) rely more on weight decay and attention dropout than dense dropout.
- Not appropriate for output layers or when computing exact posterior inference.

## Links

- [[weight_initialization|Weight Initialization]] — inverted dropout requires re-tuning learning rate and init scheme
- [[batch_normalization|Batch Normalization]] — potential conflict: BN normalises statistics that dropout distorts
- [[layer_normalization|Layer Normalization]] — LN is not affected by dropout the same way BN is
- [[01_foundations/05_statistical_learning_theory/generalization_bounds|Generalization Bounds]] — dropout reduces effective model complexity
- [[01_foundations/04_optimization/regularization_and_penalties|Regularization]] — dropout is equivalent to adaptive $L_2$ regularisation
