---
layer: 02_data_science
type: concept
status: growing
tags: [workflow, process]
created: 2026-03-06
---

# Problem Framing

## Definition

The process of translating a real-world task into a well-specified ML problem: choosing the input space, output space, loss function, and success criteria before selecting any model.

## Intuition

Jumping directly to model selection is a common mistake. The right problem formulation determines everything downstream: what data you need, which algorithms are candidates, and how you know you've succeeded. Garbage in, garbage out — and a poorly framed problem can make even perfect modeling useless.

## Formal Description

### Supervised Learning Setup

Given a dataset $\mathcal{D} = \{(\mathbf{x}_i, y_i)\}_{i=1}^n$ where $\mathbf{x}_i \in \mathcal{X}$ and $y_i \in \mathcal{Y}$, find a function $f: \mathcal{X} \to \mathcal{Y}$ that minimizes expected risk:

$$
R(f) = \mathbb{E}_{(\mathbf{x},y) \sim P}[\ell(f(\mathbf{x}), y)]
$$

**Problem types by output space:**

| Problem type | $\mathcal{Y}$ | Default loss | Example |
|---|---|---|---|
| Binary classification | $\{0,1\}$ | Log-loss | Fraud detection |
| Multi-class classification | $\{1,\ldots,K\}$ | Categorical cross-entropy | Image classification |
| Regression | $\mathbb{R}$ | MSE / MAE | House price prediction |
| Multi-label classification | $\{0,1\}^K$ | Binary cross-entropy per label | Tag prediction |
| Ordinal regression | $\{1,\ldots,K\}$ (ordered) | Ordinal loss | Credit rating |
| Ranking | Partial order over items | Pairwise / listwise loss | Search ranking |
| Sequence-to-sequence | $\mathcal{Y}^*$ | Token-level cross-entropy | Machine translation |

**Unsupervised and self-supervised problem types:**

| Problem type | Output space | Default objective | Example |
|---|---|---|---|
| Clustering | Cluster assignments | Inertia / silhouette | Customer segmentation |
| Density estimation | P(x) | Log-likelihood | Anomaly detection |
| Representation learning | Embedding z | Contrastive / reconstruction | Pre-training |
| Reinforcement learning | Policy π | Cumulative reward | Game playing |

### Choosing a Loss Function

The loss must reflect what you actually care about:

- **MSE** is sensitive to outliers (penalises large errors quadratically); prefer **MAE** or **Huber** for heavy-tailed targets.
- **Log-loss** is proper (incentivises calibrated probabilities); use it for probabilistic classifiers.
- **Class imbalance**: consider focal loss, weighted cross-entropy, or optimise directly for the business metric (e.g., PR-AUC instead of accuracy).
- Never optimise accuracy for imbalanced binary problems — it can be misleadingly high.

### Label Quality and Annotation Strategy

Before training, the label itself must be interrogated as rigorously as the features.

**Defining the label precisely:**
- What counts as a positive? (e.g., "fraud" must be defined: chargebacks only, or also disputes?)
- At what time is the label recorded? A 30-day return window produces different labels than a 7-day one.
- Which annotator defines ground truth, and is there a single authoritative source or human consensus?

**Label noise:**
- **Symmetric (random) noise**: a fraction of labels are flipped uniformly. Inflates effective Bayes error; can be partially mitigated by label smoothing or noise-robust losses.
- **Class-conditional (systematic) noise**: label errors cluster in specific classes (e.g., annotators consistently mislabel class A as class B). Far more harmful — can make learning theoretically impossible in the noisy class. Requires auditing, not just smoothing.

**Weak supervision:**
- **Programmatic labelling (Snorkel-style)**: define label functions (heuristics, regular expressions, distant supervision rules) that vote on unlabelled data. Aggregate with a generative model to produce probabilistic labels. Trades annotation cost for label accuracy.
- **Distant supervision**: align unlabelled text with a knowledge base; assume known entity pairs express a relation — fast but produces systematic noise.
- **Pseudo-labels from a teacher model**: train a teacher on clean data, use its high-confidence predictions as labels for unlabelled data. Effective when clean labels are scarce but unlabelled data is abundant.

**Annotation agreement:**
- Compute **Cohen's κ** between annotators before training. κ < 0.6 indicates substantial disagreement — training on such labels often produces a model that learns annotator idiosyncrasies rather than the true signal. Use κ as a quality gate.
- If clean labels are structurally unachievable (e.g., the concept is genuinely ambiguous), reconsider whether this is a supervised problem at all. Alternatives: learn a distribution over labels, reframe as a ranking/preference problem, or use self-supervised pre-training.

### Baseline Strategies

Always establish baselines before building complex models:

1. **Naive baseline**: always predict the majority class or overall mean. Defines the floor.
2. **Human-level performance (HLP)**: upper-bound proxy for irreducible error; skip when unavailable.
3. **Simple model baseline**: linear model, decision stump, or nearest-neighbour. Often surprisingly competitive.
4. **Previous best known result**: published benchmarks or prior production system.

The gap between naive and HLP tells you how much improvement is theoretically possible.

