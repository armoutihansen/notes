---
layer: 03_software_engineering
type: engineering
tool: Copilot/LLM
status: growing
tags: [llm, code-generation, copilot, ai-tools, productivity]
created: 2026-03-05
---

# LLM Code Generation

## Purpose

LLM-based code generation tools accelerate software development by automating predictable, pattern-heavy code. The goal is not to replace engineering judgment but to compress time spent on boilerplate, tests, and documentation — freeing attention for design decisions, architecture, and the novel parts of a problem. Understanding where LLMs add value and where they mislead is as important as knowing how to prompt them.

## Architecture

### Tool Landscape

```
┌───────────────────────────────────────────────────────┐
│                    LLM Code Tools                     │
│                                                       │
│  Tab Completion        Inline Chat        Agents      │
│  ─────────────         ───────────        ──────      │
│  GitHub Copilot        Copilot Chat       Copilot     │
│  Cursor (tab)          Cursor CMD+K       Agent Mode  │
│  Supermaven            Cody (Sourcegraph) Devin       │
│  Codeium               Continue           Claude Code │
└───────────────────────────────────────────────────────┘
         │                    │                  │
         ▼                    ▼                  ▼
    token stream          turn-based         tool-use loop
    (low-latency)         dialogue           (read/write/exec)
```

### How Completion Works

Tab completion models operate on a **Fill-in-the-Middle (FIM)** objective: given prefix code (before cursor) and suffix code (after cursor), predict the middle. This gives models awareness of what you're writing toward, not just what came before.

Context window contents at inference time:
1. Cursor prefix (up to N tokens)
2. Cursor suffix (up to M tokens)
3. Related file snippets (retrieved via BM25 or embedding similarity)
4. Open editor tabs (recency-weighted)
5. Repository-level context (if available — Copilot Enterprise, Cursor)

## Implementation Notes

### Effective Prompting for Code

The core principle: **specificity beats brevity**. The more constraints you express, the less ambiguity the model has to fill in with a plausible-but-wrong assumption.

**Language and framework**
```python
# Bad: too vague
# Parse the config

# Good: explicit contract
def parse_config(path: str) -> dict[str, Any]:
    """Load YAML config from path.

    Raises:
        FileNotFoundError: if path does not exist
        yaml.YAMLError: if file is not valid YAML
    Returns dict with string keys, values of any type.
    """
```

**Error handling style**
```python
# Specify by example in the prompt or surrounding code
# Model will match the existing pattern:

# Existing code uses explicit Result type:
def read_file(path: str) -> Result[str, IOError]: ...

# Model completion will likely use Result, not raise
```

**Tests — be specific about what to assert**
```python
# Prompt: "Write pytest tests for the function below.
# Test happy path, empty input, and that ValueError is raised
# when the argument is negative. Use parametrize."
```

**Inline chat prompts that work well**
```
/doc        — generate docstring for selected function
/tests      — generate unit tests
/fix        — explain and fix selected error
/explain    — plain-English explanation of selected code
Refactor this to use a dataclass instead of a plain dict.
Add type annotations to this function.
Convert this to async using asyncio.
Write a FastAPI endpoint that wraps this function.
```

**Whole-file generation — effective workflow**

1. Write a detailed comment block at the top: purpose, inputs, outputs, constraints, dependencies.
2. Write the function/class signatures as stubs.
3. Let the model fill in implementations one function at a time.
4. Do not generate an entire module at once — review as you go.

### When LLMs Help Most

| Task | Why LLMs excel |
|------|---------------|
| Boilerplate (CRUD, API clients) | High pattern density, low novelty |
| Unit tests | Structure is predictable; model inverts the function |
| Docstrings and comments | Pure language task, no subtle logic |
| Type annotations | Constrained vocabulary, local inference |
| Refactoring (rename, extract, convert) | Mechanical transformation |
| Regex patterns | Painful to write, easy to describe |
| SQL queries | Well-constrained language, schema often visible |
| Shell one-liners | Command flags are in training data |
| Translating between languages | `convert this Python to Go` works well |
| Explaining unfamiliar code | Strong at summarisation |

