---
layer: 05_ai_engineering
type: engineering
tool: instructor
status: growing
tags: [structured-outputs, pydantic, instructor, constrained-generation, json, llm]
created: 2026-03-05
---

# Structured Outputs

## Purpose

Structured outputs guarantee that LLM responses conform to a specific schema — typed objects, validated JSON, or constrained string patterns — rather than returning free-form text. This is essential in production pipelines where downstream code depends on a specific data shape: extraction pipelines, classification, tool call arguments, database writes, and API responses. Without schema enforcement, even small format deviations cause parsing failures that are difficult to handle gracefully at scale.

## Architecture

Three primary approaches exist on a spectrum from soft constraints to hard guarantees:

### (a) Instructor — Pydantic + LLM + Validation Loop
Instructor wraps an LLM client (OpenAI, Anthropic, Cohere, etc.) and adds a retry loop around schema validation. The developer defines a Pydantic model; Instructor translates it into a JSON schema passed to the model via the function-calling API, parses the response back into the Pydantic model, and retries with validation errors in context if parsing fails.

```
User query → LLM (with schema) → JSON response → Pydantic parse
                     ↑                                     |
                     └─── retry with error message ────────┘ (if invalid)
```

Supports nested models, optional fields, and custom validators. Retries can be configured with exponential backoff and max attempts.

### (b) OpenAI JSON Mode / `response_format`
Setting `response_format: {"type": "json_object"}` instructs the model to output valid JSON. This is a soft constraint: the model tries to comply but the schema is not enforced beyond syntactic JSON validity. `response_format: {"type": "json_schema", "json_schema": {...}}` (OpenAI structured outputs, 2024) adds strict schema enforcement at the API level with 100% compliance guarantee for supported models.

### (c) Constrained Generation — Guidance / Outlines
Grammar-constrained decoding at the token level: the sampling distribution is masked at each step to allow only tokens that extend a valid prefix according to the schema (regex, JSON schema, context-free grammar). Guarantees structurally valid output by construction — no retries needed. Requires access to the model's logit layer, so only applicable to locally-deployed models.

**Pydantic semantic validators** layer additional business logic on top of structural validation: value range constraints (`Field(ge=0, le=1)`), regex patterns, cross-field validation with `@model_validator`.

## Implementation Notes

**Instructor quickstart**
```python
import instructor
from openai import OpenAI
from pydantic import BaseModel

client = instructor.patch(OpenAI())

class UserProfile(BaseModel):
    name: str
    age: int
    email: str

profile = client.chat.completions.create(
    model="gpt-4o",
    response_model=UserProfile,
    messages=[{"role": "user", "content": "Extract: John Smith, 34, john@example.com"}]
)
# profile is a validated UserProfile instance
```

**Nested models** work transparently with Instructor — define child Pydantic models and reference them as field types.

**Validation retries**: Instructor passes the Pydantic `ValidationError` message back to the model in a follow-up turn, giving it the opportunity to correct the output. Set `max_retries=3` to bound retry loops.

**Guidance constrained generation**
```python
import guidance
lm = guidance.models.LlamaCpp(model_path)
with guidance.user():
    lm += "Extract the sentiment: " + user_input
with guidance.assistant():
    lm += guidance.select(["positive", "negative", "neutral"], name="sentiment")
```

**When to use structured outputs**
- Information extraction pipelines (entities, relations, attributes from documents)
- Classification with a fixed label set
- Tool call argument population (see [[function_calling|Function Calling]])
- Structured analytics responses (charts data, table rows)
- Any output that feeds directly into typed application code

**OpenAI Structured Outputs (strict mode)** is the lowest-friction path when using the OpenAI API and the schema can be expressed in the supported subset of JSON Schema. Use Instructor when you need cross-provider portability, richer validation logic, or automatic retries.

## Trade-offs

| Approach | Guarantee | Dev UX | Requires local model | Notes |
|---|---|---|---|---|
| Prompting only | None | Simple | No | Fragile at scale |
| JSON mode | Syntactic JSON | Low overhead | No | No schema enforcement |
| OpenAI Structured Outputs | Schema + type | Good | No | OpenAI-only, strict schema subset |
| Instructor | Pydantic semantics | Excellent | No | Retries = extra API calls |
| Constrained generation | Hard guarantee | Complex | Yes | Only local models; no retries needed |

Instructor strikes the best developer experience balance for most API-based production systems. Constrained generation is the right choice when absolute output guarantees are required and a local model is available (e.g., embedded systems, regulated environments).

## References

- Instructor documentation: https://python.useinstructor.com
- OpenAI Structured Outputs: https://platform.openai.com/docs/guides/structured-outputs
- Guidance (Microsoft): https://github.com/guidance-ai/guidance
- Outlines: https://github.com/outlines-dev/outlines
- Willard & Louf (2023). *Efficient Guided Generation for Large Language Models*.

## Links
- [[prompting_strategies|Prompting Strategies]]
- [[function_calling|Function Calling]]
- [[dspy_systematic_prompting|DSPy — Systematic Prompt Optimization]]
