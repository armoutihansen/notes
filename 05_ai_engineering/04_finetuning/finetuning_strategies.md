---
layer: 05_ai_engineering
type: engineering
tool: general
status: growing
tags: [fine-tuning, llm, peft, rlhf, domain-adaptation]
created: 2026-03-05
---

# Fine-tuning Strategies

## Purpose

Adapting pre-trained foundation models to specific tasks or domains is one of the most high-leverage activities in applied AI engineering. Rather than training models from scratch — which requires billions of tokens and enormous compute — fine-tuning starts from a powerful base and redirects its capabilities toward a target distribution: a specific domain vocabulary, a response style, a task format, or a set of safety constraints.

Fine-tuning is not always the right answer. The first decision is **RAG vs fine-tune**:

| Criterion | Prefer RAG | Prefer Fine-tuning |
|---|---|---|
| Knowledge freshness | Dynamic, frequently updated | Static domain knowledge |
| Citations needed | Yes | No |
| Domain data availability | Limited labeled pairs | Hundreds–thousands of examples |
| Response format/style | Flexible | Critical, highly specific |
| Domain vocabulary | General | Dense, specialized |
| Latency budget | Tolerant of retrieval overhead | Latency-critical path |

These approaches are **complementary, not exclusive** — a fine-tuned model with RAG is a common production pattern. Fine-tuning teaches the model *how* to respond; RAG supplies *what* to respond with.

## Architecture

Four major fine-tuning paradigms exist, ordered roughly by compute cost and data requirements:

**1. Continued Pre-training**
Domain adaptation on raw (unlabeled) text before any instruction tuning. Feed the model large corpora in the target domain (e.g., legal filings, clinical notes, scientific papers) to shift the base distribution. Useful when the domain vocabulary or reasoning patterns are far from the pre-training distribution. Requires gigabytes of domain text; typically run on the base model before SFT. See [[peft_and_lora|PEFT and LoRA]] for compute-efficient variants.

**2. Instruction Tuning (SFT)**
Supervised fine-tuning on (instruction, response) pairs to teach instruction-following behavior. Uses cross-entropy loss on the response tokens. Data formats: Alpaca (single-turn `instruction/input/output`), ChatML / ShareGPT (multi-turn with role tags). See [[instruction_data_design|Instruction Data Design]] for data considerations.

**3. PEFT — Parameter-Efficient Fine-tuning**
Update fewer than 1% of model parameters using adapter methods (LoRA, QLoRA, prefix tuning, adapter layers). Dramatically reduces GPU memory and training time while preserving most of the performance of full fine-tuning. The standard approach for most practitioners. Detailed in [[peft_and_lora|PEFT and LoRA]].

**4. Full Fine-tuning**
Update all model weights using a low learning rate (~2e-5). Maximum expressiveness but requires significant GPU memory (e.g., ~80GB+ for a 7B model in BF16 without ZeRO). Risk of catastrophic forgetting — the model loses general capabilities as it over-specializes. Mitigated by mixing general data into the fine-tuning set (data replay).

**RL Alignment (RLHF/DPO/GRPO)**
Post-SFT alignment stage: optimize model behavior against human preference signals. Covered in [[rl_finetuning|Reinforcement Learning Fine-tuning]].

## Implementation Notes

**Data scale guidance:**
- Fine-tuning requires hundreds to thousands of high-quality examples, not millions (that's pre-training scale)
- Data quality >> quantity: 1,000 carefully curated pairs routinely outperform 100,000 noisy ones
- Target diversity across task types, lengths, and difficulty levels — see [[instruction_data_design|Instruction Data Design]]

**Hyperparameter starting points:**
- LoRA learning rate: ~1e-4; full fine-tune learning rate: ~2e-5
- Epochs: 1–3; beyond 3 the model tends to overfit and lose generality
- Warmup ratio: 0.03 is a safe default
- Weight decay: 0.01

**Evaluation protocol:**
1. Task-specific held-out set (primary metric)
2. General benchmark suite (MMLU, HellaSwag, GSM8K) to check for catastrophic forgetting
3. MT-Bench or LLM-as-judge for open-ended response quality
4. Log loss curves — training loss should decrease smoothly; a flat eval loss with rising training loss signals overfitting

**Tooling:**
- **Axolotl**: YAML-based, wraps HuggingFace Trainer, supports full/LoRA/QLoRA, 100+ architectures
- **LLaMA-Factory**: WebUI + CLI, multimodal support, DPO/GRPO built-in
- **HuggingFace PEFT + TRL**: lower-level, more flexible for custom workflows

## Trade-offs

| Strategy | Performance | GPU Cost | Forgetting Risk | Data Needed |
|---|---|---|---|---|
| Full fine-tuning | Highest | Very high | High | Moderate |
| LoRA | Near-full | Low | Low | Moderate |
| QLoRA | Slightly below LoRA | Very low | Low | Moderate |
| Continued pre-training | Domain +, general ~ | Medium | Low | Large (raw text) |
| Instruction tuning only | Task-specific | Low–medium | Medium | Low–medium |

Key design tensions:
- **Specialization vs. generality**: aggressive fine-tuning improves task performance but risks narrowing the model's useful range — data replay and gentle LR schedules mitigate this
- **Fine-tune vs. RAG**: fine-tuning bakes knowledge into weights (fast inference, no retrieval infra, but stale); RAG externalizes knowledge (updatable, citable, but adds latency and infra complexity)
- **Adapter merging**: LoRA adapters can be merged into base weights at inference time (zero latency cost) or kept separate (hot-swappable but extra compute)

## References

- Hu et al. (2021) — *LoRA: Low-Rank Adaptation of Large Language Models*
- Dettmers et al. (2023) — *QLoRA: Efficient Finetuning of Quantized LLMs*
- Wei et al. (2021) — *Finetuned Language Models Are Zero-Shot Learners* (FLAN)
- Ouyang et al. (2022) — *Training language models to follow instructions with human feedback* (InstructGPT)
- Axolotl: https://github.com/OpenAccess-AI-Collective/axolotl
- LLaMA-Factory: https://github.com/hiyouga/LLaMA-Factory

## Links
- [[peft_and_lora|PEFT and LoRA]]
- [[rl_finetuning|Reinforcement Learning Fine-tuning]]
- [[instruction_data_design|Instruction Data Design]]
- [[synthetic_data_generation|Synthetic Data Generation]]
