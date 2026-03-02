---
layer: 05_ai_engineering
type: engineering
tool: google-genai
status: evergreen
tags: [agents, function-calling, llm, tool-use]
created: 2026-03-02
---

# Function Calling

## Purpose

**Function calling** (tool use) is the mechanism by which an LLM selects and invokes typed external functions based on a user request. The model does not execute the functions itself — it generates a structured call specification; the host application dispatches the actual execution and returns results.

## Architecture

```
Developer registers tools:
  Tool = [FunctionDeclaration(name, description, parameters_schema), ...]
                │
                ▼
  LLM receives: system_prompt + user_message + tool_declarations
                │
                ▼
  LLM outputs: function_calls = [{name, args}, ...]
                │
                ▼
  Host dispatches: result = function_map[name](**args)
                │
                ▼
  Host returns:  Content(role="tool", parts=[FunctionResponse(name, result)])
                │
                ▼
  LLM continues reasoning with tool results in context
```

## Implementation Notes

**Schema definition (google-genai)**

```python
from google.genai import types

schema_my_tool = types.FunctionDeclaration(
    name="my_tool",
    description="One-sentence description of what the tool does and when to use it.",
    parameters=types.Schema(
        type=types.Type.OBJECT,
        properties={
            "param_name": types.Schema(
                type=types.Type.STRING,
                description="Clear description of this parameter's purpose and expected format.",
            ),
        },
        required=["param_name"],
    ),
)

available_tools = types.Tool(function_declarations=[schema_my_tool, ...])
```

**Description quality is critical**

The model selects tools and fills arguments based entirely on the natural-language descriptions. Poor descriptions cause wrong tool selection and malformed arguments:

| Bad | Good |
|---|---|
| `"Reads a file"` | `"Reads the complete text content of a file at the given path, up to 10,000 characters"` |
| `"path: string"` | `"path: Path to the file relative to the working directory, e.g. 'src/main.py'"` |

**Dispatch pattern**

```python
function_map = {"tool_name": tool_function, ...}

def call_function(function_call, working_directory):
    name = function_call.name
    if name not in function_map:
        return make_tool_response(name, {"error": f"Unknown function: {name}"})
    args = dict(function_call.args or {})
    args["working_directory"] = working_directory  # server-side injection
    return make_tool_response(name, {"result": function_map[name](**args)})

def make_tool_response(name, response):
    return types.Content(
        role="tool",
        parts=[types.Part.from_function_response(name=name, response=response)],
    )
```

**Server-side parameter injection**

Never let the model supply security-sensitive parameters (e.g., `working_directory`, API keys, database credentials). Inject them in the dispatcher. This prevents prompt-injection attacks where the model is tricked into passing a malicious value.

**Return structured errors to the model**

Instead of raising exceptions in tool functions, return `{"error": "..."}` so the model can reason about the failure and retry or report it gracefully.

**System prompt conventions**

- State the agent's role concisely (one sentence)
- List available operations at a high level (the model already has the schema details)
- Specify path and format conventions (e.g., "all paths are relative to the working directory")

## Trade-offs

| Concern | Notes |
|---|---|
| Schema verbosity | More detailed descriptions improve accuracy but increase prompt token cost |
| Flat vs. nested schemas | Flat parameter sets are easier for the model to fill correctly |
| Optional vs. required params | Mark required if omitting would cause errors; optional adds flexibility |
| Error transparency | Returning errors enables self-correction; suppressing them causes silent failures |

## References

- [Google Gemini function calling](https://ai.google.dev/gemini-api/docs/function-calling)
- [OpenAI function calling](https://platform.openai.com/docs/guides/function-calling)

## Links

- [[agentic_loop]]
- [[03_software_engineering/security/filesystem_sandboxing]]
- [[07_projects/_completed/build-ai-agent/agent_architecture]]