### When LLMs Mislead

| Task | Why LLMs struggle |
|------|-----------------|
| Novel algorithms | No training signal for new problems |
| Subtle concurrency bugs | Require global reasoning across threads/locks |
| Security-sensitive code | May produce plausible-but-vulnerable patterns |
| Library-version-specific APIs | Training data may be outdated |
| Complex numerical code | Floating-point edge cases invisible to language models |
| Architectural decisions | Model doesn't know your constraints |
| Business logic with many invariants | Model can't hold all constraints simultaneously |
| Performance-critical hot paths | May choose readable over optimal |

Key failure modes:
- **Hallucinated APIs**: model invents methods that don't exist in the actual library version
- **Stale patterns**: training data cutoff means model uses deprecated idioms
- **Confident wrongness**: no calibrated uncertainty — incorrect code looks identical to correct code
- **Context bleed**: model may blend patterns from different frameworks

### Code Review of AI Output

AI-generated code demands more careful review than human-written code because the author has no understanding — only pattern matching. Checklist:

**Logic**
- [ ] Does the algorithm actually solve the stated problem?
- [ ] Are all edge cases handled (empty input, zero, None, overflow)?
- [ ] Are loop bounds and index arithmetic correct?
- [ ] Are early returns and guard clauses complete?

**Security**
- [ ] Is user input sanitised before use in queries / shell commands?
- [ ] Are secrets handled correctly (not logged, not in exceptions)?
- [ ] Are file paths validated to prevent path traversal?
- [ ] Are HTTP responses checked for status codes?

**Error handling**
- [ ] Are exceptions caught at the right granularity?
- [ ] Do error messages expose sensitive information?
- [ ] Are resources (files, DB connections) closed in finally/context managers?

**Style and maintainability**
- [ ] Does the code follow project conventions?
- [ ] Are there unnecessary dependencies introduced?
- [ ] Is the code over-engineered for the task?

**Tests**
- [ ] Do generated tests actually test the right thing?
- [ ] Are assertions specific (not just "does not raise")?
- [ ] Are mocks/stubs patching the right import path?

### Workflow Integration

**Copilot in VS Code / JetBrains**
- `Tab` to accept, `Alt+]` / `Alt+[` to cycle suggestions
- `Ctrl+Enter` to open suggestion panel (multiple options)
- `Ctrl+I` (JB) / inline chat shortcut for inline prompt
- `@workspace` in Copilot Chat to include repository context

**Cursor**
- `Cmd+K` — inline edit with natural language
- `Cmd+L` — open chat (includes file context automatically)
- `.cursorrules` file in repo root — persistent system prompt for project conventions

**Continue (VS Code extension, open-source)**
- Supports local models (Ollama) and remote APIs
- `config.json` configures models, context providers, slash commands
- Custom context providers can index your codebase with embeddings

## Trade-offs

| Approach | Pro | Con |
|----------|-----|-----|
| Tab completion | Zero friction, fast | Limited context, no dialogue |
| Inline chat | Targeted, interactive | Requires good prompts |
| Agent mode | Can do multi-file tasks | High risk of cascading errors |
| Cloud model (Copilot, Claude) | Best quality | Code leaves machine |
| Local model (Ollama, Continue) | Private | Smaller, lower quality |
| Context window stuffing | More info → better output | Slow, expensive, dilutes signal |

## References

- [GitHub Copilot documentation](https://docs.github.com/en/copilot)
- [Cursor docs](https://cursor.sh/docs)
- [FIM paper — Bavarian et al. 2022](https://arxiv.org/abs/2207.14255)
- [Is Your Code Generated by ChatGPT Really Correct? (Liu et al. 2023)](https://arxiv.org/abs/2305.01210)
- [Continue (open-source AI code assistant)](https://continue.dev/)

## Links
- [[mcp_protocol|MCP]]
- [[agentic_coding|Agentic Coding]]
