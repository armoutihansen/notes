---
layer: 00_meta
type: index
status: growing
tags: [glossary, definitions, vocabulary]
created: 2026-03-06
---

# Glossary

One-line definitions for key terms used across the vault. Entries link to the primary source note where one exists. Alphabetically sorted within sections.

---

## Vault Meta

| Term | Definition |
|------|------------|
| **concept note** | Note type for timeless theoretical content; template sections: Definition → Intuition → Formal Description → Applications → Trade-offs → Links. |
| **engineering note** | Note type for tooling, systems, and implementation patterns; template sections: Purpose → Architecture → Implementation Notes → Trade-offs → References → Links. |
| **evergreen** | Highest lifecycle status; note is comprehensive, accurate, and well-linked. |
| **growing** | Intermediate lifecycle status; note has substantive content but may lack cross-links or completeness. |
| **index note** | Note type serving as a sublayer or layer table of contents; lists all notes in lifecycle reading order with prev/next navigation. |
| **seed** | Initial lifecycle status; note is a stub or placeholder needing expansion. |
| **sublayer** | A numbered subdirectory within a layer (e.g., `01_foundations/03_probability_and_statistics/`). |
| **wikilink** | Obsidian-style internal link `[[path/to/note\|Display Text]]`; must use full vault-root paths for Quartz compatibility. |

---

## Mathematics

### Linear Algebra

| Term | Definition |
|------|------------|
| **basis** | A linearly independent spanning set for a vector space; every vector has a unique representation in terms of basis vectors. |
| **dot product** | $\mathbf{u}\cdot\mathbf{v} = \sum_i u_i v_i$; measures alignment between vectors; equals $\|\mathbf{u}\|\|\mathbf{v}\|\cos\theta$. |
| **eigenvalue / eigenvector** | Scalar $\lambda$ and non-zero vector $\mathbf{v}$ satisfying $A\mathbf{v}=\lambda\mathbf{v}$; eigenvectors are invariant directions under $A$. See [[01_foundations/01_linear_algebra/03_eigenvalues_and_decompositions/index\|Eigenvalues]]. |
| **gradient** | Vector of partial derivatives $\nabla_\mathbf{x} f \in \mathbb{R}^n$; points in the direction of steepest ascent. See [[01_foundations/02_calculus_and_analysis/01_differentiation/index\|Differentiation]]. |
| **Hessian** | Matrix of second-order partial derivatives $H_{ij} = \partial^2 f / \partial x_i \partial x_j$; characterises local curvature. |
| **matrix rank** | Number of linearly independent rows (or columns); determines solution space of $A\mathbf{x}=\mathbf{b}$. |
| **norm** | Function $\|\cdot\|$ measuring vector magnitude; $\ell^1$: $\sum|x_i|$; $\ell^2$: $\sqrt{\sum x_i^2}$; $\ell^\infty$: $\max|x_i|$. |
| **orthogonal matrix** | Square matrix $Q$ with $Q^T Q = I$; columns form an orthonormal basis; preserves lengths and angles. |
| **positive semidefinite (PSD)** | Symmetric matrix $A$ with $\mathbf{x}^T A \mathbf{x} \geq 0$ for all $\mathbf{x}$; all eigenvalues $\geq 0$; arises in covariance matrices and kernels. |
| **singular value decomposition (SVD)** | Factorisation $A = U\Sigma V^T$; $U,V$ orthogonal, $\Sigma$ diagonal; foundational for PCA, low-rank approximation, and pseudoinverse. |
| **span** | Set of all linear combinations of a collection of vectors. |
| **trace** | Sum of diagonal entries of a square matrix; equals sum of eigenvalues; invariant under cyclic permutations. |
| **transpose** | Matrix $A^T$ with rows and columns swapped; $(AB)^T = B^T A^T$. |

### Calculus & Analysis

| Term | Definition |
|------|------------|
| **chain rule** | $\frac{d}{dx}f(g(x)) = f'(g(x))\cdot g'(x)$; extended to vectors via Jacobians; backbone of backpropagation. See [[01_foundations/02_calculus_and_analysis/01_differentiation/chain_rule\|Chain Rule]]. |
| **Jacobian** | Matrix of all first-order partial derivatives of a vector-valued function $\mathbf{f}:\mathbb{R}^n\to\mathbb{R}^m$; $J_{ij}=\partial f_i/\partial x_j$. |
| **Taylor expansion** | Approximation of $f$ near $x_0$ as a polynomial in $(x-x_0)$; first-order: $f(x)\approx f(x_0)+f'(x_0)(x-x_0)$. |

