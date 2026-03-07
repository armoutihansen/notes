---
layer: 06_ai_engineering
type: engineering
tool: langsmith
status: growing
tags: [monitoring]
created: 2026-03-05
---
# LLM Observability

## Purpose
Monitoring, debugging, and evaluating LLM applications in production. LLM systems have failure modes that traditional software monitoring misses: degraded output quality with no exception thrown, subtle prompt regressions across model updates, non-deterministic behavior making reproduction hard, and cost that scales unexpectedly with usage patterns. LLM observability provides structured visibility into the full request lifecycle — from user input through retrieval, tool calls, LLM calls, and final output — along with aggregate metrics and quality evaluation at scale.

## Architecture
**Observability dimensions for LLM systems:**

**(a) Tracing:**
Full request trace with all sub-steps: LLM calls (model, input tokens, output tokens, latency per call), tool calls (name, input, output, latency), retrieval steps (query, retrieved docs, similarity scores), chained prompt transformations. Each step gets a span with timing and metadata. Critical for debugging multi-step agent failures where the error is buried three tool calls deep.

**(b) Evaluation:**
Automated quality metrics run over samples of production traffic. Common evaluators: exact match (structured output tasks), LLM-as-judge (correctness, helpfulness, groundedness scored by a stronger model), embedding similarity (semantic similarity to reference answers), task-specific scorers (code execution pass rate, SQL validity). Evaluation against a curated regression dataset catches prompt/model regressions before users do.

**(c) Cost monitoring:**
Token usage × price per model per API call, aggregated by user, session, feature, or product surface. Models differ 100x in price (GPT-4o vs GPT-4o-mini vs Claude Haiku). Cost spikes often indicate prompt injection attacks or runaway agent loops.

**(d) Performance:**
Latency distributions (TTFT, total latency, p50/p95/p99), error rates by model/provider, tool call failure rates, cache hit rates (semantic cache). Alert on p95 latency degradation before users notice.

**Key metrics to track:**
| Metric | Target (typical chatbot) |
|--------|--------------------------|
| TTFT (time to first token) | <200ms p95 |
| Total latency | <3s p95 |
| Tokens in | Monitor for prompt injection patterns |
| Tokens out | Alert on anomalous long outputs |
| Tool call success rate | >95% |
| Retrieval hit rate | >80% (RAG systems) |
| User thumbs up rate | Baseline + regression detection |
| Task completion rate | Business-specific SLO |

**LangSmith:**
Automatic tracing for all LangChain, LangGraph, and LlamaIndex operations via environment variable injection — zero code changes for LangChain apps. For non-LangChain code, the `@traceable` decorator instruments any Python function. Provides: run explorer with full trace trees, dataset management for regression testing, evaluator execution against datasets, annotation queues for human feedback labeling, dashboards for aggregate metrics, playground for prompt iteration on logged traces.

**OpenTelemetry alternative:**
For vendor-agnostic observability, emit spans via OpenTelemetry to any backend (Jaeger, Grafana Tempo, Datadog). The `opentelemetry-instrumentation-langchain` package auto-instruments LangChain. More work to set up but avoids vendor lock-in for organizations with existing OTel infrastructure.

## Implementation Notes
**LangSmith setup (LangChain apps — zero code changes):**
```bash
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=lsv2_...
export LANGCHAIN_PROJECT=my-project  # logical grouping of traces
```

**LangSmith for non-LangChain code:**
```python
from langsmith import traceable, Client

@traceable(name="retrieve-docs", metadata={"version": "v2"})
def retrieve(query: str) -> list[str]:
    # Your retrieval logic
    return docs

@traceable(run_type="llm")
def call_llm(messages: list) -> str:
    response = openai_client.chat.completions.create(
        model="gpt-4o-mini", messages=messages
    )
    return response.choices[0].message.content
```

**Log custom metadata per request:**
```python
from langsmith import trace
with trace("handle-request", metadata={"user_id": uid, "session_id": sid, "feature": "chat"}):
    result = my_pipeline(user_input)
```

**Create evaluation dataset from production traces:**
```python
client = Client()
# Pull traces matching a filter, then curate into a dataset
dataset = client.create_dataset("regression-v1")
client.create_examples(inputs=[...], outputs=[...], dataset_id=dataset.id)
```

**Run evaluations:**
```python
from langsmith.evaluation import evaluate, LangChainStringEvaluator

results = evaluate(
    predict_fn,  # your pipeline as a callable
    data="regression-v1",
    evaluators=[
        LangChainStringEvaluator("correctness"),  # LLM-as-judge
        LangChainStringEvaluator("conciseness"),
    ],
    experiment_prefix="gpt4o-mini-baseline",
)
```

**Sampling strategy:**
- Development and staging: 100% trace logging
- Production (early): 100% to build baseline dataset
- Production (at scale): Sample 10% for storage cost management; always log all errors and low-confidence outputs (e.g., user negative feedback, tool call failures)
- Use reservoir sampling to ensure long-tail coverage

## Trade-offs
**Full tracing vs sampling:** Full tracing gives complete visibility and enables comprehensive regression testing but incurs significant storage costs at scale (a 10-step RAG trace may be 50–200 KB; 1M daily requests = 50–200 GB/day). Sampling at 10% reduces cost 10x but misses tail events. Compromise: full trace metadata (latency, token counts, error flag) at 100%; full payload logging at 10% + all errors.

**LangSmith vs custom OTel:** LangSmith is the fastest path for LangChain apps — zero code changes, rich UI, built-in evaluators. Cost is vendor lock-in and per-run pricing at scale. OpenTelemetry is more work initially but integrates with existing enterprise observability stacks (Datadog, Grafana) and avoids additional SaaS spend.

**LLM-as-judge evaluators:** Powerful for open-ended quality assessment but introduce their own error rate (~5–10%) and latency/cost overhead. Use for evaluation datasets and sampling, not for real-time serving latency SLOs.

**Annotation queues:** Human feedback is the gold standard for quality measurement but expensive. Focus annotation effort on examples where automated evaluators disagree, low-confidence model outputs, and user-reported failures. 100–500 human-labeled examples per evaluation dimension is typically sufficient for a reliable evaluation signal.

## References
- LangSmith docs: https://docs.smith.langchain.com
- OpenTelemetry for LLMs: https://opentelemetry.io/blog/2024/llm-observability/
- Shankar et al. (2024). "Who Validates the Validators? Aligning LLM-Assisted Evaluation of LLM Outputs with Human Preferences." (LLM-as-judge reliability analysis)

## Links
- [[ai_application_architecture|AI Application Architecture]]
- [[serving_frameworks|LLM Serving Frameworks]]
- [[ml_observability|ML Observability]]
