---
layer: 03_software_engineering
type: engineering
tool: AI Agents
status: growing
tags: [agents, agentic-coding, ai-tools, automation, code-generation]
created: 2026-03-05
---

# Agentic Coding

## Purpose

Agentic software engineering extends single-shot code generation to multi-step autonomous loops: an AI agent can read files, write code, run tests, observe failures, and iterate — with minimal human intervention between steps. The promise is compressing multi-hour tasks (implement feature, add tests, fix the build) into minutes. The reality is that agents work well on well-scoped, well-tested tasks and fail in unpredictable ways on underspecified or novel problems.

## Architecture

### The Code Agent Loop

```
                    ┌──────────────────────┐
                    │      Goal / Task     │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                ┌──▶│         READ         │  understand current state
                │   │  files, tests, docs  │
                │   └──────────┬───────────┘
                │              │
                │              ▼
                │   ┌──────────────────────┐
                │   │         PLAN         │  decompose task into steps
                │   │  (chain-of-thought)  │
                │   └──────────┬───────────┘
                │              │
                │              ▼
                │   ┌──────────────────────┐
                │   │         WRITE        │  edit files, create files
                │   │   (tool: file edit)  │
                │   └──────────┬───────────┘
                │              │
                │              ▼
                │   ┌──────────────────────┐
                │   │         TEST         │  run tests, lint, type-check
                │   │  (tool: shell exec)  │
                │   └──────────┬───────────┘
                │              │
                │   ┌──────────▼───────────┐
                │   │       OBSERVE        │  parse output, check pass/fail
                │   └──────────┬───────────┘
                │              │
                │    ┌─────────▼─────────┐
                │    │  Pass? ──── Yes ──▶│ Done
                │    │  No               │
                │    └────────┬──────────┘
                │             │
                └─────────────┘   (iterate up to N times)
```

The loop is instantiated differently by each tool — but the shape is universal.

### Tool Use in Coding Agents

Agents are LLMs augmented with structured tool-call capabilities. The model emits tool calls; a harness executes them and feeds results back as context. Common tools in coding agents:

| Tool | Purpose | Risk |
|------|---------|------|
| `read_file(path)` | Read source files | Low |
| `write_file(path, content)` | Create or overwrite files | Medium — can corrupt |
| `edit_file(path, old, new)` | Surgical text replacement | Medium — mis-match |
| `bash(command)` | Run shell commands | **High** — arbitrary execution |
| `search_files(pattern)` | grep / glob over codebase | Low |
| `browser(url)` | Fetch documentation / issues | Medium — prompt injection |
| `list_directory(path)` | Explore structure | Low |

The `bash` tool is the most powerful and most dangerous. It enables running tests and builds but also enables accidental `rm -rf`, network calls, and exfiltration if not sandboxed.

## Implementation Notes

### Current Agentic Coding Tools

**GitHub Copilot Agent Mode (2025)**
- Activated via the Copilot Chat panel with `#agent` or by selecting "Agent Mode"
- Can read, create, and edit multiple files across the workspace
- Runs terminal commands (with user approval by default)
- Integrated into VS Code and JetBrains; aware of LSP diagnostics and test output
- Safety: commands require explicit user approval unless running in fully-autonomous mode

**Cursor**
- Composer (multi-file edit) and Agent mode
- `Cmd+Shift+I` opens Composer — describe a feature, Cursor writes across all affected files
- Agent mode adds terminal execution; can run builds and tests autonomously
- `.cursorrules` at repo root acts as a persistent system prompt: coding conventions, project structure, style rules

**Claude Code (Anthropic CLI, 2025)**
- Terminal-based agent: `claude` in any directory gives it full shell + file access
- Reads the entire repository before planning
- Uses `bash`, `read_file`, `write_file`, `search` tools natively
- Can be pointed at GitHub issues: `claude "implement the feature described in issue #42"`
- `CLAUDE.md` at repo root: persistent instructions, build commands, conventions

**Devin (Cognition AI)**
- Cloud-hosted, fully autonomous — runs in a sandboxed VM with browser + terminal
- Long-horizon tasks: clone repo, understand codebase, implement feature, open PR
- Human-in-the-loop via Slack integration for clarification

**SWE-agent (Princeton)**
- Research system; open-source
- Designed to resolve GitHub issues autonomously
- Demonstrated competitive performance on SWE-bench (resolving real GitHub issues)

### Effective Agent Scaffolding

The quality of agent output is highly sensitive to how the task is specified and the context made available. Treat agent prompting as a form of engineering.

**Clear goal specification**

Vague:
```
Add error handling to the API.
```

Effective:
```
Add error handling to all FastAPI endpoint functions in src/api/routes/.
- Wrap each endpoint body in a try/except
- Catch ValueError and return HTTP 422 with the error message
- Catch all other exceptions and return HTTP 500 with a generic message
- Do not modify the function signatures or return types
- Run pytest after each file change and only proceed if tests pass
```

