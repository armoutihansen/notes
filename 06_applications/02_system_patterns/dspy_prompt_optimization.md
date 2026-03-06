---
layer: 06_applications
type: application
status: seed
tags: [dspy, prompt-optimization, rag, agents, declarative-prompting]
created: 2026-03-06
---

# DSPy Prompt Optimization

## Purpose

Implementation patterns for using DSPy (Stanford NLP) to build declarative, optimizable LLM pipelines. Instead of hand-crafting prompt strings, DSPy separates the *structure* of a task (a `Signature`) from the *optimization* of how to perform it. Optimizers like `BootstrapFewShot` and `MIPRO` auto-generate few-shot demonstrations and instruction improvements from labelled data — typically improving task accuracy 10–30% over manual prompting.

### Examples

- Multi-hop question answering with automatic few-shot examples
- RAG pipeline with auto-tuned retrieval and generation prompts
- Agent with ReAct reasoning and data-driven optimization

## Architecture

**Installation and LM configuration:**
```python
pip install dspy
```

```python
import dspy

# Configure LM — any OpenAI-compatible, Anthropic, or local model
lm = dspy.LM("openai/gpt-4o-mini", api_key="sk-...")
# Or: dspy.LM("anthropic/claude-3-5-haiku-latest")
# Or: dspy.LM("ollama_chat/llama3.1", base_url="http://localhost:11434")
dspy.configure(lm=lm)
```

**Signatures — the core abstraction:**
```python
# Inline form (quick)
qa = dspy.Predict("question -> answer")

# Class form (recommended — add docstring and field constraints)
class SummarizeArticle(dspy.Signature):
    """Summarize a news article into 3–5 bullet points."""
    article: str = dspy.InputField()
    summary: str = dspy.OutputField(desc="Bullet points starting with '•'")

# Chain-of-Thought adds a reasoning field automatically
cot = dspy.ChainOfThought(SummarizeArticle)
result = cot(article="Full article text here...")
print(result.reasoning)  # intermediate reasoning
print(result.summary)    # final output
```

**Composing a multi-stage module:**
```python
class MultiHopQA(dspy.Module):
    def __init__(self):
        self.gen_query = dspy.ChainOfThought("question -> search_query")
        self.retrieve   = dspy.Retrieve(k=3)        # needs a retriever configured
        self.gen_answer = dspy.ChainOfThought("context, question -> answer")

    def forward(self, question: str) -> dspy.Prediction:
        query   = self.gen_query(question=question).search_query
        context = "\n".join(self.retrieve(query).passages)
        answer  = self.gen_answer(context=context, question=question).answer
        return dspy.Prediction(answer=answer)

qa = MultiHopQA()
print(qa(question="Who invented the transformer architecture?").answer)
```

**Optimization with BootstrapFewShot:**
```python
from dspy.teleprompt import BootstrapFewShot

# Labelled training examples (50–200 sufficient for BootstrapFewShot)
trainset = [
    dspy.Example(question="What is GQA?", answer="Grouped Query Attention...").with_inputs("question"),
    # ... more examples
]

def exact_match(example, pred, trace=None):
    return example.answer.lower() in pred.answer.lower()

optimizer = BootstrapFewShot(metric=exact_match, max_bootstrapped_demos=4)
optimized_qa = optimizer.compile(qa, trainset=trainset)
optimized_qa.save("models/multihop_qa_v1.json")
```

**Optimization with MIPRO (stronger, needs more data):**
```python
from dspy.teleprompt import MIPROv2

optimizer = MIPROv2(metric=exact_match, auto="medium")  # auto selects trials
optimized = optimizer.compile(
    qa,
    trainset=trainset,
    requires_permission_to_run=False
)
```

**Typed output with Pydantic:**
```python
from pydantic import BaseModel

class Entity(BaseModel):
    name: str
    type: str       # "person" | "org" | "location"
    relevance: float

class ExtractEntities(dspy.Signature):
    """Extract named entities from the text."""
    text: str = dspy.InputField()
    entities: list[Entity] = dspy.OutputField()

extractor = dspy.TypedPredictor(ExtractEntities)
result = extractor(text="DeepMind, a subsidiary of Alphabet, published Gemini...")
for e in result.entities:
    print(e.name, e.type)
```

**ReAct agent with tools:**
```python
def search_web(query: str) -> str:
    """Search the web for current information."""
    # integrate with Tavily, SerpAPI, etc.
    return search_results

class AnswerWithSearch(dspy.Signature):
    """Answer the question, using search if needed."""
    question: str = dspy.InputField()
    answer: str = dspy.OutputField()

agent = dspy.ReAct(AnswerWithSearch, tools=[search_web])
print(agent(question="Latest vLLM release version?").answer)
```

**Evaluation:**
```python
from dspy.evaluate import Evaluate

evaluator = Evaluate(devset=testset, metric=exact_match, num_threads=4)
print("Before:", evaluator(qa))
print("After: ", evaluator(optimized_qa))
```

**Saving and loading:**
```python
optimized_qa.save("models/qa_v1.json")
qa.load("models/qa_v1.json")
```

## Links
- [[dspy_systematic_prompting|DSPy Systematic Prompting]]
- [[rag_architecture|RAG Architecture]]
- [[prompting_strategies|Prompting Strategies]]
- [[structured_outputs|Structured Outputs]]
