---
layer: 06_applications
type: application
status: growing
tags: [vector-database, faiss, chroma, similarity-search, ann, index-types, embeddings]
created: 2026-05-10
---

# Vector Database Retrieval

## Purpose

Vector databases enable semantic similarity search over dense embeddings. This note covers the two most commonly used options — FAISS (pure-speed, CPU/GPU, no metadata) and Chroma (metadata filtering, persistent, developer-friendly) — and explains when to use each, index type selection, batch upsert, and k-NN query patterns.

### Examples

**FAISS**: In-memory ANN index for a static 10M-vector embedding cache where query speed is the only concern and there are no metadata filters.

**Chroma**: Hybrid search over a document Q&A corpus where you need to filter by `source`, `date`, or `category` alongside semantic similarity.

---

## Architecture

```
Text / image / audio
        ↓
   Embedding model (produces dense vector)
        ↓
   ┌────────────────────┬──────────────────────────┐
   │        FAISS       │          Chroma           │
   │   pure ANN search  │  vector + metadata filter │
   │   in-memory/mmap   │  persistent, HTTP server  │
   │   no persistence   │  with SDK                 │
   └────────────────────┴──────────────────────────┘
        ↓                          ↓
   k nearest vectors          top-k + filtered results
```

---

## FAISS — Index Types and Basic Search

```bash
pip install faiss-cpu    # CPU
pip install faiss-gpu    # NVIDIA GPU
```

```python
import faiss
import numpy as np

d = 384          # embedding dimension (e.g., all-MiniLM-L6-v2)
n = 100_000      # number of vectors

# Generate random embeddings (replace with real vectors)
np.random.seed(42)
vectors = np.random.randn(n, d).astype("float32")
faiss.normalize_L2(vectors)   # required for cosine similarity with IndexFlatIP

# --- Flat index (exact search — ground truth, small datasets) ---
index_flat = faiss.IndexFlatIP(d)   # Inner product = cosine after L2 normalisation
index_flat.add(vectors)
print(f"Index size: {index_flat.ntotal}")

# Query
query = np.random.randn(1, d).astype("float32")
faiss.normalize_L2(query)
D, I = index_flat.search(query, k=5)
print("Top-5 indices:", I[0], "Scores:", D[0])
```

---

## FAISS — IVF Index (Scale to Millions)

```python
# IVF (Inverted File) index — approximate, fast, scales to millions of vectors
nlist    = 256      # number of Voronoi cells (rule of thumb: sqrt(n))
quantizer = faiss.IndexFlatIP(d)
index_ivf = faiss.IndexIVFFlat(quantizer, d, nlist, faiss.METRIC_INNER_PRODUCT)

# IVF must be trained before adding vectors
index_ivf.train(vectors)
index_ivf.add(vectors)

# nprobe: number of cells to search (higher = better recall, slower)
index_ivf.nprobe = 16
D, I = index_ivf.search(query, k=5)
```

---

## FAISS — HNSW Index (Best for Sub-100ms Latency)

```python
# HNSW — graph-based index; best recall/speed trade-off; no GPU needed
M        = 32      # connections per node (higher = better recall, more memory)
ef_const = 200     # construction-time search depth
index_hnsw = faiss.IndexHNSWFlat(d, M, faiss.METRIC_INNER_PRODUCT)
index_hnsw.hnsw.efConstruction = ef_const
index_hnsw.hnsw.efSearch = 64   # search-time depth (increase for better recall)
index_hnsw.add(vectors)
D, I = index_hnsw.search(query, k=5)
```

---

## FAISS — Persist and Load

```python
# Save to disk
faiss.write_index(index_hnsw, "hnsw_index.faiss")

# Load from disk
index_loaded = faiss.read_index("hnsw_index.faiss")
```

---

## Chroma — Metadata Filtering + Hybrid Search

```bash
pip install chromadb
```

```python
import chromadb
from chromadb.utils.embedding_functions import SentenceTransformerEmbeddingFunction

embed_fn = SentenceTransformerEmbeddingFunction(model_name="all-MiniLM-L6-v2")

client     = chromadb.PersistentClient(path="./chroma_db")
collection = client.get_or_create_collection(
    name="documents",
    embedding_function=embed_fn,
    metadata={"hnsw:space": "cosine"},
)

# --- Batch upsert (add or overwrite) ---
collection.upsert(
    ids=["doc_1", "doc_2", "doc_3"],
    documents=[
        "GDPR requires explicit consent for personal data processing.",
        "The refund policy allows returns within 30 days with receipt.",
        "Machine learning models can overfit when trained on small datasets.",
    ],
    metadatas=[
        {"source": "legal", "date": "2024-01-15", "lang": "en"},
        {"source": "policy", "date": "2024-03-01", "lang": "en"},
        {"source": "ml_guide", "date": "2024-02-20", "lang": "en"},
    ],
)

# --- k-NN query ---
results = collection.query(
    query_texts=["What are the GDPR rules?"],
    n_results=3,
)
print(results["documents"])

# --- Metadata-filtered query ---
results_filtered = collection.query(
    query_texts=["refund process"],
    n_results=2,
    where={"source": {"$eq": "policy"}},   # pre-filter by metadata
)
```

---

## FAISS vs. Chroma — Decision Table

| Criteria | FAISS | Chroma |
|---|---|---|
| Scale | Billions of vectors (GPU) | Up to ~10M vectors comfortably |
| Metadata filtering | ❌ Manual via external index | ✅ Built-in `where` filter |
| Persistence | Manual serialization | ✅ Automatic (SQLite / HTTP) |
| HTTP server mode | ❌ Embedded only | ✅ `chromadb.HttpClient()` |
| GPU acceleration | ✅ | ❌ |
| Hybrid (dense + sparse) | Manual | Via LlamaIndex / LangChain |
| Latency (10M vectors) | <1 ms (HNSW) | ~5–50 ms |
| Best for | Pure speed, offline batch | Dev to production, metadata filtering |

---

## Batch Upsert Performance Tips

```python
# FAISS: add in batches, not one-by-one
BATCH_SIZE = 10_000
for i in range(0, len(vectors), BATCH_SIZE):
    index_hnsw.add(vectors[i:i+BATCH_SIZE])

# Chroma: batch upserts (default max 5461 per call)
BATCH_SIZE = 5000
for i in range(0, len(ids), BATCH_SIZE):
    collection.upsert(
        ids=ids[i:i+BATCH_SIZE],
        documents=docs[i:i+BATCH_SIZE],
        metadatas=metadatas[i:i+BATCH_SIZE],
    )
```

---

## Links

**AI Engineering**
- [[05_ai_engineering/03_rag_and_agents/vector_stores|Vector Stores]] — FAISS, Chroma, pgvector, Qdrant comparison and selection criteria
- [[05_ai_engineering/03_rag_and_agents/rag_architecture|RAG Architecture]] — how vector retrieval fits into the full RAG pipeline

**System Patterns**
- [[chroma_vector_store|Chroma Vector Store]] — Chroma-specific patterns including HTTP server mode
- [[rag_pipeline_pattern|RAG Pipeline Pattern]] — uses Chroma for the full document Q&A pipeline