**Constrained action space**

Reduce the blast radius by scoping what the agent can touch:
- Specify which directories are in scope
- Forbid modifying test files (let the agent only make tests pass, not change them)
- Specify which commands may be run
- Set a maximum number of files to change

**Human-in-the-loop checkpoints**

For non-trivial tasks, break the work into phases with review points:
1. Agent reads codebase and produces a plan (STOP — human reviews plan)
2. Agent implements plan up to first test run (STOP — human reviews diff)
3. Agent iterates on failures (STOP — human reviews final diff before commit)

Most tools support this via approval modes. Claude Code: `--approvals` flag. Copilot Agent Mode: "Require approval for all commands."

**CLAUDE.md / .cursorrules — persistent system context**

```markdown
# Project: MyApp API

## Build & Test
- Install: `pip install -e ".[dev]"`
- Test: `pytest tests/ -q`
- Lint: `ruff check . && mypy src/`
- Type check must pass before committing

## Conventions
- All functions must have type annotations
- Use `Result[T, E]` pattern from `returns` library for error-prone operations
- No print statements; use `structlog` for logging
- Tests use `pytest` with `pytest-asyncio` for async tests
- Fixtures in `tests/conftest.py`

## Architecture
- `src/api/` — FastAPI route handlers (thin, delegate to services)
- `src/services/` — business logic
- `src/models/` — SQLAlchemy models
- `src/schemas/` — Pydantic schemas
- External APIs in `src/clients/`

## Do Not
- Do not modify `tests/` unless asked
- Do not add new top-level dependencies without noting them
- Do not change database migrations without explicit instruction
```

### SWE-bench as a Benchmark

[SWE-bench](https://www.swebench.com/) evaluates agents on real GitHub issues from popular Python repositories. Each task: given a failing test that was added when the issue was fixed, resolve the issue so the test passes.

2024 state:
- GPT-4 (no agent): ~2%
- SWE-agent (GPT-4): ~12%
- Devin: ~14%
- Claude 3.5 Sonnet (SWE-agent): ~19%

These numbers are advancing rapidly. SWE-bench Verified (curated, harder subset) remains challenging.

### Risks and Mitigations

**Hallucinated dependencies**

Agent adds `import some_library` that doesn't exist, causing import errors. Mitigation: run `pip install` in a virtual environment; fail loudly on missing dependencies; check that dependencies are in `requirements.txt` or `pyproject.toml`.

**Cascading errors**

A wrong first change causes subsequent changes to work around the bug rather than fix it, producing a self-consistent but incorrect solution. Mitigation: small scoped tasks; verify intermediate state; revert and retry if the diff grows unexpectedly large.

**Test gaming**

Agent modifies tests to make them pass rather than fixing the underlying code. Mitigation: mark test files as read-only in the agent's action space; separate test-writing and implementation tasks.

**Security — arbitrary code execution**

Agents with `bash` access can exfiltrate credentials, install malware, or destroy data if given a malicious prompt (e.g., via a file in the repo containing prompt injection). Mitigations:
- Run agents in a container / VM with no network access (air-gap except required APIs)
- No cloud credentials in the environment
- No `sudo` or root
- Audit all generated shell commands before execution

**Context drift**

Long agent sessions accumulate context that causes the model to forget earlier constraints or mix up file paths. Mitigation: keep task scope narrow; start a fresh agent session for each distinct task.

## Trade-offs

| Approach | Pro | Con |
|----------|-----|-----|
| Fully autonomous | Fast, no human wait time | High risk of cascading failures |
| Human-in-the-loop | Safe, reviewable | Slower, defeats some of the value |
| Cloud agent (Devin) | Sandboxed VM, powerful | Expensive, code leaves machine |
| Local IDE agent (Cursor, Copilot) | Fast, private | Less sandboxed, terminal risk |
| Narrow task scope | Reliable, predictable | Doesn't capture full value |
| Broad task scope | More leverage | Exponential failure modes |
| Test-driven scaffolding | Agent has clear success signal | Requires good existing test suite |

## References

- [SWE-bench: Can Language Models Resolve Real-World GitHub Issues? (Jimenez et al. 2023)](https://arxiv.org/abs/2310.06770)
- [SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering (Yang et al. 2024)](https://arxiv.org/abs/2405.15793)
- [GitHub Copilot Agent Mode](https://code.visualstudio.com/docs/copilot/chat/chat-agent-mode)
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [Cursor Agent Mode](https://docs.cursor.com/agent)
- [Devin (Cognition)](https://cognition.ai/)

## Links
- [[llm_code_generation|LLM Code Gen]]
- [[mcp_protocol|MCP]]
