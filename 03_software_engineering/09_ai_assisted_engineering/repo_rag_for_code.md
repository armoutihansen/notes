---
layer: 03_software_engineering
type: engineering
tool: RAG/Embeddings
status: growing
tags: [rag, embeddings, code-search, retrieval, vector-search]
created: 2026-03-05
---

# Repo RAG for Code

## Purpose

Retrieval-Augmented Generation (RAG) applied to codebases augments LLM prompts with semantically relevant code snippets retrieved from an embedding index. This overcomes the context window limit — a 10 million line monorepo cannot fit in any prompt — and reduces hallucination by grounding responses in actual code. Core use cases: codebase Q&A, intelligent code search, documentation generation, and AI assistants that understand your specific project.

## Architecture

### Pipeline Overview

```
            Index Phase (offline / periodic)
            ───────────────────────────────
  Source code
       │
       ▼
  ┌──────────┐    ┌─────────────────┐    ┌──────────────────┐
  │  Chunker │───▶│ Embedding model │───▶│  Vector store    │
  │ (AST /   │    │ (CodeBERT /     │    │ (Chroma / Weaviate│
  │  file /  │    │  voyage-code /  │    │  / pgvector)     │
  │  function│    │  text-embed-3)  │    └──────────────────┘
  └──────────┘    └─────────────────┘
  
            Query Phase (online)
            ──────────────────────
  User query
       │
       ▼
  ┌────────────┐    ┌──────────────────┐    ┌──────────────┐
  │  Embed     │───▶│  ANN search      │───▶│  Reranker    │
  │  query     │    │  (cosine / IP)   │    │  (optional)  │
  └────────────┘    └──────────────────┘    └──────┬───────┘
                                                   │
                                                   ▼
                                          ┌────────────────┐
                                          │  LLM prompt    │
                                          │  = query +     │
                                          │  top-k chunks  │
                                          └────────────────┘
```

## Implementation Notes

### Chunking Strategies for Code

The right chunking granularity determines retrieval quality. Too coarse: retrieved chunks contain mostly irrelevant code. Too fine: individual lines lack context.

**File-level chunking**
- Each file is one chunk.
- Pro: full context, no boundary issues.
- Con: long files blow out context windows; dilutes relevance score.
- Best for: small files, config files, `__init__.py`, READMEs.

**Function/method-level chunking**
- Parse source into functions using tree-sitter or the language's AST.
- Each function (signature + body + docstring) is one chunk.
- Pro: tight semantic unit — best precision for "how does X work" queries.
- Con: inter-function dependencies require multiple retrievals.
- Best for: the default strategy for most codebases.

```python
import tree_sitter_python as tspython
from tree_sitter import Language, Parser

PY_LANGUAGE = Language(tspython.language())
parser = Parser(PY_LANGUAGE)

def extract_functions(source: str) -> list[dict]:
    tree = parser.parse(source.encode())
    functions = []
    for node in tree.root_node.children:
        if node.type in ("function_definition", "decorated_definition"):
            functions.append({
                "text": source[node.start_byte:node.end_byte],
                "start_line": node.start_point[0],
                "end_line": node.end_point[0],
            })
    return functions
```

**AST-aware sliding window**
- Slide a token window across the file, but only break at AST node boundaries (never mid-function).
- Pro: handles large functions; preserves context.
- Con: complex to implement correctly.

**Class-level chunking**
- Entire class (including all methods) as one chunk.
- Best for OOP codebases where method interaction is key to understanding.

**Overlap chunking**
- Add N lines of overlap between adjacent chunks to avoid slicing a relevant pattern across a boundary.
- Typical overlap: 10–20% of chunk size.

### Code-Specific Embedding Models

General-purpose text embeddings underperform on code because they lack exposure to syntax, identifier semantics, and cross-language patterns.

