---
layer: 05_ai_engineering
type: engineering
tool: google-genai
status: evergreen
tags: [agents, agentic-loop, llm, function-calling]
created: 2026-03-02
---

# Agentic Loop

## Purpose

An **agentic loop** is a multi-iteration control structure that drives an LLM to call tools repeatedly until it reaches a terminal condition (final text response or iteration cap). It is the core runtime pattern for tool-using AI agents.

## Architecture

```
User prompt
    │
    ▼
messages = [Content(role="user", text=prompt)]
    │
    ▼
┌──────────────────────────────────────────────┐
│  for iteration in range(max_iterations):     │
│                                              │
│    response = llm.generate(messages, tools)  │
│    append response.content → messages        │
│                                              │
│    if response.function_calls:               │
│      for call in function_calls:             │
│        result = dispatch(call)               │
│        append result → messages              │
│      continue                                │
│                                              │
│    else:  ← terminal: text response          │
│      return response.text                    │
│                                              │
└──────────────────────────────────────────────┘
    │
    ▼  (iteration cap reached)
  exit / raise
```

**Key invariant**: the message history grows monotonically — all prior tool calls and results remain visible to the model at every iteration. This gives the model full planning context without external memory.

## Implementation Notes

**Termination detection**

```python
for _ in range(max_iterations):
    response = llm.generate(messages, tools)
    messages.append(response.content)

    if response.function_calls:
        results = [dispatch(call) for call in response.function_calls]
        messages.append(Content(role="user", parts=results))
    else:
        print(response.text)
        break
else:
    # for-else: loop exhausted without break
    sys.exit(1)
```

**Batch function call handling**

The model may return multiple function calls in a single response. Collect all results before appending:

```python
function_results = []
for call in response.function_calls:
    result = dispatch(call)
    function_results.append(result.parts[0])
messages.append(Content(role="user", parts=function_results))
```

**Message role conventions** (Google Gemini / OpenAI-compatible):

| Role | When used |
|---|---|
| `"user"` | Initial prompt and tool results |
| `"model"` / `"assistant"` | LLM responses (including function call declarations) |
| `"tool"` | Function call results (Gemini: `Content(role="tool", parts=[Part.from_function_response(...)])`) |

**Iteration cap**

Set conservatively (e.g., 20). Exceeding it usually signals a broken tool or prompt. On cap: log and exit rather than silently returning empty output.

## Trade-offs

| Choice | Pro | Con |
|---|---|---|
| Full history in context | Model sees all prior results; better planning | Context grows linearly; expensive for long tasks |
| Summarise history | Bounded token cost | Requires summarisation logic; may lose detail |
| Hard iteration cap | Simple; prevents runaway cost | No graceful degradation — abrupt failure |
| Soft cap with partial return | Partial results available | More complex; partial results may be misleading |

## References

- [Google Gemini function calling docs](https://ai.google.dev/gemini-api/docs/function-calling)
- ReAct: *Synergizing Reasoning and Acting in Language Models* (Yao et al., 2022)

## Links

- [[function_calling]]
- `07_projects/_completed/build-ai-agent/agent_architecture`
