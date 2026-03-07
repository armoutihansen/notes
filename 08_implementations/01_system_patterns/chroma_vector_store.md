---
layer: 08_implementations
type: application
status: growing
tags: [pattern, retrieval, llm]
created: 2026-03-06
---

# Chroma Vector Store

## Purpose

Implementation patterns for using Chroma as an embedding database in RAG and semantic search applications. Chroma provides a simple 4-function API (add, query, get, delete), persistent storage, rich metadata filtering, and seamless LangChain/LlamaIndex integration. It is the standard choice for local development and open-source RAG projects before graduating to managed alternatives such as Pinecone or Qdrant.

### Examples

- RAG over internal documentation
- Semantic search across product catalogue
- Long-term agent memory store

## Architecture

**Persistence and client setup:**
```python
import chromadb
from chromadb.utils import embedding_functions

# Persistent local storage — survives process restarts
client = chromadb.PersistentClient(path="./chroma_db")

# OpenAI embeddings (recommended for quality)
openai_ef = embedding_functions.OpenAIEmbeddingFunction(
    api_key="sk-...",
    model_name="text-embedding-3-small"
)

# HuggingFace embeddings (free, local)
hf_ef = embedding_functions.HuggingFaceEmbeddingFunction(
    api_key="hf-...",
    model_name="sentence-transformers/all-mpnet-base-v2"
)

collection = client.get_or_create_collection(
    name="docs",
    embedding_function=openai_ef,
    metadata={"hnsw:space": "cosine"}  # cosine similarity
)
```

**Ingest documents with metadata:**
```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

# Chunk documents
splitter = RecursiveCharacterTextSplitter(chunk_size=512, chunk_overlap=50)
chunks = splitter.split_documents(raw_docs)

# Add to Chroma — embeddings generated automatically
collection.add(
    documents=[c.page_content for c in chunks],
    metadatas=[c.metadata for c in chunks],
    ids=[f"doc_{i}" for i in range(len(chunks))]
)
```

**Similarity search with metadata filtering:**
```python
# Basic query — returns top-5 most relevant chunks
results = collection.query(
    query_texts=["What is the refund policy?"],
    n_results=5
)

# With metadata pre-filter — narrows search space before ANN
results = collection.query(
    query_texts=["pricing information"],
    n_results=3,
    where={
        "$and": [
            {"source": {"$eq": "docs"}},
            {"page": {"$gte": 1}}
        ]
    }
)

# Access results
for doc, meta, dist in zip(
    results["documents"][0],
    results["metadatas"][0],
    results["distances"][0]
):
    print(f"[score={1-dist:.3f}] {meta} → {doc[:100]}")
```

**LangChain integration (RAG retriever):**
```python
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings

vectorstore = Chroma(
    client=client,
    collection_name="docs",
    embedding_function=OpenAIEmbeddings()
)

retriever = vectorstore.as_retriever(
    search_type="mmr",          # Maximum Marginal Relevance — diversity-aware
    search_kwargs={"k": 5, "fetch_k": 20}
)
docs = retriever.invoke("How do I cancel my subscription?")
```

**Server mode (multi-process / production):**
```bash
# Start Chroma HTTP server
chroma run --path ./chroma_db --port 8000

# Connect from application
import chromadb
client = chromadb.HttpClient(host="localhost", port=8000)
```

**Updating and deleting documents:**
```python
# Update — re-embeds if documents changed
collection.update(ids=["doc_5"], documents=["Updated content..."])

# Delete stale entries by metadata filter
collection.delete(where={"source": {"$eq": "outdated_source"}})
```

**Choosing embedding model:**

| Model | Dim | Latency | Cost | Quality |
|---|---|---|---|---|
| `text-embedding-3-small` | 1536 | ~50ms | $0.02/1M tokens | High |
| `all-MiniLM-L6-v2` (HF) | 384 | ~10ms local | Free | Medium |
| `all-mpnet-base-v2` (HF) | 768 | ~20ms local | Free | Medium-high |

**When to graduate to managed alternatives:**
- Collection size > 1M vectors → consider Qdrant or Weaviate
- Multi-tenant production SaaS → Pinecone (fully managed)
- Existing PostgreSQL stack → `pgvector` extension

## Links

**AI Engineering**
- [[vector_stores|Vector Stores]] — overview of vector store options (Chroma, FAISS, pgvector, Qdrant)
- [[rag_architecture|RAG Architecture]] — retrieval-augmented generation pipeline this pattern serves

**System Patterns**
- [[model_serving_with_fastapi|Model Serving with FastAPI]] — wrap Chroma retrieval behind a REST endpoint
- [[rag_pipeline_pattern|RAG Pipeline Pattern]] — full chunking → embedding → retrieval → generation pipeline
- [[vector_database_retrieval|Vector Database Retrieval]] — FAISS vs Chroma comparison and index-type selection
