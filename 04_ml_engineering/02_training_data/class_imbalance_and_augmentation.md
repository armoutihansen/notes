---
layer: 04_ml_engineering
type: engineering
tool: general
status: growing
tags: [class-imbalance, data-augmentation, smote, resampling, synthetic-data]
created: 2026-03-05
---

# Class Imbalance and Data Augmentation

## Purpose

Real-world datasets are rarely balanced. Fraud occurs in 0.1% of transactions; rare diseases affect 1 in 10,000 patients; critical manufacturing defects appear in 0.5% of units. A model trained naively on imbalanced data learns to exploit the imbalance rather than detect the minority class. This note covers the root causes of imbalance, strategies to address it, and augmentation techniques to increase the effective diversity of training data.

## Architecture

### Root Cause of Imbalance

Class imbalance arises from two sources:

1. **Natural imbalance**: The underlying phenomenon is genuinely rare (fraud, disease, safety events). The data accurately reflects the world.
2. **Collection bias**: The data collection process over- or under-samples certain classes. A product review dataset may have few 1-star reviews because dissatisfied users churn rather than write reviews.

These require different responses. Natural imbalance requires modelling the prior carefully and setting decision thresholds deliberately. Collection bias requires correcting the sample to better reflect the true distribution.

### The Accuracy Paradox

On a dataset with 99% negative examples, a model that always predicts "negative" achieves 99% accuracy. This is the accuracy paradox: high accuracy coexists with a useless model. For imbalanced datasets, always evaluate with **precision, recall, F1 (or Fβ), AUROC, and AUPRC**. AUPRC (area under the precision-recall curve) is particularly informative when the positive class is rare; AUROC can be misleadingly optimistic.

### Resampling

**Oversampling** increases the representation of the minority class:

- **Random oversampling**: Duplicate minority examples at random. Simple but increases overfitting risk since no new information is added.
- **SMOTE (Synthetic Minority Oversampling Technique)**: Generate synthetic minority examples by interpolating between real minority examples in feature space. For each minority point, select one of its *k*-nearest minority neighbours and create a new point along the line segment between them. Reduces overfitting risk compared to duplication. ADASYN is an adaptive extension that generates more examples in harder-to-classify regions.

**Undersampling** removes majority examples:

- **Random undersampling**: Discard majority examples at random. Fast but discards potentially useful information.
- **Tomek links**: Remove majority examples that form a Tomek link with a minority example (i.e., they are mutual nearest neighbours across the class boundary). Cleans the decision boundary without large information loss.
- **Cluster centroids**: Replace clusters of majority examples with their centroids. More aggressive information compression.

In practice, **combining oversampling and undersampling** often outperforms either alone. A common recipe: SMOTE the minority class to ~50% representation, then randomly undersample the majority class. The `imbalanced-learn` library implements all standard methods.

### Cost-Sensitive Learning

Instead of changing the dataset, adjust the **loss function** to penalise misclassification of the minority class more heavily.

**Class weights**: Pass `class_weight='balanced'` (scikit-learn) or compute `n_samples / (n_classes * np.bincount(y))`. The model's loss for each example is scaled by the inverse class frequency, making minority class errors proportionally more costly.

**Focal loss** (Lin et al., 2017, RetinaNet): Down-weights the loss for easy examples (high-confidence correct predictions) and up-weights it for hard ones. `FL(p_t) = -α_t(1 - p_t)^γ log(p_t)`. The γ parameter (typically 2) controls focus; α handles class imbalance. Particularly effective for dense object detection and any task with extreme imbalance.

Cost-sensitive learning does not change the dataset, making it easier to combine with other regularisation techniques. It is the preferred approach when the training set is already representative of the deployment distribution.

### Data Augmentation

Augmentation creates new training examples by applying semantics-preserving transformations to existing ones. Its primary purpose is regularisation (reducing overfitting) as well as addressing imbalance for the minority class.

