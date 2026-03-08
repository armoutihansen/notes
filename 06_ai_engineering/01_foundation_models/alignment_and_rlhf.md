---
layer: 06_ai_engineering
type: engineering
tool: general
status: growing
tags: [fine-tuning, safety]
created: 2026-03-05
---

# Alignment and RLHF

## Purpose

Pre-trained language models optimise for next-token prediction on internet text — a distribution that includes harmful, biased, and incoherent content. **Alignment** is the process of steering a pre-trained model to be **helpful, harmless, and honest (HHH)**; to follow user instructions; and to decline dangerous or deceptive requests.

The practical outcome is the difference between a raw base model (completion-only, no instruction following, unpredictable refusals) and a chat model (instruction-following, safety-aware, consistent persona). All deployed chat models (ChatGPT, Claude, Gemini, Llama-Chat) have been through an alignment pipeline.

Alignment is increasingly a safety and compliance engineering concern, not just a research topic. Models deployed in products must behave predictably across adversarial inputs, jailbreaks, and distribution shift.

## Architecture

### Stage 1 — Supervised Fine-Tuning (SFT)

Fine-tune the pre-trained base model on a **curated dataset of (prompt, ideal response) pairs** written or filtered by human annotators. This teaches the model the desired response format, tone, and instruction-following behaviour.

- Dataset: 10K–100K high-quality demonstrations.
- Training: standard autoregressive cross-entropy loss on response tokens only (mask the prompt).
- Result: a model that follows instructions but may still produce harmful outputs or be easily manipulated.

### Stage 2 — Reward Model (RM) Training

Train a **reward model** that predicts human preference scores. Data collection: for the same prompt, generate two or more completions; human annotators rank them. Use the **Bradley-Terry model** to convert pairwise rankings into scalar reward scores.

$$
L_{\text{RM}} = -\log \sigma(r(x, y_w) - r(x, y_l))
$$

where `r` is the reward model, `y_w` is the preferred completion, `y_l` is the rejected one. The RM is typically the SFT model with its final language-model head replaced by a scalar regression head.

### Stage 3 — RL with PPO (RLHF)

Use **Proximal Policy Optimisation (PPO)** to fine-tune the SFT model to maximise reward model scores while staying close to the SFT distribution (KL divergence penalty):

$$
\text{Objective} = \mathbb{E}[r_\theta(x, y)] - \beta \cdot \text{KL}[\pi_\theta(y|x) \| \pi_{\text{SFT}}(y|x)]
$$

- `β` controls the KL penalty; too low → reward hacking; too high → no improvement.
- Requires four models in GPU memory simultaneously: policy, reference (SFT), reward, value function.
- Computationally expensive: 2–4× the cost of SFT.

### Constitutional AI (Anthropic)

An alternative to human preference data:

1. **Critique-Revision**: generate a response; prompt the model to critique it against a set of **Constitutional Principles** (e.g., "Is this response harmful?"); revise based on the critique. Repeat iteratively to generate improved SFT data.
2. **RLAIF (RL from AI Feedback)**: replace human preference labellers with a stronger AI judge. The judge evaluates response pairs against the constitution; these AI preferences train the reward model.

Constitutional AI reduces reliance on human annotation and makes alignment principles explicit and auditable.

### DPO — Direct Preference Optimization

DPO (Rafailov et al., 2023) eliminates the reward model and RL loop entirely. It derives a **closed-form update** that directly trains the policy from (prompt, chosen, rejected) preference triples:

$$
L_{\text{DPO}} = -\mathbb{E}\left[\log \sigma\left(\beta \log\frac{\pi_\theta(y_w|x)}{\pi_{\text{ref}}(y_w|x)} - \beta \log\frac{\pi_\theta(y_l|x)}{\pi_{\text{ref}}(y_l|x)}\right)\right]
$$

This is mathematically equivalent to RLHF with an implicit reward, but:
- No separate reward model to train or maintain.
- No RL loop; standard supervised fine-tuning optimiser (AdamW).
- More stable training; less hyperparameter sensitivity.
- Requires the reference SFT model in memory for computing log-ratios.

**GRPO** (Group Relative Policy Optimisation, DeepSeek R1) is a PPO variant that eliminates the value function by using group-relative baselines, reducing memory and improving stability for reasoning-heavy tasks.

## Implementation Notes

**Data quality dominates**: for SFT, 10K high-quality examples outperform 100K noisy examples. Annotator consistency and rubric clarity are critical. Common sources: OpenAssistant, ShareGPT, Alpaca, UltraChat.

**Reward hacking risks**: the reward model is an imperfect proxy for human preference. Without KL penalty, models learn to exploit RM weaknesses — producing verbose, sycophantic, or superficially impressive-sounding outputs. Monitor: average response length drift, diversity metrics, held-out human evaluations.

**KL divergence penalty** prevents the policy from diverging too far from the SFT model. If the policy collapses to repetitive or degenerate outputs, increase `β`. If improvement plateaus, decrease `β`.

**Constitutional AI as a self-supervised tool**: useful when human preference data is expensive or unavailable for a new domain. Define a domain-specific constitution (e.g., for medical AI: accuracy, safety, citing uncertainty) and generate RLAIF labels.

**HuggingFace TRL library** provides production-ready implementations:

```python
from trl import SFTTrainer, DPOTrainer, PPOTrainer, RewardTrainer
```

`DPOTrainer` is the recommended starting point for most teams: simpler, stable, no RL infra required.

**Evaluation during alignment**: track reward model scores on a held-out set, win-rate against the SFT baseline (using a judge model), and safety red-teaming pass rates.

## Trade-offs

| Method | Pros | Cons |
|---|---|---|
| RLHF (PPO) | State-of-the-art; flexible reward shaping | Complex infra; 4 models in memory; unstable |
| DPO | Simple; stable; no RM needed | Requires preference data; no online exploration |
| Constitutional AI | Scalable; reduces human annotation | Requires capable base model for self-critique |
| SFT only | Cheap; fast | Does not capture preferences; easily jailbroken |

RLHF remains the gold standard for frontier models but is operationally expensive. DPO is the practical default for most fine-tuning pipelines. Constitutional AI is best for teams that need scalable self-supervised alignment signals.

## References

- Ouyang et al. (2022). *Training language models to follow instructions with human feedback* (InstructGPT). OpenAI.
- Rafailov et al. (2023). *Direct Preference Optimization: Your Language Model is Secretly a Reward Model*. Stanford.
- Bai et al. (2022). *Constitutional AI: Harmlessness from AI Feedback*. Anthropic.
- Schulman et al. (2017). *Proximal Policy Optimization Algorithms*. OpenAI.
- Sheng et al. (2024). *DeepSeek-R1*: GRPO for reasoning.

## Links

- [[foundation_model_overview|Foundation Model Overview]]
- [[tokenization|Tokenization]]
- [[rl_finetuning|Reinforcement Learning Fine-tuning]]
- [[distributed_training|Distributed Training]]
