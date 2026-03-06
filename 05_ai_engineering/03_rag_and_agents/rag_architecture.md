---
layer: 05_ai_engineering
type: ai_system
status: growing
tags: [rag, retrieval, vector-store, chunking, embedding, llm, hallucination]
created: 2026-03-05
---

# RAG Architecture

## Goal

Augment LLM generation with retrieved relevant context to reduce hallucination and enable access to domain knowledge without fine-tuning. RAG decouples the knowledge store from the model parameters: knowledge can be updated, versioned, and audited without retraining. It is the primary pattern for building knowledge-grounded LLM applications — document Q&A, enterprise search, code assistants over large codebases, and customer support systems.

## Architecture

### Naive RAG (baseline pipeline)
```
User query
  → Embed query (same model as index)
  → Vector similarity search → top-k chunks
  → Prompt: [system] + [top-k chunks] + [user query]
  → LLM generates answer
```
Simple to implement but suffers from retrieval precision issues and context window packing inefficiency.

### Advanced RAG
Extensions to naive RAG that address its failure modes:

- **Query rewriting**: Reformulate the user query into a more retrieval-friendly form using a prompt. Helps when the query is conversational or ambiguous.
- **HyDE (Hypothetical Document Embeddings)**: Generate a hypothetical answer to the query, embed the hypothetical answer (not the query), and retrieve against that embedding. Bridges the query-document embedding distribution gap.
- **Multi-query retrieval**: Generate 3–5 paraphrases of the query, retrieve for each, deduplicate results. Improves recall at the cost of extra embedding + search calls.
- **Parent-child chunking**: Index small chunks (child) for precise retrieval, but return the larger surrounding context (parent) to the LLM. Balances retrieval precision with generation context sufficiency.
- **Re-ranking**: After ANN retrieval (top-k=20–50), run a cross-encoder reranker to score all candidates against the query and return only the top-n (e.g., n=5). Cross-encoders see (query, chunk) jointly and are significantly more accurate than bi-encoders.
- **Metadata filtering**: Pre-filter the vector search by structured metadata (date, source, document type, author) before ANN search. Reduces noise and improves precision for scoped queries.

### Modular RAG
Composition of retrieval, routing, and generation modules that can be swapped or chained. Includes:
- **Routing**: Choose between different retrieval strategies or knowledge sources based on query classification.
- **Fusion**: Merge results from multiple retrievers (dense + sparse) using Reciprocal Rank Fusion (RRF).
- **Iterative retrieval**: The LLM generates intermediate reasoning steps that trigger additional retrieval rounds (IRCoT — Interleaved Retrieval + CoT).

### Indexing Pipeline
```
Raw documents
  → Document loading (PDFs, HTML, Markdown, code)
  → Chunking (fixed-size 512 tokens with 10% overlap | semantic | recursive character split)
  → Embedding (OpenAI text-embedding-ada-002 | sentence-transformers/bge-large-en)
  → Vector store upsert (with metadata: source, chunk_id, page, timestamp)
```

## Components

**Document ingestion**: LangChain and LlamaIndex provide 300+ document loaders (PDF via pypdf, HTML via BeautifulSoup, Notion, Confluence, GitHub). Normalise to plain text before chunking.

**Chunking strategy**: Chunk size is a critical hyperparameter. Smaller chunks (128–256 tokens) increase retrieval precision but lose surrounding context. Larger chunks (512–1024 tokens) preserve context but reduce precision. Recursive character splitting (LangChain `RecursiveCharacterTextSplitter`) handles mixed content well. Semantic chunking splits at paragraph/section boundaries.

**Embedding model**: The embedding model determines the quality of the vector space. `text-embedding-ada-002` (OpenAI) is a reliable baseline. `bge-large-en-v1.5` (BAAI) outperforms ada-002 on MTEB benchmarks and is freely available. The same model must be used at indexing and query time.

**Vector store**: See [[vector_stores|Vector Stores]] for detailed comparison. Chroma for development; Pinecone, Weaviate, or pgvector for production.

