---
layer: 05_ai_engineering
type: engineering
tool: dspy
status: evergreen
tags: [dspy, prompt-optimization, few-shot, pipeline, llm, systematic]
created: 2026-03-05
---

# DSPy — Systematic Prompt Optimization

## Purpose

DSPy is a programmatic framework for building and optimizing LLM pipelines without manual prompt engineering. Rather than hand-crafting prompts and few-shot examples, DSPy treats prompt construction as a compilation problem: the developer defines the task declaratively (input fields, output fields, quality metric), and an optimizer automatically discovers the best prompts and demonstrations by searching over the space of possible configurations. This eliminates the guesswork and brittleness of manual prompt engineering, especially for complex multi-hop pipelines where prompt changes in one module can interact unpredictably with downstream modules.

## Architecture

### DSPy Signatures
A `Signature` declares the input and output fields of a module, with descriptions that serve as the prompt template. No prompt text is written manually.

```python
class QuestionAnswer(dspy.Signature):
    """Answer a factual question accurately."""
    question = dspy.InputField(desc="A factual question to be answered")
    answer = dspy.OutputField(desc="A concise, accurate answer to the question")
```

The signature's docstring and field descriptions become the foundation of the generated prompt. The optimizer's job is to enrich this with few-shot examples and, optionally, refined instructions.

### DSPy Modules
Modules wrap signatures and implement a forward pass. Built-in modules:
- **`dspy.Predict`**: Direct call to the LLM with the signature. Equivalent to zero-shot or few-shot prompting.
- **`dspy.ChainOfThought`**: Automatically adds a `reasoning` field before the output field, eliciting chain-of-thought reasoning.
- **`dspy.ReAct`**: Implements the ReAct loop (see [[agentic_loop|Agentic Loop]]) with tool use, wrapping tool definitions into the signature framework.
- **`dspy.MultiChainComparison`**: Generates multiple CoT reasoning chains and compares them before producing the final answer (like Self-Consistency).
- **`dspy.ProgramOfThought`**: Generates and executes Python code to answer the question.

Modules compose naturally into programs:
```python
class RAGPipeline(dspy.Module):
    def __init__(self, num_passages=3):
        self.retrieve = dspy.Retrieve(k=num_passages)
        self.generate = dspy.ChainOfThought("context, question -> answer")

    def forward(self, question):
        context = self.retrieve(question).passages
        return self.generate(context=context, question=question)
```

### DSPy Optimizers (Teleprompters)
Optimizers take a program, a training set, and a metric function, then automatically construct an optimized version of the program with better prompts and/or few-shot examples.

- **`BootstrapFewShot`**: The simplest optimizer. Runs the teacher program on training examples, filters for correct outputs, and selects the best demonstrations as few-shot examples. No LLM calls for optimization itself (beyond the training set forward passes).
- **`BootstrapFewShotWithRandomSearch`**: Extends BootstrapFewShot with random search over demonstration selection. More robust selection.
- **`BayesianSignatureOptimizer` / `MIPRO`**: Uses a meta-LLM to propose and evaluate new instruction phrasings for each module's signature. Jointly optimizes instructions and demonstrations. Best quality; requires more optimization budget.
- **`BootstrapFinetune`**: Instead of in-context demonstrations, generates training data from the bootstrap process and fine-tunes the underlying model.

### Compilation Flow
```
DSPy Program (modules with signatures)
    + Training set (input/output pairs)
    + Metric function (correctness check)
    ↓
Optimizer.compile()
    → Run teacher on trainset
    → Evaluate with metric
    → Select best demonstrations / refine instructions
    ↓
Compiled program (with optimized prompts baked in)
    → Deploy to production
```

## Implementation Notes

