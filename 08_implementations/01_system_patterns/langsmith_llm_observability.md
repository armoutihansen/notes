---
layer: 08_implementations
type: application
status: growing
tags: [pattern, evaluation, llmops]
created: 2026-03-06
---

# LangSmith LLM Observability

## Purpose

Implementation patterns for instrumenting, evaluating, and monitoring LLM applications with LangSmith. LangSmith traces the full request lifecycle — prompt inputs, LLM calls (model, tokens, latency), tool calls, retrieval steps — and surfaces aggregate quality metrics, cost tracking, and regression testing against curated datasets. It is the fastest path to observability for LangChain apps (zero code changes) and integrates cleanly with non-LangChain code via the `@traceable` decorator.

### Examples

- Zero-config tracing for a LangChain RAG pipeline
- Regression dataset built from production traces and run on every PR
- LLM-as-judge evaluation of response quality across model versions

## Architecture

**Installation and environment setup:**
```bash
pip install langsmith
export LANGSMITH_API_KEY="lsv2_..."
export LANGSMITH_TRACING=true
export LANGSMITH_PROJECT="my-app-prod"   # logical grouping of traces
```

**Zero-code LangChain tracing (env vars sufficient):**
```python
# No code changes needed — every LangChain/LangGraph call is automatically traced
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

llm   = ChatOpenAI(model="gpt-4o-mini")
chain = ChatPromptTemplate.from_template("Summarize: {text}") | llm

result = chain.invoke({"text": "Long article..."})
# → LangSmith shows full trace: prompt → LLM call (tokens, latency) → output
```

**Non-LangChain tracing with `@traceable`:**
```python
from langsmith import traceable

@traceable(name="retrieve-docs", run_type="retriever")
def retrieve(query: str) -> list[str]:
    # your retrieval logic — inputs/outputs captured automatically
    return vector_store.search(query)

@traceable(name="call-llm", run_type="llm")
def call_llm(prompt: str) -> str:
    response = openai_client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}]
    )
    return response.choices[0].message.content

@traceable(name="rag-pipeline")
def answer(question: str) -> str:
    docs    = retrieve(question)     # child span
    context = "\n".join(docs)
    return call_llm(f"Context:\n{context}\n\nQuestion: {question}")  # child span
```

**Attaching metadata per request (user, session, feature flags):**
```python
from langsmith import trace

with trace("handle-request", metadata={
    "user_id": uid,
    "session_id": sid,
    "feature": "product-search",
    "model_version": "v2.1"
}):
    result = answer(user_query)
```

**Adding user feedback signal:**
```python
from langsmith import Client
client = Client()

# After user clicks thumbs-up/down, log feedback against the run_id
client.create_feedback(
    run_id=run_id,           # captured from trace metadata or run context
    key="user_rating",
    score=1.0,               # 1=positive, 0=negative
    comment="Great answer"
)
```

**Creating an evaluation dataset from production traces:**
```python
from langsmith import Client

client = Client()

# Pull recent traces that got positive user feedback
runs = list(client.list_runs(
    project_name="my-app-prod",
    filter='and(eq(feedback_key, "user_rating"), eq(feedback_score, 1))',
    limit=200
))

# Create a curated regression dataset
dataset = client.create_dataset(name="regression-v1", description="Golden examples")
client.create_examples(
    inputs=[{"question": r.inputs["question"]} for r in runs],
    outputs=[{"answer": r.outputs["output"]}  for r in runs],
    dataset_id=dataset.id
)
```

**Running automated evaluations:**
```python
from langsmith.evaluation import evaluate, LangChainStringEvaluator

results = evaluate(
    lambda x: answer(x["question"]),   # your pipeline as a callable
    data="regression-v1",
    evaluators=[
        LangChainStringEvaluator("correctness"),   # LLM-as-judge (1–5 scale)
        LangChainStringEvaluator("conciseness"),
    ],
    experiment_prefix="gpt4o-mini-v2",
    num_repetitions=1
)
print(results.to_pandas().groupby("evaluator_name")["score"].mean())
```

**OpenTelemetry alternative (vendor-agnostic):**
```python
# For Grafana / Datadog / Jaeger stacks instead of LangSmith
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter

# Auto-instrument LangChain
from opentelemetry.instrumentation.langchain import LangchainInstrumentor
LangchainInstrumentor().instrument()
```

**Sampling strategy for production:**

| Phase | Sampling | Rationale |
|---|---|---|
| Development | 100% | Full visibility for debugging |
| Staging | 100% | Catch regressions before prod |
| Production (early) | 100% | Build baseline dataset |
| Production (scaled) | 10% + all errors | Control storage cost |

**Key metrics to track in LangSmith dashboards:**
- TTFT (time to first token): alert at p95 > 500 ms
- Total latency: track p50/p95 weekly trend
- Input/output token counts: sudden spikes = prompt injection or runaway loops
- Tool call success rate: < 95% warrants investigation
- User rating ratio: baseline + weekly change detection

## References

- [LangSmith Documentation](https://docs.smith.langchain.com/)

## Links
- [[llm_observability|LLM Observability]]
- [[ai_application_architecture|AI Application Architecture]]
- [[ml_observability|ML Observability]]
- [[langsmith-observability (skill)|LangSmith Observability Skill]]