| Model | Provider | Context | Notes |
|-------|----------|---------|-------|
| `voyage-code-2` | Voyage AI | 16K tokens | State-of-art for code retrieval (2024) |
| `text-embedding-3-large` | OpenAI | 8K tokens | Strong general + code, widely available |
| `CodeBERT` | Microsoft/HF | 512 tokens | Open, fine-tunable, older |
| `UniXcoder` | Microsoft/HF | 512 tokens | Multi-task code model, open |
| `codesage-large` | Salesforce/HF | 2K tokens | Open, strong on code search |
| `jina-embeddings-v2-base-code` | Jina | 8K tokens | Open, multilingual code |

For private codebases with domain-specific vocabulary, fine-tuning a smaller open model on in-house code pairs (function + docstring as positive pairs) meaningfully improves retrieval.

### Vector Store Options

| Store | Deployment | Notes |
|-------|-----------|-------|
| **Chroma** | Local / Docker | Simplest setup, good for dev |
| **pgvector** | Postgres extension | If already using Postgres |
| **Weaviate** | Docker / Cloud | Hybrid BM25 + vector built-in |
| **Qdrant** | Docker / Cloud | Fast, Rust-based, good filtering |
| **Pinecone** | SaaS only | Managed, no infra |
| **FAISS** | In-process | Fastest for read-heavy, no persistence |

For code search, **hybrid retrieval** (BM25 keyword + vector semantic) consistently outperforms either alone — exact identifier names are better matched by BM25, semantic intent by vector.

```python
# Hybrid retrieval with Weaviate
results = collection.query.hybrid(
    query="function that validates email addresses",
    alpha=0.5,           # 0 = pure BM25, 1 = pure vector
    limit=10,
    return_metadata=wvc.query.MetadataQuery(score=True)
)
```

### Building a Codebase Index

```python
import os
from pathlib import Path
from chromadb import PersistentClient
from chromadb.utils.embedding_functions import OpenAIEmbeddingFunction

SUPPORTED_EXTENSIONS = {".py", ".ts", ".go", ".java", ".rs", ".cpp", ".c", ".md"}

def index_repository(repo_path: str, collection_name: str = "codebase"):
    client = PersistentClient(path="./.codebase_index")
    embed_fn = OpenAIEmbeddingFunction(
        api_key=os.environ["OPENAI_API_KEY"],
        model_name="text-embedding-3-large"
    )
    collection = client.get_or_create_collection(
        name=collection_name,
        embedding_function=embed_fn,
        metadata={"hnsw:space": "cosine"}
    )

    documents, metadatas, ids = [], [], []

    for path in Path(repo_path).rglob("*"):
        if path.suffix not in SUPPORTED_EXTENSIONS:
            continue
        if any(part.startswith(".") or part in ("node_modules", "__pycache__", "venv")
               for part in path.parts):
            continue

        source = path.read_text(errors="ignore")
        chunks = chunk_by_function(source, str(path))  # or other strategy

        for i, chunk in enumerate(chunks):
            chunk_id = f"{path}:{i}"
            documents.append(chunk["text"])
            metadatas.append({
                "file": str(path),
                "start_line": chunk["start_line"],
                "end_line": chunk["end_line"],
                "language": path.suffix.lstrip(".")
            })
            ids.append(chunk_id)

    # Upsert in batches
    batch_size = 100
    for i in range(0, len(documents), batch_size):
        collection.upsert(
            documents=documents[i:i+batch_size],
            metadatas=metadatas[i:i+batch_size],
            ids=ids[i:i+batch_size]
        )

    print(f"Indexed {len(documents)} chunks from {repo_path}")
    return collection
```

### Query and Generation