**Image augmentation** (torchvision, Albumentations):
- Geometric: horizontal flip, rotation (±30°), crop, resize, affine transform.
- Colour: brightness/contrast jitter, hue shift, grayscale conversion, channel dropout.
- Advanced: CutOut (randomly zeroing patches), GridDistortion, Elastic Transform.
- MixUp: linear interpolation of two images and their labels: `x̃ = λx_i + (1-λ)x_j`, `ỹ = λy_i + (1-λ)y_j`. Encourages linear behaviour between training examples; strong regulariser.

**Text augmentation**:
- **Synonym replacement**: Replace *n* non-stopword tokens with synonyms (WordNet, word embeddings).
- **Back-translation**: Translate to another language and back. Preserves semantics while varying surface form; computationally expensive.
- **Random insertion/deletion/swap**: EDA (Easy Data Augmentation, Wei & Zou 2019) combines these four operations.
- **Contextual augmentation**: Replace tokens with MLM predictions from BERT.

**Tabular augmentation**:
- SMOTE operates in feature space (described above).
- **Gaussian noise injection**: Add small Gaussian noise to continuous features.
- **Feature permutation**: Shuffle correlated features within class; useful for tree models.

### Synthetic Data Generation

When real data is insufficient, synthetic data generation can supplement it:

- **GANs (CTGAN, TVAE)**: Learn the joint distribution of tabular data and sample from it. High quality but mode collapse is a risk; requires careful evaluation.
- **Variational Autoencoders**: Generate synthetic examples by sampling the latent space; smoother than GANs but sometimes blurrier.
- **Rule-based simulation**: Domain-specific simulators (physics engines, financial models) generate high-quality synthetic data when ground truth dynamics are known.

Synthetic data should always be evaluated on its utility for downstream models, not just its statistical fidelity to the real distribution.

### When Each Approach is Appropriate

| Imbalance ratio | Recommended approach |
|---|---|
| 1:2 to 1:10 | Class weights; adjust decision threshold |
| 1:10 to 1:100 | SMOTE + class weights; focal loss |
| 1:100 to 1:1000 | SMOTE, undersampling, augmentation, anomaly detection framing |
| >1:1000 | Anomaly detection or one-class classification; synthetic data |

## Implementation Notes

- Always apply resampling only to the **training set**. Resampling the validation or test set biases evaluation.
- When using SMOTE, apply it after train/test split to prevent data leakage.
- For decision-making, tune the **classification threshold** post-training to achieve the desired precision-recall trade-off rather than relying on the default 0.5. Use the PR curve on validation data.
- Log the class distribution before and after resampling as part of the data pipeline metadata.

## Trade-offs

| Strategy | Pros | Cons |
|---|---|---|
| Class weights | Simple, no dataset modification | May not correct severe imbalance |
| Oversampling (random) | Simple | Overfitting risk |
| SMOTE | Reduces overfitting vs. random oversampling | Assumes linear interpolation is valid; poor for high-dimensional sparse data |
| Undersampling | Reduces training time | Information loss |
| Augmentation | Strong regularisation | Domain-specific; wrong augmentations can hurt |
| Synthetic (GAN) | Can generate arbitrarily large datasets | Training instability; distribution mismatch risk |

## References

- Chawla, N. V. et al. (2002). *SMOTE: Synthetic Minority Over-sampling Technique*. JAIR.
- Lin, T-Y. et al. (2017). *Focal Loss for Dense Object Detection*. ICCV.
- Zhang, H. et al. (2018). *Mixup: Beyond Empirical Risk Minimization*. ICLR.
- Wei, J. & Zou, K. (2019). *EDA: Easy Data Augmentation Techniques*. EMNLP.

## Links
- [[data_labeling|Data Labeling Strategies]]
- [[dataset_versioning|Dataset Versioning and Lineage]]
- [[regularization|Regularization]]
- [[loss_functions|Loss Functions]]
- [[fairness_metrics|Fairness Metrics]]
