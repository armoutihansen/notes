---
layer: 05_ai_engineering
type: ai_system
status: growing
tags: [architecture, system-design, llm, model-gateway, data-flywheel, production]
created: 2026-03-05
---
# AI Application Architecture

## Goal
Design principles and patterns for building reliable, maintainable AI applications around foundation models. Foundation models are powerful but non-deterministic, expensive, and under continuous evolution — the system surrounding the model largely determines production quality. Good architecture decouples the application from any specific model provider, creates a data flywheel for continuous improvement, and manages context deliberately rather than naively concatenating strings.

## Architecture
**Layer model:**
```
User
  └─ Frontend / API Gateway
      └─ Orchestration Layer (LangChain / LlamaIndex / DSPy / custom)
          ├─ Model Gateway (routing, auth, rate limiting, semantic cache)
          │   ├─ Primary LLM provider (OpenAI, Anthropic, Google)
          │   └─ Fallback LLM provider
          ├─ Tool Registry (search, code execution, external APIs)
          ├─ Vector Store (retrieval, memory)
          └─ Observability (LangSmith / OpenTelemetry)
```

**Context management:**
The model's context window is the most important resource to manage explicitly. Components of a typical context:
1. **System prompt** — persona, capabilities, constraints, output format instructions
2. **Conversation history** — compressed using summarization or sliding window when approaching context limits
3. **Retrieved context** — documents from RAG, ranked by relevance, truncated to fit
4. **Tool results** — structured outputs from tool calls, potentially large (e.g., code execution output)
5. **Current user turn** — the immediate input

Context overflow is a hard failure; context bloat increases latency and cost linearly. Budget each component explicitly: e.g., system=1k, history=4k, retrieved=6k, tools=2k, user=1k within a 16k context.

**Model gateway pattern:**
An internal proxy between the orchestration layer and LLM providers abstracts provider APIs, enforces auth/rate limiting, implements semantic caching, and enables routing logic. Key behaviors:
- *Semantic caching:* Cache responses for queries semantically similar to previously seen queries (embed query, vector search cache, serve if similarity >threshold). Can reduce costs 30–80% for FAQ-heavy workloads. LiteLLM + Redis provides this out of the box.
- *Fallback routing:* Primary → fallback model on error (500, rate limit, timeout). E.g., GPT-4o primary, GPT-4o-mini fallback; or Anthropic primary, OpenAI fallback.
- *Cost-aware routing:* Route simple queries to cheap models (GPT-4o-mini, Claude Haiku), complex queries to capable models (GPT-4o, Claude 3.5 Sonnet). Classifier determines query complexity tier.

**Data flywheel:**
Production queries → sample and annotate (human labels or LLM-as-judge) → evaluation dataset → measure regressions with each prompt/model change → fine-tune or improve prompt → redeploy → repeat. The flywheel accelerates over time: more production data → better evaluation coverage → faster, more confident iteration.

## Components
| Component | Purpose | Tools |
|-----------|---------|-------|
| Model gateway | Provider abstraction, caching, routing | LiteLLM, custom proxy |
| Orchestration | Multi-step reasoning, tool use, RAG | LangChain, LlamaIndex, DSPy |
| Vector store | Semantic retrieval, memory | Chroma (dev), Pinecone/Weaviate/pgvector (prod) |
| Tool registry | External capabilities (search, code, APIs) | LangChain tools, custom functions |
| Observability | Tracing, evaluation, cost monitoring | LangSmith, OpenTelemetry + Grafana |
| Human feedback | Preference signal for improvement loop | Thumbs up/down, correction interface |
| A/B testing | Compare prompts/models on live traffic | Feature flags (LaunchDarkly), custom |
| Cache | Semantic + exact match response cache | Redis + LiteLLM, GPTCache |

**Orchestration framework selection:**
- *LangChain:* Broad ecosystem, best observability integration (LangSmith), good for RAG and agents. High abstraction can obscure behavior.
- *LlamaIndex:* Stronger data/retrieval primitives; best for complex RAG (multi-document, hierarchical indices).
- *DSPy:* Systematic prompt optimization via compilation; best when prompts are complex and tunable.
- *Custom/minimal:* Prefer for simple chains where framework overhead is unjustified. Often the right call for production-critical paths.

