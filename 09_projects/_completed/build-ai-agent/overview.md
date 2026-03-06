---
layer: 09_projects
type: project
status: completed
owner: jesper
created: 2026-03-02
tags: [ai-agents, gemini, function-calling, python]
---

# Build AI Agent

## Goal

Build a minimal but fully functional AI agent using the Google Gemini API. The agent accepts natural-language prompts via CLI and autonomously completes multi-step coding tasks by calling sandboxed tools in an iterative reasoning loop.

## Scope (In / Out)

**In:**
- Agentic loop with up to 20 tool-call iterations
- Four sandboxed tool functions (list files, read file, run Python, write file)
- CLI interface (`python main.py "<prompt>" [--verbose]`)
- Working directory sandboxing to prevent path traversal
- Demo: agent builds a calculator module from scratch

**Out:**
- Multi-turn conversation / persistent memory
- Web search or external API tools
- Streaming responses
- Custom tool registration at runtime
- Multi-agent coordination

## Deliverables

- `main.py` — agentic loop and CLI entry point
- `prompts.py` — system prompt defining agent role and available operations
- `call_function.py` — function dispatcher with sandboxed `working_directory` injection
- `functions/` — four tool implementations (`get_files_info`, `get_file_content`, `run_python_file`, `write_file`)
- `calculator/` — demo working directory used to validate the agent end-to-end

## Data

N/A — no datasets. The agent operates on the file system of a sandboxed working directory.

## Modeling

Uses **Gemini 2.5 Flash** via `google-genai` SDK with function calling enabled. No fine-tuning or custom model training.

## Engineering

| Concern | Choice |
|---|---|
| Language | Python 3.13 |
| Model | Gemini 2.5 Flash |
| SDK | `google-genai==1.12.1` |
| Env management | `uv` + `.env` via `python-dotenv` |
| Max iterations | 20 function calls per prompt |
| Sandboxing | `os.path.commonpath` path validation |
| Execution | `subprocess.run` with 30s timeout |
| File read limit | 10,000 characters per file |

## Timeline

Completed (learning project).

## Links

- [[agent_architecture]]
- [[lessons_learned]]
- Related layer: `06_ai_engineering/` (LLM system patterns, agentic architectures)
