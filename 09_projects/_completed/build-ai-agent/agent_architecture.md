---
layer: 09_projects
type: ai_system
status: evergreen
tags: [llm, function-calling, gemini, agentic-loop]
created: 2026-03-02
---

# Agent Architecture

## Goal

A CLI coding agent that completes multi-step coding tasks autonomously via tool use. Given a natural-language prompt, it plans a sequence of file-system and code-execution operations, executes them, and iterates based on results until it produces a final answer.

## Architecture

```
User prompt (CLI)
      │
      ▼
┌─────────────────────────────────────────────┐
│  Agentic Loop  (main.py, max 20 iterations) │
│                                             │
│  1. Build messages list                     │
│  2. Call Gemini 2.5 Flash with tools        │
│  3. If response = function_calls:           │
│       dispatch → tool → append result       │
│       → continue loop                       │
│  4. If response = text:                     │
│       print final answer → break            │
└─────────────────────────────────────────────┘
      │
      ▼
  Tool Dispatcher  (call_function.py)
  Injects working_directory; routes to:
  ┌──────────────────────────────────┐
  │  get_files_info                  │
  │  get_file_content                │
  │  run_python_file                 │
  │  write_file                      │
  └──────────────────────────────────┘
```

**Message history** accumulates across iterations (LLM sees full context including prior tool results). This enables multi-step reasoning without external memory.

## Components

### `main.py` — Agentic Loop

- Parses CLI args (`user_prompt`, `--verbose`)
- Maintains `messages: list[Content]` as the running conversation
- Calls `client.models.generate_content` with `tools=[available_functions]` and `system_instruction=system_prompt`
- On `function_calls` in response: dispatches each call, collects results, appends as `role="user"` Content, continues
- On text response: prints and breaks
- On 20-iteration limit: `sys.exit(1)`

### `prompts.py` — System Prompt

Concise role definition instructing the model to:
1. Make a function call plan for each user request
2. Use only paths relative to the working directory
3. Available operations: list files, read contents, execute Python, write files

### `call_function.py` — Dispatcher

- Defines `available_functions` as a `types.Tool` with all four `FunctionDeclaration` schemas
- `call_function(function_call, verbose)` maps function name → implementation, injects `working_directory="./calculator"`, and wraps result in `types.Content(role="tool", ...)`

### `functions/` — Tool Implementations

| Tool | Description | Key constraints |
|---|---|---|
| `get_files_info` | Lists directory contents with `file_size` and `is_dir` | Blocked outside working dir |
| `get_file_content` | Reads file content | Max 10,000 chars; truncation marker appended |
| `run_python_file` | Executes `.py` file via `subprocess.run` | 30s timeout; `.py` extension required |
| `write_file` | Creates or overwrites a file | Blocked outside working dir; blocked on directories |

**Sandboxing pattern** (all tools):
```python
abs_path = os.path.abspath(working_directory)
full_path = os.path.normpath(os.path.join(abs_path, user_path))
if os.path.commonpath([abs_path, os.path.abspath(full_path)]) != abs_path:
    raise ValueError("Outside permitted working directory")
```

## Evaluation

Validated manually: agent successfully built and tested a `calculator/` module (add, subtract, multiply, divide) from a single natural-language prompt in one session. No formal benchmarks.

## Failure Modes

| Mode | Behaviour |
|---|---|
| Iteration limit exceeded | `sys.exit(1)` — no graceful degradation |
| Path traversal attempt | `ValueError` returned to model as tool error |
| Subprocess timeout (>30s) | `CalledProcessError` propagated |
| Missing API key | `RuntimeError` at startup |
| File read >10k chars | Content truncated with marker; model may miss context |

## Cost / Latency

- **Model**: Gemini 2.5 Flash — optimised for speed and cost
- **Token usage**: logged per iteration when `--verbose` is set (`prompt_token_count`, `candidates_token_count`)
- **Latency**: dominated by LLM inference; tool execution (file I/O, subprocess) adds negligible overhead for small files

## Links

- [[overview]]
- [[lessons_learned]]
- Related: `06_ai_engineering/` — agentic architectures, function calling patterns, LLMOps
