---
layer: 08_implementations
type: application
status: growing
tags: [workflow, retrieval, reasoning, llm]
created: 2026-03-06
---

# LLM Coding Assistant

## Purpose

End-to-end implementation of an LLM-powered coding assistant that uses MCP tools for file system access, RAG for codebase retrieval, and an agentic loop for multi-step code generation and refactoring. Synthesized from: [[mcp_protocol|MCP Protocol]], [[agentic_coding|Agentic Coding]], [[repo_rag_for_code|Repo RAG for Code]], [[llm_code_generation|LLM Code Generation]], [[06_ai_engineering/04_rag_and_agents/rag_architecture|RAG Architecture]], [[06_ai_engineering/04_rag_and_agents/agentic_loop|Agentic Loop]].

## Architecture

```
IDE / CLI interface
    │
    │  User request: "Add auth middleware to the FastAPI app"
    ▼
Agentic loop (Claude / GPT-4o)
    │
    ├── Plan: read relevant files → understand structure → write patch
    │
    ├─► MCP Tool calls:
    │     ├── search_code("middleware")     ← ripgrep search
    │     ├── read_file("src/main.py")      ← read current code
    │     ├── read_file("tests/test_api.py")
    │     └── run_tests("tests/")           ← verify current tests pass
    │
    ├─► RAG retrieval:
    │     ├── embed query "FastAPI auth middleware JWT"
    │     ├── vector search → top-k code chunks
    │     └── inject into context: relevant patterns from codebase + docs
    │
    ├─► LLM generates patch (structured output via instructor)
    │
    └─► Tool call: write_file("src/middleware/auth.py", code)
             + run_tests("tests/") ← verify tests still pass

Final response: diff + explanation to user
```

**Component stack:**
| Layer | Technology | Role |
|-------|-----------|------|
| LLM | Claude 3.5 / GPT-4o | Code generation, planning |
| Agent framework | MCP + custom loop | Tool orchestration |
| Code retrieval | FAISS + code embeddings | Codebase-aware context |
| Structured output | instructor + Pydantic | Type-safe LLM responses |
| Sandboxing | subprocess + path guards | Safe tool execution |

### Examples

**1. Code RAG index (`rag/indexer.py`):**
```python
import ast, pathlib
from sentence_transformers import SentenceTransformer
import faiss, numpy as np, json

MODEL = SentenceTransformer("jinaai/jina-embeddings-v2-base-code")

def chunk_python_file(path: pathlib.Path) -> list[dict]:
    """Split a Python file into function/class chunks."""
    source = path.read_text()
    try:
        tree = ast.parse(source)
    except SyntaxError:
        return [{"path": str(path), "code": source, "name": str(path.name)}]
    chunks = []
    for node in ast.walk(tree):
        if isinstance(node, (ast.FunctionDef, ast.AsyncFunctionDef, ast.ClassDef)):
            lines = source.splitlines()
            chunk_lines = lines[node.lineno - 1 : node.end_lineno]
            chunks.append({
                "path": str(path),
                "name": node.name,
                "code": "\n".join(chunk_lines),
                "lineno": node.lineno,
            })
    return chunks or [{"path": str(path), "code": source, "name": str(path.name)}]

def build_index(repo_root: str, index_path: str):
    repo = pathlib.Path(repo_root)
    all_chunks = []
    for py_file in repo.rglob("*.py"):
        if "/.venv" in str(py_file) or "/node_modules" in str(py_file):
            continue
        all_chunks.extend(chunk_python_file(py_file))

    texts = [f"# {c['path']}:{c['name']}\n{c['code']}" for c in all_chunks]
    embeddings = MODEL.encode(texts, normalize_embeddings=True, show_progress_bar=True)
    embeddings = np.array(embeddings, dtype="float32")

    index = faiss.IndexFlatIP(embeddings.shape[1])   # inner product = cosine on normalised
    index.add(embeddings)

    faiss.write_index(index, f"{index_path}.faiss")
    with open(f"{index_path}.meta.json", "w") as f:
        json.dump(all_chunks, f)

    print(f"Indexed {len(all_chunks)} chunks from {repo_root}")
```