**Retriever**: Dense retriever (embedding similarity, ANN via HNSW); sparse retriever (BM25 — keyword overlap); hybrid retriever (dense + sparse with RRF). Dense excels at semantic similarity; sparse excels at exact keyword match. Hybrid provides the best recall.

**Reranker**: Cross-encoder models (e.g., `cross-encoder/ms-marco-MiniLM-L-6-v2`) significantly improve ranking quality at the cost of ~50–200ms latency. Run after ANN retrieval on the top-20–50 candidates.

**Generator**: The main LLM (GPT-4o, Claude 3.5, Llama 3). Prompt template must instruct the model to use only retrieved context and to cite sources when possible.

**Citation/attribution layer**: Map model-generated sentences back to source chunks via overlap matching or a separate attribution model. Required for high-trust applications (legal, medical, financial).

## Evaluation

**Ragas framework** provides automated evaluation metrics without human annotation:
- **Faithfulness**: What proportion of the generated answer is supported by the retrieved context? (Measures hallucination.)
- **Answer Relevancy**: How relevant is the generated answer to the question?
- **Context Precision**: Of the retrieved chunks, what fraction was actually relevant?
- **Context Recall**: What fraction of the relevant information was retrieved?

**Retriever-level metrics** (requires a labeled dataset with relevant document annotations):
- Hit Rate: Is at least one relevant chunk in top-k?
- Mean Reciprocal Rank (MRR): Where does the first relevant chunk rank?

**Generator-level metrics**:
- Faithfulness score (Ragas) vs reference answers
- ROUGE/BLEU against reference answers (weak signal; prefer LLM-as-judge)

## Failure Modes

**Retrieval failure**: The correct chunks are not in top-k. Causes: embedding space mismatch, poor chunking (splits across relevant content), wrong metadata filter, inadequate index coverage. Diagnosis: check hit rate on a labeled eval set.

**Lost in the middle**: LLMs attend primarily to the beginning and end of the context window. Chunks in the middle of a long prompt are underutilised. Mitigation: use reranking to push the most relevant chunks to position 0; use fewer, higher-quality chunks.

**LLM ignores retrieved context**: Model relies on parametric memory instead of retrieved chunks, especially when chunks conflict with training data or when the query is interpreted as a general knowledge question. Mitigation: explicit prompt instruction ("Answer based only on the provided context."), faithfulness fine-tuning.

**Hallucination on chunks**: Model reads the chunk correctly but generates unsupported additions or extrapolations. Faithfulness score catches this. Mitigation: lower temperature, citation enforcement.

**Stale index**: New or updated documents not indexed. The LLM may answer correctly with retrieved context but the context is outdated. Mitigation: incremental indexing pipeline with change detection; metadata `updated_at` timestamp filtering.

**Chunk boundary splits reasoning context**: A key fact is split across two chunks; neither chunk alone is sufficient. Mitigation: overlapping windows, parent-child chunking, semantic chunking at natural boundaries.

## Cost / Latency

| Stage | Cost Driver | Typical Latency |
|---|---|---|
| Embedding (query) | 1 API call | ~20ms |
| ANN search | Vector DB | ~10–50ms |
| Reranking (cross-encoder) | Compute | ~50–200ms |
| LLM generation | Token count | ~200–2000ms |
| **Total (with reranking)** | | **~300–800ms** |

Embedding cost at index time is O(D × cost_per_token) — a one-time expense per document corpus. At query time, only the query embedding is needed: O(1). HNSW vector search scales as O(log N) with number of documents. Reranking adds ~50–200ms regardless of corpus size. Without reranking, latency is typically 200–400ms.

## Links
- [[vector_stores|Vector Stores]]
- [[agentic_loop|Agentic Loop]]
- [[prompt_injection_and_guardrails|Prompt Injection and Guardrails]]
- [[prompting_strategies|Prompting Strategies]]
- [[caching_strategies|Caching Strategies]]
