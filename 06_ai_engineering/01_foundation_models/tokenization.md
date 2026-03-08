---
layer: 06_ai_engineering
type: engineering
tool: huggingface-tokenizers
status: growing
tags: [nlp, algorithm]
created: 2026-03-05
---

# Tokenization

## Purpose

Tokenization is the process of converting raw text into a sequence of integer **token IDs** that a language model can process, and converting predicted token IDs back into text at inference time. It sits at the boundary between human-readable text and model computation, and its design has significant downstream effects on model capability, cost, and behaviour.

A tokenizer defines a **vocabulary** — a fixed set of tokens (subword units, characters, or bytes) — and a **merging algorithm** that determines how text is segmented. Because models are trained with a specific tokenizer, the tokenizer is a fixed dependency of every model checkpoint: using a mismatched tokenizer produces garbage outputs.

From an engineering standpoint, tokenization affects: (1) input/output cost (tokens are the billing unit for API calls); (2) context utilisation (how many "characters of meaning" fit in a 128K window); (3) model capability for specific domains (code, numbers, non-English text).

## Architecture

### Byte Pair Encoding (BPE)

Originally a data compression algorithm, adapted for NLP (Sennrich et al., 2016):

1. Start with a vocabulary of individual bytes or Unicode characters.
2. Count all adjacent pair frequencies in the training corpus.
3. Merge the most frequent pair into a new token; add to vocabulary.
4. Repeat until vocabulary size is reached (e.g., 50K merges for GPT-2's 50,257-token vocab).

BPE is greedy and deterministic. The merge rules are serialised alongside the vocabulary and applied at inference time. **Byte-level BPE** (GPT-2, GPT-4, Llama) operates at the byte level, providing complete Unicode coverage without an `[UNK]` token — every possible string can be tokenized.

### WordPiece

Used by BERT and its variants. Similar to BPE but the merge criterion maximises the **likelihood of the training corpus** under a language model rather than raw pair frequency:

$$
\text{score}(A, B) = \frac{\text{freq}(A, B)}{\text{freq}(A) \times \text{freq}(B)}
$$

WordPiece uses a `##` prefix to mark subword continuations (e.g., `playing` → `play`, `##ing`). This makes token boundaries more linguistically interpretable but couples the tokenizer to a specific LM objective.

### SentencePiece

Language-agnostic tokenizer (Kudo & Richardson, 2018) that operates directly on raw Unicode text without pre-tokenization (no language-specific whitespace splitting). Supports both BPE and **Unigram** algorithm:

- **Unigram**: starts with a large vocabulary and iteratively removes tokens that minimally degrade corpus likelihood. Produces probabilistic segmentations; useful for data augmentation.
- SentencePiece treats spaces as characters (`▁` prefix); handles CJK, Arabic, and other scripts uniformly.
- Used by: T5, LLaMA (1/2), Mistral, Gemma, many multilingual models.

### Tiktoken (OpenAI)

OpenAI's production tokenizer; BPE with a Rust backend. Notably fast (10× faster than HuggingFace slow tokenizers). Vocabularies: `cl100k_base` (100,277 tokens; GPT-4, text-embedding-ada-002), `o200k_base` (200,019 tokens; GPT-4o). Available via the `tiktoken` Python package.

### Vocabulary Sizes

| Tokenizer | Vocab Size | Model Family |
|---|---|---|
| GPT-2 | 50,257 | GPT-2 |
| SentencePiece (LLaMA 2) | 32,000 | LLaMA 2, Mistral |
| SentencePiece (LLaMA 3) | 128,256 | LLaMA 3 |
| cl100k_base | 100,277 | GPT-4 |
| o200k_base | 200,019 | GPT-4o |

### Special Tokens

Every tokenizer defines reserved special tokens used by the model's training regime:

| Token | Purpose |
|---|---|
| `<BOS>` / `<s>` | Beginning of sequence |
| `<EOS>` / `</s>` | End of sequence; stop generation |
| `<PAD>` | Padding to uniform batch length |
| `<UNK>` | Unknown token (absent in byte-level BPE) |
| `[INST]`, `[/INST]` | Instruction delimiters (Llama 2 chat) |
| `<\|im_start\|>`, `<\|im_end\|>` | ChatML format (GPT-4 API, Qwen) |

Incorrectly applying chat templates (or omitting them) is one of the most common sources of degraded fine-tuned model performance.

## Implementation Notes

### HuggingFace Tokenizers Library

The `tokenizers` library provides a Rust-backed fast tokenizer framework used by all `transformers` models:

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("meta-llama/Meta-Llama-3-8B-Instruct")

# Encode text
tokens = tokenizer("Hello, world!", return_tensors="pt")
# {'input_ids': tensor([[128000,  9906,    11,   1917,     0]]), ...}

# Apply chat template
messages = [{"role": "user", "content": "What is 2+2?"}]
formatted = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)