```python
def answer_code_question(query: str, collection, llm_client, top_k: int = 8) -> str:
    # Retrieve relevant chunks
    results = collection.query(
        query_texts=[query],
        n_results=top_k,
        include=["documents", "metadatas", "distances"]
    )

    chunks = results["documents"][0]
    metas = results["metadatas"][0]

    # Format context
    context_parts = []
    for chunk, meta in zip(chunks, metas):
        context_parts.append(
            f"File: {meta['file']} (lines {meta['start_line']}-{meta['end_line']})\n"
            f"```{meta['language']}\n{chunk}\n```"
        )
    context = "\n\n---\n\n".join(context_parts)

    # Prompt LLM
    response = llm_client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=2048,
        system="You are a code assistant. Answer questions based only on the provided code context. "
               "Cite file paths when referencing code.",
        messages=[{
            "role": "user",
            "content": f"Context from codebase:\n\n{context}\n\nQuestion: {query}"
        }]
    )
    return response.content[0].text
```

### IDE and Tool Integration

**Continue (VS Code / JetBrains)**

`config.json` to enable codebase indexing:
```json
{
  "contextProviders": [
    {
      "name": "codebase",
      "params": {
        "nRetrieve": 25,
        "nFinal": 5,
        "useReranking": true
      }
    },
    {
      "name": "code",
      "params": {}
    }
  ],
  "embeddingsProvider": {
    "provider": "openai",
    "model": "text-embedding-3-large",
    "apiKey": "$OPENAI_API_KEY"
  }
}
```

**Cursor** — built-in codebase indexing. `@codebase` in chat queries the index. Settings → Features → Codebase Indexing to configure.

**Custom integration via LSP + embeddings** — combine LSP for symbol resolution (go-to-definition, references) with embedding-based semantic search. LSP gives you the structural graph; embeddings give you semantic similarity. Together they enable queries like "find all functions similar to `parse_user_input` that handle authentication."

### Limitations

**Stale index** — code changes continuously. Without incremental re-indexing, the vector store drifts from the actual codebase. Solutions: watch filesystem events and re-index changed files, or run a nightly re-index job.

**Hallucinated APIs** — even with retrieved context, models may blend retrieved code with memorised patterns and invent non-existent methods. Always verify generated code against retrieved chunks and run tests.

**Embedding model mismatch** — changing the embedding model requires re-indexing everything. Pin the model version; store it in collection metadata.

**Long functions** — a 500-line function hits model context limits. Either split at logical sub-sections or summarise with a secondary LLM call and store the summary as a separate searchable document.

**Cross-file reasoning** — RAG retrieves isolated chunks. Understanding how `ServiceA.process()` calls `RepositoryB.save()` requires retrieving multiple related chunks and the model connecting them. Re-ranking with MMR (Maximal Marginal Relevance) increases diversity and helps surface related-but-not-identical code.

## Trade-offs

| Approach | Pro | Con |
|----------|-----|-----|
| Function-level chunks | High precision | Miss inter-function patterns |
| File-level chunks | Full context | Low precision, token waste |
| BM25 only | Exact match, fast, no model needed | Misses semantic similarity |
| Vector only | Semantic similarity | Misses exact identifiers |
| Hybrid (BM25 + vector) | Best of both | More complex pipeline |
| In-process FAISS | Fastest search | No persistence, no filtering |
| Managed vector DB | Easy to run | Cost, data leaves machine |
| Fine-tuned embeddings | Best quality for your code | Training cost, maintenance |

## References

- [CodeBERT: Pre-Trained Models for Programming and Natural Language (Feng et al. 2020)](https://arxiv.org/abs/2002.08155)
- [Voyage Code embeddings](https://docs.voyageai.com/docs/embeddings)
- [Continue codebase indexing](https://docs.continue.dev/walkthroughs/codebase-embeddings)
- [LlamaIndex code splitter](https://docs.llamaindex.ai/en/stable/api_reference/node_parsers/code/)
- [tree-sitter](https://tree-sitter.github.io/tree-sitter/)

## Links
- [[mcp_protocol|MCP]]
- [[agentic_coding|Agentic Coding]]
- [[05_ai_engineering/03_rag_and_agents/rag_architecture|RAG Architecture]]
- [[04_databases_and_storage/nosql_patterns|NoSQL Patterns (vector DBs)]]
