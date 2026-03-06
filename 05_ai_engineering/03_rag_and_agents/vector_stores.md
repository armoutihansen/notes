---
layer: 05_ai_engineering
type: engineering
tool: chroma
status: growing
tags: [vector-store, chroma, faiss, pgvector, pinecone, hnsw, similarity-search, rag]
created: 2026-03-05
---

# Vector Stores

## Purpose

Efficient similarity search over high-dimensional dense vectors for RAG and semantic search applications. Vector stores index embedding vectors such that nearest-neighbour queries (find the k most similar vectors to a query vector) can be answered in sub-linear time over millions or billions of vectors. They are the retrieval backbone of RAG systems, semantic search engines, recommendation systems, and duplicate detection pipelines.

## Architecture

### Indexing Algorithms

**HNSW (Hierarchical Navigable Small Worlds)**
The dominant algorithm for approximate nearest-neighbour (ANN) search in most production vector stores. Builds a layered graph structure where each node connects to its nearest neighbours at multiple scales. At query time, graph traversal finds approximate nearest neighbours in O(log N) with high recall (typically 95–99% with default parameters). Memory-intensive: stores the full graph structure in RAM. Default in Chroma, Weaviate, Qdrant, and pgvector.

Key parameters:
- `M`: number of bi-directional links per node (default 16). Higher = better recall, more memory.
- `ef_construction`: size of the dynamic candidate list during build (default 200). Higher = better index quality, slower build.
- `ef_search`: size of the candidate list at query time. Higher = better recall, slower query.

**IVF (Inverted File Index)**
Partitions the vector space into `nlist` Voronoi cells via k-means clustering. At query time, searches only the nearest `nprobe` cells. Faster than HNSW for very large datasets (>10M vectors) when recall can be slightly compromised. FAISS default; supports GPU acceleration.

**Flat (exact search)**
Brute-force L2 or cosine similarity over all vectors. Guarantees exact results. Suitable for up to ~100k vectors before latency becomes unacceptable. `faiss.IndexFlatL2` — the reference baseline for measuring recall of approximate indices.

### Vector Store Options

**Chroma**
In-memory or persistent local vector store with a simple Python API. No infrastructure setup. Supports metadata filtering, collections, and multiple embedding functions. Best for prototyping, local development, notebooks, and small-scale applications (<1M vectors). Backed by DuckDB + Parquet for persistence.

**FAISS (Facebook AI Similarity Search)**
High-performance library for ANN search on CPU and GPU. No built-in persistence (must serialize/deserialize index manually), no metadata store, no server — it is a library, not a service. Supports GPU-accelerated search with massive throughput for batch workloads. Best for high-performance local inference pipelines and research.

**Pinecone**
Managed, serverless vector database. HTTP API, automatic scaling, built-in metadata filtering, namespaces for multi-tenancy. Zero infrastructure management. Best for production applications where operational overhead must be minimized. Costs per vector stored + per query.

**Weaviate**
Open-source vector database with a GraphQL API, built-in hybrid search (BM25 + dense), modules for automatic vectorization (OpenAI, Cohere, HuggingFace). Schema-based object model. Can be self-hosted or used as a managed cloud service.

**pgvector**
PostgreSQL extension that adds a `vector` column type and ANN index (HNSW or IVF). Allows joining vector similarity search with relational data in a single SQL query. Best when application data already lives in PostgreSQL and operational simplicity is valued. Recall is slightly lower than dedicated vector stores at large scale.

**Qdrant**
Open-source vector store with binary quantization, payload filtering, and a clean REST/gRPC API. Strong recall benchmarks. Good self-hosted alternative to Pinecone.

## Implementation Notes

**Chroma quickstart**
```python
import chromadb
from chromadb.utils import embedding_functions

client = chromadb.Client()  # in-memory, or chromadb.PersistentClient(path="./chroma_db")
emb_fn = embedding_functions.OpenAIEmbeddingFunction(api_key=OPENAI_API_KEY, model_name="text-embedding-ada-002")

collection = client.create_collection("knowledge_base", embedding_function=emb_fn)

collection.add(
    documents=["chunk text 1", "chunk text 2"],
    metadatas=[{"source": "doc1.pdf", "page": 1}, {"source": "doc1.pdf", "page": 2}],
    ids=["chunk_001", "chunk_002"]
)

results = collection.query(query_texts=["What is RAG?"], n_results=5)
```

**FAISS exact and approximate**
```python
import faiss, numpy as np

dim = 1536  # embedding dimension
# Exact search
index_flat = faiss.IndexFlatL2(dim)
index_flat.add(vectors)  # numpy array, shape (N, dim)
distances, indices = index_flat.search(query_vec, k=5)

# Approximate: IVF + quantization
quantizer = faiss.IndexFlatL2(dim)
index_ivf = faiss.IndexIVFFlat(quantizer, dim, nlist=100)
index_ivf.train(vectors)
index_ivf.add(vectors)
index_ivf.nprobe = 10  # search 10 cells
```

**Embedding model choice is the highest-leverage decision**
The quality of the embedding model dominates vector store choice in recall experiments. Evaluate on your domain using MTEB benchmarks or domain-specific labeled queries. `bge-large-en-v1.5` (BAAI) consistently outperforms `text-embedding-ada-002` on most benchmarks and is freely deployable.

**Metadata filtering before ANN search**
Most production stores support pre-filtering by metadata predicates before ANN search. Use this aggressively to scope retrieval to relevant document subsets (by date, department, document type, user permissions). Reduces both noise and query cost.

**Hybrid search (sparse + dense)**
Combine dense (embedding) and sparse (BM25) retrieval using Reciprocal Rank Fusion (RRF):
```
final_score = 1/(k + dense_rank) + 1/(k + sparse_rank)   # k=60 typical
```
Hybrid consistently outperforms either method alone, especially for queries with important keywords. Weaviate and Qdrant have built-in hybrid search; for others, run two retrievers and fuse manually.

**Index size estimation**
`N × dim × 4 bytes` (float32) for flat index. HNSW overhead ≈ `N × M × 8 bytes` additional. For 1M vectors at dim=1536: ~6GB flat, ~1.5GB HNSW overhead.

## Trade-offs

| Store | Best For | Limitations |
|---|---|---|
| Chroma | Dev/prototyping, local | Not production-scale; single-node |
| FAISS | High-performance local/batch | No persistence, no metadata, library only |
| pgvector | Data already in Postgres | Lower recall at very large scale |
| Pinecone | Production, zero-ops | Vendor lock-in, cost per vector |
| Weaviate | Self-hosted prod, hybrid search | Operationally heavier than Pinecone |
| Qdrant | Self-hosted prod, binary quantization | Less ecosystem tooling |

For most RAG prototypes: Chroma → pgvector (if Postgres is already in stack) or Pinecone (if managed is acceptable) for production.

## References

- Malkov & Yashunin (2020). *Efficient and Robust Approximate Nearest Neighbor Search Using HNSW*. IEEE TPAMI.
- Johnson et al. (2021). *Billion-Scale Similarity Search with GPUs*. IEEE Big Data.
- Chroma documentation: https://docs.trychroma.com
- FAISS documentation: https://faiss.ai
- pgvector: https://github.com/pgvector/pgvector

## Links
- [[rag_architecture|RAG Architecture]]
- [[agentic_loop|Agentic Loop]]
- [[nosql_patterns|NoSQL Patterns]]