### Probability & Statistics

| Term | Definition |
|------|------------|
| **Bayes' theorem** | $P(B\mid A) = P(A\mid B)P(B)/P(A)$; updates prior belief with likelihood to obtain a posterior. See [[01_foundations/03_probability_and_statistics/bayesian_inference\|Bayesian Inference]]. |
| **central limit theorem (CLT)** | Sample mean of $n$ i.i.d. variables with finite variance converges in distribution to $\mathcal{N}(\mu, \sigma^2/n)$ as $n\to\infty$. See [[01_foundations/03_probability_and_statistics/probability_distributions\|Distributions]]. |
| **conditional independence** | $X \perp Y \mid Z$ iff $P(X,Y\mid Z)=P(X\mid Z)P(Y\mid Z)$; foundational for graphical models and Naive Bayes. |
| **conjugate prior** | A prior $p(\theta)$ whose posterior $p(\theta\mid x)$ is in the same parametric family; simplifies Bayesian updates analytically. See [[01_foundations/03_probability_and_statistics/bayesian_inference\|Bayesian Inference]]. |
| **covariance** | $\text{Cov}(X,Y)=\mathbb{E}[(X-\mu_X)(Y-\mu_Y)]$; measures linear co-variation; zero for independent variables. |
| **expected value** | $\mathbb{E}[X]=\sum_x x\,p(x)$ or $\int x\,f(x)\,dx$; linear operator (linearity holds without independence). |
| **exponential family** | Class of distributions with density $p(x\mid\eta)=h(x)\exp(\eta^T T(x)-A(\eta))$; includes Gaussian, Bernoulli, Poisson, Gamma. |
| **hypothesis testing** | Statistical procedure testing a null hypothesis $H_0$ using a test statistic; controlled by significance level $\alpha$ (Type I error rate). See [[01_foundations/03_probability_and_statistics/hypothesis_testing\|Hypothesis Testing]]. |
| **MAP estimate** | Maximum a posteriori: $\hat\theta = \arg\max_\theta p(\theta\mid x) = \arg\max_\theta p(x\mid\theta)p(\theta)$; MLE with a prior. |
| **maximum likelihood estimation (MLE)** | $\hat\theta = \arg\max_\theta p(x\mid\theta)$; finds parameters most consistent with observed data. |
| **p-value** | Probability of observing a test statistic at least as extreme as the observed value under $H_0$; not the probability $H_0$ is true. |
| **posterior** | $p(\theta\mid x) \propto p(x\mid\theta)p(\theta)$; updated belief about parameters after observing data. |
| **prior** | $p(\theta)$; belief about parameters before observing data. |
| **variance** | $\text{Var}(X)=\mathbb{E}[(X-\mathbb{E}[X])^2]$; measures spread; $\text{Var}(aX)=a^2\text{Var}(X)$. |

### Optimisation

| Term | Definition |
|------|------------|
| **Adam** | Adaptive gradient optimiser combining momentum and RMSProp; per-parameter learning rates using first and second moment estimates. See [[01_foundations/04_optimization/gradient_descent_optimization\|Gradient Descent]]. |
| **convex function** | $f$ satisfying $f(\lambda x+(1-\lambda)y)\leq\lambda f(x)+(1-\lambda)f(y)$; any local minimum is global. See [[01_foundations/04_optimization/convex_optimization\|Convex Optimisation]]. |
| **duality** | Relationship between a primal optimisation problem and its Lagrangian dual; strong duality holds under Slater's condition. |
| **gradient descent** | Iterative optimisation: $\theta_{t+1}=\theta_t - \alpha\nabla_\theta L$; converges to a local minimum for smooth objectives. See [[01_foundations/04_optimization/gradient_descent_optimization\|Gradient Descent]]. |
| **KKT conditions** | Necessary (and sufficient for convex problems) first-order conditions for constrained optimisation; stationarity, primal/dual feasibility, complementary slackness. See [[01_foundations/04_optimization/lagrangian_and_constrained_optimization\|Lagrangian]]. |
| **Lagrangian** | $\mathcal{L}(x,\lambda,\mu)=f(x)+\sum_i\lambda_i g_i(x)+\sum_j\mu_j h_j(x)$; encodes constrained problem as unconstrained via multipliers. |
| **learning rate** | Step size $\alpha$ in gradient descent; too large → divergence; too small → slow convergence; typically scheduled or adapted. |
| **SGD (stochastic gradient descent)** | Gradient descent using a mini-batch (or single sample) gradient estimate per step; noisier but faster per update and better for large datasets. |