**Minimal working example**
```python
import dspy

# Configure the LLM
lm = dspy.OpenAI(model="gpt-4o-mini", max_tokens=1000)
dspy.settings.configure(lm=lm)

# Define signature
class Classify(dspy.Signature):
    """Classify the sentiment of the given text."""
    text = dspy.InputField()
    sentiment = dspy.OutputField(desc="one of: positive, negative, neutral")

# Create module
classifier = dspy.Predict(Classify)

# Direct inference (no optimization yet)
result = classifier(text="This product is fantastic!")
print(result.sentiment)  # "positive"
```

**Optimization with BootstrapFewShot**
```python
from dspy.teleprompt import BootstrapFewShot

# Define metric
def sentiment_metric(example, pred, trace=None):
    return example.sentiment.lower() == pred.sentiment.lower()

# Prepare training set
trainset = [dspy.Example(text=t, sentiment=s).with_inputs("text")
            for t, s in training_data]

# Compile
optimizer = BootstrapFewShot(metric=sentiment_metric, max_bootstrapped_demos=4)
compiled_classifier = optimizer.compile(classifier, trainset=trainset)

# The compiled_classifier now has optimized few-shot examples in its prompt
```

**Multi-model backend support**
DSPy works with any LLM via its LM adapter layer: `dspy.OpenAI`, `dspy.Anthropic`, `dspy.HFModel` (HuggingFace), `dspy.Ollama`. Swap backends without changing the program code.

**Systematic ablation without manual prompt tweaking**
DSPy enables principled experimentation: change the module type (Predict vs ChainOfThought), change the training set, change the metric, and re-compile. Each configuration is reproducible and comparable.

**When to use DSPy vs manual prompting**
- Use DSPy when: the pipeline has multiple chained modules; few-shot example selection is laborious; you have a labeled dataset (even 20–50 examples); you need to systematically compare prompt variants.
- Use manual prompting when: the task is simple (single module); you have no labeled data; you need immediate results without optimization overhead.

**Integration with RAG**
`dspy.Retrieve` integrates with a configurable retriever backend (ColBERT, OpenAI embeddings, custom). Combined with `ChainOfThought`, this enables an automatically optimized RAG pipeline — the optimizer can learn which retrieval patterns and reasoning styles produce the most faithful answers. See [[rag_architecture|RAG Architecture]].

## Trade-offs

**Optimization requires labeled data**: Even BootstrapFewShot (the cheapest optimizer) needs a training set. For novel tasks without any labels, DSPy's optimizers cannot run. Cold-start problem: begin with zero-shot or manual few-shot, collect signal, then optimize.

**Abstraction overhead**: DSPy adds a compilation layer that can obscure what prompts are actually being sent to the LLM. Use `dspy.inspect_history()` to examine the compiled prompts. The abstraction is worth it for complex pipelines; may be overkill for single-module use cases.

**Optimizer LLM cost**: MIPRO and BayesianSignatureOptimizer use a meta-LLM to generate instruction candidates — this incurs additional API cost during optimization (typically 10–50× the cost of a single forward pass). BootstrapFewShot has no meta-LLM cost. Treat optimization as a one-time expense amortized over production inference.

**Best for complex multi-hop pipelines**: DSPy's value scales with pipeline complexity. A single-step classifier sees modest gains from optimization; a multi-hop QA system with retrieval, reasoning, and verification sees substantial gains because the optimizer can propagate signal across all modules simultaneously.

## References

- Khattab et al. (2023). *DSPy: Compiling Declarative Language Model Calls into Self-Improving Pipelines*. arXiv:2310.03714.
- Khattab et al. (2024). *Optimizing Instructions and Demonstrations for Multi-Stage Language Model Programs (MIPRO)*. arXiv:2406.11695.
- DSPy documentation: https://dspy-docs.vercel.app
- DSPy GitHub: https://github.com/stanfordnlp/dspy

## Links
- [[prompting_strategies|Prompting Strategies]]
- [[rag_architecture|RAG Architecture]]
- [[agentic_loop|Agentic Loop]]
- [[structured_outputs|Structured Outputs]]
