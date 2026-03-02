---
layer: 07_projects
type: project
status: evergreen
owner: jesper
created: 2026-03-02
tags: [ai-agents, function-calling, gemini, lessons]
---

# Lessons Learned — Build AI Agent

## Function Calling with Gemini SDK

- Tool schemas are defined as `types.FunctionDeclaration` with a `types.Schema` for parameters — analogous to OpenAI's JSON schema tool definitions.
- The model selects and fills in tool arguments based on parameter descriptions; **description quality directly affects tool choice accuracy**.
- Multiple function calls can be returned in a single response (`response.function_calls` is a list) — the loop must handle batches, not just single calls.
- Tool results are fed back as `Content(role="tool", parts=[Part.from_function_response(...)])`, not as ordinary user messages.

## Agentic Loop Design

- The core pattern is: *prompt → LLM → function calls → results → LLM → … → final text*. Message history grows with each iteration, keeping full context visible to the model.
- Termination condition: the model returns a text response (no function calls). Fallback: hard iteration cap (`sys.exit(1)` at 20).
- The model can plan several calls per iteration — batch dispatching avoids unnecessary round-trips.

## Sandboxing File System Tools

- `os.path.commonpath([abs_working_dir, abs_target])` is a clean way to enforce that all paths stay inside a designated directory.
- `os.path.normpath` + `os.path.abspath` must both be applied before comparison to neutralise `../` traversal attempts.
- Returning structured errors to the model (rather than raising exceptions) allows the agent to recover gracefully.

## System Prompt Design

- A short, imperative system prompt (role + list of available operations + path convention) was sufficient. The model inferred sequencing and error handling from context.
- Explicitly stating "all paths are relative to the working directory" prevented the model from constructing absolute paths.

## Tool Design Principles

- Each tool should do one thing with clear inputs and outputs — the model generalises better.
- Read limits (10,000 chars) and execution timeouts (30s) are necessary safety bounds; the truncation marker helps the model detect incomplete content.
- The working directory should be injected server-side (not passed by the model) to prevent prompt-injection attacks.

## Extracted to Core Layers

- [[05_ai_engineering/agents/agentic_loop]] — generalised agentic loop pattern
- [[05_ai_engineering/agents/function_calling]] — tool schema design, dispatch, server-side injection
- [[03_software_engineering/security/filesystem_sandboxing]] — `commonpath` sandboxing pattern

## Links

- [[overview]]
- [[agent_architecture]]