---

## Modeling

| Term | Definition |
|------|------------|
| **ARIMA** | Auto-regressive integrated moving average; classical time series model for stationary (after differencing) univariate series. See [[02_modeling/03_model_families/09_time_series_models/time_series_models\|Time Series Models]]. |
| **attention mechanism** | Soft, differentiable lookup: computes a weighted sum of values $V$ using similarity scores between queries $Q$ and keys $K$. See [[02_modeling/03_model_families/07_transformers/attention_mechanism\|Attention]]. |
| **AUC-ROC** | Area under the receiver operating characteristic curve; threshold-invariant classification metric equal to $P(\hat p_+ > \hat p_-)$. See [[02_modeling/05_evaluation_and_validation/evaluation_and_validation\|Evaluation]]. |
| **bias–variance trade-off** | Generalisation error = bias² + variance + irreducible noise; simpler models → higher bias; complex models → higher variance. See [[01_foundations/05_statistical_learning_theory/bias_variance_analysis\|Bias–Variance]]. |
| **calibration** | A model is calibrated if predicted probabilities match empirical frequencies; assessed via reliability diagram and ECE. See [[02_modeling/05_evaluation_and_validation/evaluation_and_validation\|Evaluation]]. |
| **cross-validation (k-fold)** | Partition data into $k$ folds; train on $k-1$, evaluate on 1; repeat $k$ times; provides $k\times$ more evaluation signal than a single hold-out. |
| **DBSCAN** | Density-based clustering; groups core points within $\varepsilon$-neighbourhoods; handles arbitrary shapes and labels outliers as noise. See [[02_modeling/03_model_families/06_unsupervised_learning/unsupervised_learning\|Unsupervised Learning]]. |
| **decision tree** | Recursive binary partition of feature space; splits chosen to minimise impurity (Gini/entropy for classification, MSE for regression). |
| **dropout** | Regularisation technique: randomly zero activations during training with probability $p$; reduces co-adaptation of neurons. See [[02_modeling/04_training_dynamics/regularization\|Regularisation]]. |
| **early stopping** | Halt training when validation loss stops improving; prevents overfitting; a form of implicit regularisation. See [[02_modeling/04_training_dynamics/early_stopping\|Early Stopping]]. |
| **ElasticNet** | Regularised regression combining L1 (Lasso) and L2 (Ridge) penalties; $\lambda[\alpha\|\beta\|_1 + (1-\alpha)\|\beta\|_2^2/2]$. |
| **F1 score** | Harmonic mean of precision and recall: $2\text{PR}/(P+R)$; useful for imbalanced classes. |
| **feature engineering** | Creating informative input features from raw data; includes polynomial features, lag features, cyclical encoding, and target encoding. See [[02_modeling/02_data_science/feature_engineering\|Feature Engineering]]. |
| **GLM (generalised linear model)** | Extends linear regression to non-Gaussian targets via a link function; special cases include logistic regression (logit link) and Poisson regression (log link). See [[02_modeling/03_model_families/01_linear_and_glm/linear_and_glm\|Linear Models & GLMs]]. |
| **GMM (Gaussian mixture model)** | Probabilistic clustering model: $p(x)=\sum_k\pi_k\mathcal{N}(x\mid\mu_k,\Sigma_k)$; fitted via EM algorithm. See [[02_modeling/03_model_families/03_probabilistic_models/probabilistic_models\|Probabilistic Models]]. |
| **gradient boosting** | Ensemble method that sequentially fits shallow trees to the residuals (pseudo-gradients) of the current ensemble; XGBoost and LightGBM are implementations. |
| **hyperparameter** | Model configuration set before training (e.g., learning rate, regularisation strength, number of trees); tuned via cross-validation or Bayesian optimisation. |
| **k-means** | Iterative clustering: assign each point to nearest centroid, recompute centroids; minimises within-cluster sum of squares. See [[02_modeling/03_model_families/06_unsupervised_learning/unsupervised_learning\|Unsupervised Learning]]. |
| **kernel trick** | Implicitly computes inner products in a high-dimensional feature space via a kernel function $k(x,x')$; avoids explicit feature map computation. See [[02_modeling/03_model_families/05_kernel_methods/kernel_methods\|Kernel Methods]]. |
| **Lasso (L1 regularisation)** | Adds $\lambda\|\beta\|_1$ to loss; induces sparsity by driving coefficients exactly to zero; performs feature selection. |
| **LightGBM** | Gradient boosting library using leaf-wise tree growth and histogram-based splits; faster than XGBoost on large datasets. |
| **logistic regression** | Linear classifier applying sigmoid to a linear combination of features; outputs calibrated probabilities for binary classification. |
| **loss function** | Scalar measuring prediction error; cross-entropy for classification, MSE for regression; optimised during training. See [[02_modeling/04_training_dynamics/loss_functions\|Loss Functions]]. |
| **LSTM** | Long Short-Term Memory; RNN variant with gated cell state; captures long-range sequential dependencies. See [[02_modeling/03_model_families/04_neural_networks/recurrent_networks\|Recurrent Networks]]. |
| **Naive Bayes** | Probabilistic classifier assuming conditional independence of features given class: $P(y\mid x)\propto P(y)\prod_i P(x_i\mid y)$. |
| **overfitting** | Model performs well on training data but poorly on unseen data; caused by excess capacity relative to data; mitigated by regularisation, early stopping, or more data. |
| **PCA (principal component analysis)** | Linear dimensionality reduction via SVD; projects data onto directions of maximum variance. See [[02_modeling/03_model_families/06_unsupervised_learning/unsupervised_learning\|Unsupervised Learning]]. |
| **precision / recall** | Precision = TP/(TP+FP); recall = TP/(TP+FN); precision-recall trade-off is controlled by classification threshold. |
| **random forest** | Ensemble of decision trees trained on bootstrap samples with random feature subsets; reduces variance via averaging. |
| **regularisation** | Penalty on model complexity added to loss (L1/L2) or applied implicitly (dropout, early stopping); reduces overfitting. See [[02_modeling/04_training_dynamics/regularization\|Regularisation]]. |
| **Ridge (L2 regularisation)** | Adds $\lambda\|\beta\|_2^2$ to loss; shrinks coefficients towards zero but does not set them exactly to zero. |
| **SHAP** | SHapley Additive exPlanations; decomposes model predictions into additive feature contributions with game-theoretic guarantees. See [[02_modeling/06_interpretability/shap_and_feature_attribution\|SHAP]]. |
| **softmax** | $\text{softmax}(z)_i = e^{z_i}/\sum_j e^{z_j}$; converts logits to a probability distribution over classes. |
| **stationarity** | A time series is weakly stationary if its mean and autocovariance are time-invariant; required by ARIMA; tested with the ADF test. |
| **SVM (support vector machine)** | Maximum-margin classifier; finds the hyperplane maximising the margin between classes; extended to non-linear boundaries via kernels. See [[02_modeling/03_model_families/05_kernel_methods/kernel_methods\|Kernel Methods]]. |
| **t-SNE** | Non-linear dimensionality reduction for 2D/3D visualisation; preserves local neighbourhood structure; not suitable for downstream ML. |
| **transformer** | Deep learning architecture using multi-head self-attention and positional encoding; dominant for sequence tasks. See [[02_modeling/03_model_families/07_transformers/transformer\|Transformer]]. |
| **UMAP** | Non-linear dimensionality reduction; faster than t-SNE and better preserves global structure; suitable for large datasets. |
| **XGBoost** | Gradient boosting library with second-order gradient approximation, regularisation, and parallel tree construction. |

---

## ML Engineering

| Term | Definition |
|------|------------|
| **A/B test** | Controlled experiment comparing a treatment (new model) to a control (current model) on live traffic; used to measure real-world impact. |
| **canary deployment** | Gradually shift a fraction of traffic to a new model version; roll back if metrics degrade before full rollout. |
| **data drift** | Change in input feature distribution $P(X)$ between training and production; detected via statistical tests (KS, PSI). |
| **feature store** | Centralised repository for computed features with consistent serving between training and inference; prevents train–serve skew. See [[04_ml_engineering/03_feature_engineering/index\|Feature Engineering]]. |
| **MLflow** | Open-source ML lifecycle platform: experiment tracking, model registry, and model serving. See [[04_ml_engineering/04_model_development/index\|Model Development]]. |
| **model registry** | Versioned store of trained model artefacts with stage transitions (Staging → Production); enables reproducibility and governance. |
| **online learning** | Model updates continuously as new data arrives; suited for non-stationary environments. |
| **PSI (population stability index)** | Measures shift between two distributions; PSI < 0.1: stable; 0.1–0.25: minor shift; > 0.25: major drift requiring investigation. |
| **training–serving skew** | Discrepancy between features computed at training time and at inference time; causes silent model degradation. |

---

## AI Engineering

| Term | Definition |
|------|------------|
| **chunking** | Splitting documents into segments for embedding and retrieval in RAG pipelines; chunk size and overlap affect recall and coherence. |
| **context window** | Maximum number of tokens a language model can process in a single forward pass; limits document length and conversation history. |
| **embedding** | Dense vector representation of text, image, or other data in a latent semantic space; similarity is measured by cosine or dot product. |
| **fine-tuning** | Further training a pre-trained model on a task-specific dataset; updates weights (full fine-tuning) or only adapters (LoRA/QLoRA). See [[05_ai_engineering/04_finetuning/index\|Fine-tuning]]. |
| **hallucination** | LLM generating plausible-sounding but factually incorrect or unsupported content; mitigated by RAG, grounding, and output validation. |
| **KV cache** | Cached key-value attention tensors for previously processed tokens; enables efficient autoregressive decoding without recomputation. |
| **LoRA (Low-Rank Adaptation)** | Parameter-efficient fine-tuning: learns low-rank updates $\Delta W = AB$ added to frozen weight matrices; far fewer trainable parameters. See [[05_ai_engineering/04_finetuning/index\|Fine-tuning]]. |
| **prompt engineering** | Designing input text to elicit desired LLM behaviour; includes chain-of-thought, few-shot examples, and system prompts. See [[05_ai_engineering/02_prompt_engineering/index\|Prompt Engineering]]. |
| **quantisation** | Reducing model weight precision (FP32 → INT8/INT4) to decrease memory footprint and increase inference throughput with minimal accuracy loss. See [[05_ai_engineering/06_inference_optimization/index\|Inference Optimisation]]. |
| **RAG (retrieval-augmented generation)** | Combines a retrieval system (vector search over a knowledge base) with an LLM; grounds responses in retrieved context. See [[05_ai_engineering/03_rag_and_agents/index\|RAG and Agents]]. |
| **reranker** | A cross-encoder model that scores query–document pairs for relevance; used after initial retrieval to improve precision before LLM generation. |
| **system prompt** | Instruction prepended to a conversation that shapes LLM behaviour, persona, and constraints; not visible to end-users in production. |
| **temperature** | Sampling parameter scaling logits before softmax; higher → more random outputs; temperature = 0 → greedy (deterministic) decoding. |
| **token** | Basic unit of text processed by an LLM; typically ~0.75 words for English; models have a fixed context window measured in tokens. |
| **tool use / function calling** | LLM capability to invoke external tools (APIs, code interpreters, databases) by outputting structured JSON; enables agentic behaviour. |
| **vLLM** | High-throughput LLM inference library using PagedAttention for efficient KV cache management; enables continuous batching. See [[05_ai_engineering/06_inference_optimization/index\|Inference Optimisation]]. |

---

## Links

- [[00_meta/conventions.md|Conventions]]
- [[01_foundations/index|Foundations]]
- [[02_modeling/index|Modeling]]
- [[03_software_engineering/index|Software Engineering]]
- [[04_ml_engineering/index|ML Engineering]]
- [[05_ai_engineering/index|AI Engineering]]
