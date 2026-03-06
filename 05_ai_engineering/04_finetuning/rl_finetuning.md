---
layer: 05_ai_engineering
type: engineering
tool: trl
status: evergreen
tags: [rlhf, dpo, grpo, ppo, alignment, trl, preference-learning]
created: 2026-03-05
---

# Reinforcement Learning Fine-tuning

## Purpose

Supervised fine-tuning teaches a model to produce outputs that match a training distribution. It does not directly optimize for what makes outputs *good*. Reinforcement learning fine-tuning closes this gap by aligning model behavior with human preferences, verifiable correctness signals, or learned reward functions. This stage is what transforms a capable base model into a model that is helpful, harmless, and honest — and more recently, into one that reasons through problems step-by-step.

The field has moved rapidly: RLHF via PPO (InstructGPT, 2022) established the paradigm; DPO (2023) simplified it dramatically; GRPO (DeepSeek-R1, 2025) enabled scalable reasoning training without a value function. The TRL library (HuggingFace) provides production-quality implementations of all major RL trainers.

## Architecture

**RLHF via PPO**

The classical three-stage pipeline:
1. **SFT**: supervised fine-tune on demonstration data
2. **Reward Model (RM)**: train a regression head to predict human preference scores from (prompt, response) pairs; typically initialized from SFT model
3. **PPO**: optimize the SFT policy to maximize RM score, subject to a KL divergence penalty against the SFT reference policy: `r = RM(x, y) - β·KL(π_θ || π_ref)`

The KL term prevents reward hacking — the policy cannot diverge too far from the SFT baseline. PPO requires running four models simultaneously (policy, reference, reward, value) — memory-intensive and complex to tune.

**DPO — Direct Preference Optimization**

Bypasses the reward model and PPO entirely. Given preference pairs `(y_w, y_l)` (winner, loser) for each prompt `x`, DPO optimizes a single objective:

```
L_DPO = -E[log σ(β · (log π_θ(y_w|x)/π_ref(y_w|x) - log π_θ(y_l|x)/π_ref(y_l|x)))]
```

This is equivalent to RLHF under the Bradley-Terry preference model but requires no reward model training and no PPO loop. Dataset format requires `prompt`, `chosen`, and `rejected` columns.

**GRPO — Group Relative Policy Optimization**

Introduced in DeepSeek-R1 for reasoning tasks. For each prompt, sample `G` completions, score each with a reward function `r_i`, then normalize within the group:

```
advantage_i = (r_i - mean(r)) / std(r)
```

Update the policy to increase the probability of high-advantage completions, with a clipped PPO-style objective. Key differences from PPO:
- No value function (uses group statistics instead) → simpler, lower memory
- No learned reward model — reward function is *verifiable* (unit tests for code, symbolic solvers for math, format checkers)
- Naturally handles reasoning chains: reward the final answer, not intermediate steps

**KTO — Kahneman-Tversky Optimization**

Requires only binary labels (thumbs up / thumbs down) rather than paired comparisons. Based on Kahneman-Tversky prospect theory: downweights losses more than equivalent gains. Easiest data collection of all RL methods — any existing user feedback signal can be used.

**TRL Trainers (HuggingFace):**
- `SFTTrainer` — instruction tuning with packing and LoRA support
- `RewardTrainer` — trains reward models from preference data
- `PPOTrainer` — full RLHF pipeline
- `DPOTrainer` — preference optimization without RM
- `GRPOTrainer` — group-relative policy optimization for reasoning

## Implementation Notes

**DPO with TRL:**
```python
from trl import DPOTrainer, DPOConfig

training_args = DPOConfig(
    beta=0.1,              # KL coefficient — lower = more aggressive RL
    max_length=1024,
    per_device_train_batch_size=4,
    gradient_accumulation_steps=4,
    learning_rate=5e-7,    # much lower than SFT
    num_train_epochs=1,
)

trainer = DPOTrainer(
    model=model,
    ref_model=ref_model,   # frozen SFT reference; or use implicit reference with peft
    args=training_args,
    train_dataset=dataset, # must have 'prompt', 'chosen', 'rejected' columns
)
trainer.train()
```