# Count tokens (for cost estimation)
n_tokens = len(tokenizer.encode("some text"))
```

Training a custom tokenizer on domain text (code, medical records, a new language):

```python
from tokenizers import ByteLevelBPETokenizer

tokenizer = ByteLevelBPETokenizer()
tokenizer.train(files=["corpus.txt"], vocab_size=32000, min_frequency=2,
                special_tokens=["<s>", "</s>", "<unk>", "<pad>"])
tokenizer.save_model("custom_tokenizer/")
```

### Tokenizer Mismatch Issues

Fine-tuning a model with a different tokenizer than the base model was pre-trained with causes **embedding dimension mismatches** and undefined behaviour for token IDs outside the original vocabulary. Always use the tokenizer shipped with the model checkpoint. If adding new special tokens (e.g., `<|tool_call|>`), use `tokenizer.add_special_tokens()` and then `model.resize_token_embeddings(len(tokenizer))`.

### Token Counting for Cost Estimation

API pricing is per-token. Estimate costs before deployment:

```python
import tiktoken
enc = tiktoken.encoding_for_model("gpt-4o")
n = len(enc.encode(document_text))
cost = n * 5e-6  # $5 per 1M input tokens (gpt-4o as of 2024)
```

### Subword Tokenization Implications

- **Code**: identifiers like `transformers` may tokenize as `transform` + `ers`; camelCase splits at case boundaries; indentation spaces are significant and tokenize non-uniformly across languages.
- **Numbers**: `12345` may tokenize as `123` + `45` or `1` + `2` + `3` + `4` + `5`, making arithmetic unreliable. Models with dedicated number tokenization (e.g., Llama 3's larger vocabulary) handle this better.
- **Multilingual text**: smaller vocabularies with BPE trained on English-dominant corpora may encode Chinese or Arabic characters as many individual byte tokens (2–4× higher token count than English at equivalent content), inflating cost and wasting context.

## Trade-offs

**Larger vocabulary**: fewer tokens per input sequence (more content per context window; lower inference cost); larger embedding table (more parameters, more memory); may have rare tokens that are under-represented in training.

**Smaller vocabulary**: more universal but longer sequences for the same content; better coverage of token-frequency distribution.

**SentencePiece**: excellent for multilingual models; language-agnostic training; Unigram supports stochastic segmentation for data augmentation. Less efficient than tiktoken for English-only workloads.

**Byte-level BPE**: handles all Unicode without `[UNK]`; slightly less efficient than character-level BPE for some languages; used by all major English-centric models.

## References

- Sennrich et al. (2016). *Neural Machine Translation of Rare Words with Subword Units* (BPE). ACL.
- Kudo & Richardson (2018). *SentencePiece: A simple and language independent subword tokenizer and detokenizer*. EMNLP.
- Schuster & Nakamura (2012). *Japanese and Korean Voice Search* (WordPiece). ICASSP.
- HuggingFace. *Tokenizers documentation*. huggingface.co/docs/tokenizers.
- OpenAI. *Tiktoken*. github.com/openai/tiktoken.

## Links

- [[foundation_model_overview|Foundation Model Overview]]
- [[alignment_and_rlhf|Alignment and RLHF]]
