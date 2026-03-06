---
layer: 02_modeling
type: concept
status: seed
tags: [speech, sequence_models, keyword_spotting]
created: 2026-03-02
---

# Trigger Word Detection

## Definition

A sequence modeling task of detecting the exact timing of a specific keyword or phrase (trigger word) in an audio stream, used in always-on voice assistants.

## Intuition

The model listens to audio continuously; it needs to detect not just whether a trigger word was said, but when, so the system can activate at the right moment.

## Formal Description

Input: spectrogram features $x^{<1>}, \ldots, x^{<T_x>}$ from a fixed-size audio window.

Output: binary sequence $y^{<1>}, \ldots, y^{<T_y>}$ where $y^{<t>} = 1$ for steps immediately after the trigger word ends; label "1" for several consecutive steps after the trigger word (to handle class imbalance and give the model a larger target window).

Architecture: 1D convolutions (local audio feature extraction) → RNN/LSTM layers → sigmoid output per time step.

Training data: synthesis — record trigger word + random background noise + random other audio, mix at random volume levels; class imbalance: far more 0-labels than 1-labels → weight positive examples more heavily.

## Applications

"Hey Siri", "OK Google", "Alexa" — on-device, low-power, always-on detection; keyword spotting in smart speakers.

## Trade-offs

Model must be small enough to run continuously on device (low FLOPs, low memory); latency requirements (respond within ~200ms of trigger); false positive rate must be very low (annoying to users); false negative rate also matters (missed wakes frustrate users).

## Links

- [[recurrent_networks]]
- [[02_modeling/03_model_families/04_neural_networks/index|Neural Networks Index]]
- [[01_foundations/06_deep_learning_theory/backpropagation_through_time|Backpropagation Through Time — gradient flow through time steps]]
- [[01_foundations/06_deep_learning_theory/cross_entropy_loss|Cross-Entropy Loss — binary classification objective]]