### Temporal and Deployment Considerations

**Point-in-time correctness**: every feature used during training must be a value that would have been available at prediction time. Joins against event tables (e.g., transaction history, clickstream) must be bounded by the prediction timestamp. A single leaky feature can inflate reported performance dramatically and produce a model that is useless in production.

**Temporal leakage**: for any task where data has a time dimension, the train/test split must be time-based (earliest → latest), never random. Random splits allow future information to leak into training and produce optimistic metrics that collapse at deployment.

**Distribution shift risk:**
- **Covariate shift** — P(X) changes between train and deploy (e.g., user demographics shift). Features become less representative; model predictions degrade gradually.
- **Concept drift** — P(Y|X) changes (e.g., the meaning of "fraud" changes as attackers adapt). The model's learned decision boundary becomes incorrect even if features are stable.
- The deployment environment should directly inform feature engineering choices: prefer features known to be stable over time over those that are highly predictive but brittle.

**Batch vs online inference:**
- Batch jobs allow heavier feature computation (multi-table aggregations, ML-derived features, large lookups) because latency tolerance is high.
- Online inference requires features retrievable in sub-100 ms, which typically means pre-computed feature stores, low-cardinality lookups, or lightweight real-time aggregations. This constraint must be identified during problem framing — not after the model is trained.

**Cold start**: for user- or item-based tasks, clarify the cold-start scenario before modeling. What happens for a new user with zero history? What is the fallback? Cold-start handling often drives architecture decisions (e.g., content-based fallback, popularity prior, or a separate cold-start model).

### Data Availability Assessment

Even a perfectly framed problem is infeasible if the necessary data cannot be obtained. Answer these questions before committing to a design:

1. **Inference-time availability**: can every required feature be computed at inference time without access to information that arrives after the prediction timestamp? If not, the feature cannot be used.
2. **Label availability**: is the label obtainable for training examples? Account for annotation cost, annotation lag (e.g., a churn label requires waiting 30 days), and whether historical labels exist at all for cold-start periods.
3. **Representativeness**: is the training distribution representative of the deployment distribution? Systematic exclusions in historical data (survivorship bias, selection bias, policy artifacts) will silently degrade real-world performance.
4. **Minimum viable dataset size**:
   - Rough heuristic for linear models: at least 10× the number of parameters per class.
   - For neural networks, dataset size typically matters more than architecture choice; scaling laws favour more data over deeper models in the low-data regime.
5. **When data is insufficient**: consider transfer learning (pre-train on related data), data augmentation (synthetic examples), semi-supervised learning (unlabelled data), or reframing as a simpler problem with fewer parameters. If none of these are viable, the problem may not be solvable with ML at the required performance level — and that conclusion is itself valuable.

### Success Criteria

Define before any modeling:

- **Primary metric**: the single metric that drives decisions (e.g., F1 @ threshold 0.5, RMSE in dollars).
- **Constraints**: latency (< 100 ms), memory (< 2 GB), interpretability (must explain each decision), fairness (max 5 pp demographic parity gap).
- **Business acceptance threshold**: the minimum performance required for deployment.
- **Offline → online gap**: clarify whether offline metrics proxy for actual business value; validate with an A/B test.

### Framing Checklist

1. What is the prediction target? (exact label definition)
2. What are the natural input features? (avoid leakage)
3. What is the deployment context? (batch vs online, cold-start vs warm-start)
4. Is this a distribution shift risk? (train/test from same distribution?)
5. What are the cost asymmetries? (FP vs FN costs)
6. What human fallback exists?
7. What is the annotation process and what is the expected label noise level?
8. Is the split time-based or random, and is point-in-time correctness enforced?
9. Is data available at inference time, or does it require a separate pipeline?

## Applications

- Reframing fraud detection as a ranking problem (flag top $k$ suspicious cases) rather than binary classification when label noise is high.
- Choosing MAE over MSE for insurance claim amount prediction where large outliers are legitimate.
- Identifying data leakage: a feature recorded after the prediction time must be excluded.

## Trade-offs

- A tightly specified metric may not align with the true business objective (Goodhart's law).
- Multi-objective problems (e.g., accuracy + fairness + cost) require explicit weighting or constraint satisfaction — there is no free lunch.

## References

- Sculley et al. (2015). "Hidden Technical Debt in Machine Learning Systems." NeurIPS.
- Zinkevich (2022). "Rules of Machine Learning: Best Practices for ML Engineering." Google.
- Ratner et al. (2017). "Snorkel: Rapid Training Data Creation with Weak Supervision." VLDB.

## Links

- [[01_foundations/05_statistical_learning_theory/evaluation_metrics|Evaluation Metrics]]
- [[01_foundations/05_statistical_learning_theory/bias_variance_analysis|Bias–Variance Analysis]]
- [[02_data_science/03_exploratory_data_analysis/exploratory_data_analysis|Exploratory Data Analysis]]
- [[03_modeling/06_training_and_regularization/loss_functions|Loss Functions]]
- [[03_modeling/07_evaluation_and_model_selection/index|Evaluation and Validation]]
