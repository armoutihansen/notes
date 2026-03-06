---
layer: 05_ai_engineering
type: engineering
tool: general
status: growing
tags: [function-calling, tool-use, agents, json-schema, openai, llm]
created: 2026-03-05
---

# Function Calling

## Purpose

Function calling is the structured mechanism by which LLMs invoke external tools and APIs. Rather than describing an action in free-form text (which requires fragile output parsing), the model produces a formally structured tool call — a JSON object specifying a tool name and typed arguments — that the application can execute programmatically. This is the foundational mechanism for agentic systems, data extraction, and any LLM workflow that must interact with external systems reliably.

## Architecture

### Core Pattern

```
1. Developer defines tools as JSON schema
      { name, description, parameters: { properties, required } }

2. Tool schemas passed to LLM alongside user message

3. LLM outputs a structured tool call:
      { "name": "get_weather", "arguments": { "location": "Tokyo", "unit": "celsius" } }

4. Application validates arguments and executes the tool

5. Tool result returned to LLM as a tool message:
      { "role": "tool", "content": "{\"temperature\": 22, \"conditions\": \"cloudy\"}" }

6. LLM generates a natural language response based on the result
   (or issues another tool call — repeat until done)
```

### Parallel Function Calling
GPT-4 Turbo and GPT-4o support parallel function calling: the model can issue multiple tool calls in a single turn when the tools are independent. The application executes all calls concurrently and returns all results before the next LLM turn. This halves the number of LLM turns for tasks with independent sub-operations.

```json
[
  {"name": "get_weather", "arguments": {"location": "Tokyo"}},
  {"name": "get_weather", "arguments": {"location": "London"}}
]
```

### Forced Tool Use
To guarantee the model uses a specific tool (e.g., for structured extraction), set `tool_choice`:
```python
tool_choice={"type": "function", "function": {"name": "extract_entities"}}
```
The model must call the specified function, effectively turning function calling into a constrained structured output mechanism. See [[structured_outputs|Structured Outputs]].

## Implementation Notes

**Tool schema definition**
```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "search_knowledge_base",
            "description": "Search the product knowledge base for relevant information. Use this when the user asks about product features, pricing, or policies. Returns a list of relevant document excerpts.",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "The search query. Should be a concise, keyword-rich reformulation of the user's question."
                    },
                    "max_results": {
                        "type": "integer",
                        "description": "Maximum number of results to return. Default: 5.",
                        "default": 5
                    }
                },
                "required": ["query"]
            }
        }
    }
]
```

**Tool description quality is the primary engineering lever**
The LLM selects tools entirely based on names and descriptions — the code behind the tool is invisible to it. Invest heavily in:
- Clear statement of what the tool does and when to use it.
- Precise description of each parameter's meaning and expected format.
- What the output represents and how to interpret it.
- What the tool does *not* do (to prevent misuse).

**Tool result format**
Return a structured object that includes status, data, and sufficient metadata for the model to reason about the result:
```json
{
  "status": "success",
  "results": [...],
  "total_found": 12,
  "query_used": "Tokyo weather"
}
```
On error, include a helpful message and a suggested recovery action.

**Streaming tool calls**
OpenAI and Anthropic support streaming tool call deltas, allowing the UI to show partial tool call information before execution. Useful for improving perceived responsiveness in interactive applications.

**Security: server-side argument validation**
Never trust LLM-generated tool arguments as safe inputs. The model may have been manipulated (prompt injection) into generating malicious arguments. Always validate all arguments server-side before execution:
- Schema validation (types, ranges, required fields)
- Authorization checks (can this user access this resource?)
- Sanitization for SQL, shell, file path injection
See [[prompt_injection_and_guardrails|Prompt Injection and Guardrails]].

**Caching deterministic tool calls**
If a tool is deterministic and expensive (e.g., a database query), cache results keyed by (tool_name, arguments) within a session. Reduces cost and latency for repeated identical calls in the same agent run.

**Tool selection pre-filtering**
When the agent has access to many tools (>20), the model's ability to select correctly degrades. Implement a two-stage approach: first classify the query to identify the relevant tool category, then present only the relevant subset of tool schemas to the LLM. See [[agentic_loop|Agentic Loop]].

**Never expose raw function signatures**
Tool names and descriptions are visible to the model (and potentially to adversaries via prompt extraction). Design tool names and descriptions to be descriptive of purpose, not implementation. Do not include internal system paths, credentials, or architectural details in tool schemas.

## Trade-offs

**More tools → lower accuracy**: Empirically, model accuracy on tool selection decreases as the number of available tools grows. Keep the tool set focused; use routing to limit tool visibility per query context.

**Strict schemas vs. flexibility**: Tight type constraints (`enum` values, `minimum`/`maximum`) reduce the space of valid outputs, helping the model produce well-formed arguments, but can fail on edge cases. Balance strictness with flexibility based on observed failure modes.

**Parallel calls vs. sequential**: Parallel tool calling reduces turn count but requires the application to handle concurrent execution and error aggregation. Sequential execution is simpler but slower.

**Forced tool use as extraction**: Using `tool_choice` to force a specific tool call is a reliable pattern for structured extraction (equivalent to Instructor without the library dependency). Latency and cost are identical to standard function calling.

## References

- OpenAI Function Calling documentation: https://platform.openai.com/docs/guides/function-calling
- Anthropic Tool Use documentation: https://docs.anthropic.com/en/docs/tool-use
- Schick et al. (2023). *Toolformer: Language Models Can Teach Themselves to Use Tools*. NeurIPS.

## Links
- [[agentic_loop|Agentic Loop]]
- [[structured_outputs|Structured Outputs]]
- [[prompt_injection_and_guardrails|Prompt Injection and Guardrails]]
- [[multi_agent_systems|Multi-Agent Systems]]