**GRPO for reasoning tasks (e.g., math):**
```python
from trl import GRPOTrainer, GRPOConfig

def reward_fn(completions, ground_truths):
    # verifiable reward: +1 if answer matches, 0 otherwise
    rewards = []
    for c, gt in zip(completions, ground_truths):
        answer = extract_answer(c)
        rewards.append(1.0 if answer == gt else 0.0)
    return rewards

# combine format reward + correctness reward
def combined_reward(completions, ground_truths):
    format_rewards = [1.0 if has_think_tags(c) else 0.0 for c in completions]
    correct_rewards = reward_fn(completions, ground_truths)
    return [0.3 * f + 0.7 * c for f, c in zip(format_rewards, correct_rewards)]
```

**Key hyperparameters:**
- `beta` (KL coefficient): 0.01–0.5; lower = more aggressive alignment, higher reward hacking risk
- GRPO group size `G`: 4–16; larger groups give more stable advantage estimates
- RL learning rate: 10–100× lower than SFT (1e-7 to 5e-6 range)
- Run for 1 epoch typically — RL stages overfit fast

**Reward hacking signals:**
- Reward increases sharply but outputs become incoherent or repetitive
- KL divergence from reference grows beyond 10–20 nats
- Monitor: RM score distribution, response length distribution, perplexity on held-out set

## Trade-offs

| Method | Data Required | Complexity | Use Case |
|---|---|---|---|
| PPO | Preference pairs + RM training | High (4 models) | Maximum control, custom RM |
| DPO | Preference pairs (chosen/rejected) | Low | Helpfulness/safety alignment |
| GRPO | Prompts + verifiable reward fn | Medium | Math, code, reasoning tasks |
| KTO | Binary labels (good/bad) | Very low | When paired data unavailable |

**PPO vs DPO:**
- PPO allows online data collection (model generates, human rates, update); DPO is offline only
- DPO is simpler to implement and more training-stable; PPO offers more flexibility in reward shaping
- In practice, DPO matches or exceeds PPO on most alignment benchmarks

**GRPO specifics:**
- Requires a *verifiable* reward signal — not suitable for open-ended creative tasks
- Extremely effective for reasoning: models learn to generate extended chain-of-thought as a side effect of optimizing answer correctness
- Can collapse to reward hacking if reward function has loopholes (e.g., always outputting "42" for math problems with that answer)

**DPO data quality:**
- Preference pairs with large quality gaps train more effectively than marginal distinctions
- Chosen responses should be genuinely good, not just better than rejected
- Noisy preference labels (where raters disagree) degrade DPO performance more than PPO

## References

- Ouyang et al. (2022) — *Training language models to follow instructions with human feedback* (InstructGPT, arXiv:2203.02155)
- Rafailov et al. (2023) — *Direct Preference Optimization* (arXiv:2305.18290)
- Shao et al. (2024) — *DeepSeekMath: Pushing the Limits of Mathematical Reasoning in Open Language Models* (GRPO introduction)
- DeepSeek-AI (2025) — *DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning*
- Ethayarajh et al. (2023) — *KTO: Model Alignment as Prospect Theoretic Optimization*
- TRL library: https://github.com/huggingface/trl

## Links
- [[finetuning_strategies|Fine-tuning Strategies]]
- [[peft_and_lora|PEFT and LoRA]]
- [[instruction_data_design|Instruction Data Design]]
- [[synthetic_data_generation|Synthetic Data Generation]]
- [[alignment_and_rlhf|Alignment and RLHF]]
- [[experiment_tracking|Experiment Tracking]]
- [[distributed_training|Distributed Training]]