**2. RAG retrieval (`rag/retriever.py`):**
```python
import faiss, json, numpy as np
from sentence_transformers import SentenceTransformer

class CodeRetriever:
    def __init__(self, index_path: str):
        self.model = SentenceTransformer("jinaai/jina-embeddings-v2-base-code")
        self.index = faiss.read_index(f"{index_path}.faiss")
        with open(f"{index_path}.meta.json") as f:
            self.chunks = json.load(f)

    def search(self, query: str, top_k: int = 5) -> list[dict]:
        emb = self.model.encode([query], normalize_embeddings=True)
        emb = np.array(emb, dtype="float32")
        scores, idxs = self.index.search(emb, top_k)
        return [
            {**self.chunks[i], "score": float(scores[0][j])}
            for j, i in enumerate(idxs[0])
            if i < len(self.chunks)
        ]
```

**3. Agentic loop with MCP tools (`agent/loop.py`):**
```python
import anthropic, json
from rag.retriever import CodeRetriever
from mcp_client import MCPClient   # wraps the MCP server we built

client = anthropic.Anthropic()
retriever = CodeRetriever("/workspace/.mcp/index")
mcp = MCPClient("/workspace/.mcp/server.py")

SYSTEM = """You are an expert software engineer working inside a code repository.
You have access to tools to read/write files, run tests, and search code.
Before writing code, read the relevant files. After writing, run tests to verify correctness.
"""

def run_agent(user_request: str, max_turns: int = 10):
    # Augment with RAG context
    rag_hits = retriever.search(user_request, top_k=4)
    rag_context = "\n\n".join(
        f"```python\n# {h['path']}:{h['name']}\n{h['code']}\n```" for h in rag_hits
    )

    messages = [{
        "role": "user",
        "content": f"{user_request}\n\nRelevant codebase context:\n{rag_context}"
    }]
    tools = mcp.list_tools_as_anthropic_schema()

    for _ in range(max_turns):
        response = client.messages.create(
            model="claude-3-5-sonnet-20241022",
            max_tokens=4096,
            system=SYSTEM,
            tools=tools,
            messages=messages,
        )
        messages.append({"role": "assistant", "content": response.content})

        if response.stop_reason == "end_turn":
            # Extract final text response
            for block in response.content:
                if block.type == "text":
                    return block.text
            return "Done"

        # Handle tool calls
        tool_results = []
        for block in response.content:
            if block.type == "tool_use":
                result = mcp.call_tool(block.name, block.input)
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": result,
                })
        messages.append({"role": "user", "content": tool_results})

    return "Max turns reached"
```

**4. CLI entrypoint (`main.py`):**
```python
import sys
from agent.loop import run_agent

if __name__ == "__main__":
    request = " ".join(sys.argv[1:]) or input("Request: ")
    print(run_agent(request))
```

**Usage:**
```bash
# Build codebase index
python -m rag.indexer --repo /workspace --output /workspace/.mcp/index

# Run assistant
python main.py "Add rate limiting middleware to the FastAPI app"
python main.py "Write a pytest test for the predict endpoint"
python main.py "Refactor UserService to use async/await throughout"
```

## References

- [Model Context Protocol Documentation](https://modelcontextprotocol.io/)

## Links
- [[mcp_protocol|MCP Protocol]]
- [[agentic_coding|Agentic Coding]]
- [[repo_rag_for_code|Repo RAG for Code]]
- [[llm_code_generation|LLM Code Generation]]
- [[mcp_server_implementation|MCP Server Implementation]]
- [[pytest_testing_patterns|pytest Testing Patterns]]
- [[06_ai_engineering/04_rag_and_agents/rag_architecture|RAG Architecture]]
- [[06_ai_engineering/04_rag_and_agents/agentic_loop|Agentic Loop]]
- [[06_ai_engineering/04_rag_and_agents/function_calling|Function Calling]]