## Evaluation
**Latency SLOs:**
- TTFT: p50 <100ms, p95 <300ms (with streaming; users tolerate latency better when tokens start flowing)
- Total latency: p50 <1.5s, p95 <3s for typical chat; p95 <10s for complex agent tasks
- Tool call overhead: instrument and budget individually

**Quality metrics on sampled production traffic:**
- Correctness (LLM-as-judge, 1–5 scale): track weekly trend
- Groundedness (for RAG): proportion of response claims supported by retrieved context
- Refusal rate: proportion of valid requests incorrectly refused by safety systems
- User satisfaction: thumbs up/(thumbs up + thumbs down) ratio

**A/B testing:**
Route 5–10% of production traffic to new prompt/model variant. Measure: user satisfaction signal (if available), task completion, session length, cost per session. Minimum detectable effect ~2% requires ~5k sessions per variant for adequate power.

**Cost per conversation:**
Track as p50/p95 to catch runaway agent loops. Alert threshold: p99 > 10× p50 suggests pathological behavior.

## Failure Modes
**Prompt injection:** Malicious content in tool outputs, retrieved documents, or user input attempts to override the system prompt or hijack tool calls. Mitigations: (1) separate user/tool content from instructions via clear delimiters and role enforcement; (2) classify tool outputs through LlamaGuard before including in context; (3) sandbox tool execution; (4) monitor for instruction-pattern tokens in retrieved content.

**LLM provider outage:** Unmitigated, a primary provider outage causes total product failure. Mitigation: fallback routing in the model gateway, with automatic failover on 5xx/timeout. Test quarterly. LiteLLM supports this natively.

**Context window overflow:** Application fails or silently truncates context when inputs exceed the model's context limit, causing incoherent outputs. Mitigation: explicit context budgeting, conversation summarization before window boundary, early warning metric (log when context >80% of limit).

**Hallucination in high-stakes decisions:** Model confidently asserts false information. Mitigation: RAG with groundedness checking, structured output with validation, human review for high-stakes outputs, uncertainty quantification (sample multiple completions, flag high variance).

**Feedback loop amplification:** Model fine-tuned on its own outputs can amplify biases and reduce diversity. Mitigation: always include a ground-truth anchor (verified examples from humans), monitor output diversity metrics, never fine-tune exclusively on model-generated data.

**Tool call error cascades:** One failed tool call causes the agent to hallucinate downstream results rather than retry or escalate. Mitigation: explicit error handling in tool definitions, retry logic with backoff, agent should report uncertainty rather than fabricate results.

## Cost / Latency
Input token cost >> output token cost for most providers (~3–5x). Long system prompts and large retrieved contexts dominate costs in RAG applications. Cache aggressively:
- *Exact cache:* Hash of full prompt string → cached response. Effective for repeated FAQ-style queries.
- *Semantic cache:* Embedding similarity search → cached response. 30–80% hit rate for FAQ workloads; near-zero for open-ended generation.
- *Prompt prefix cache:* OpenAI and Anthropic both support automatic prefix caching for long system prompts — up to 90% discount on repeated prefix tokens. Structure prompts to keep the static prefix as long as possible.

**Streaming:** Start rendering output to the user as soon as the first token arrives. Reduces perceived latency dramatically even when total latency is unchanged. Required for any user-facing chatbot or copilot.

**Model cost tiers (approximate 2025 pricing, input/output per 1M tokens):**
- Tier 1 (capable): GPT-4o ~$5/$15, Claude 3.5 Sonnet ~$3/$15
- Tier 2 (balanced): GPT-4o-mini ~$0.15/$0.60, Claude Haiku ~$0.25/$1.25
- Tier 3 (self-hosted): Llama-3-8B-Q4 on consumer GPU ~$0.005 equivalent

Route to the cheapest model that satisfies quality requirements for each request type.

## Links
- [[llm_observability|LLM Observability]]
- [[safety_and_content_moderation|Safety and Content Moderation]]
- [[serving_frameworks|LLM Serving Frameworks]]
- [[system_design_fundamentals|System Design Fundamentals]]
- [[fastapi_patterns|FastAPI Patterns]]
