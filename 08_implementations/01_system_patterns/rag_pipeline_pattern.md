---
layer: 08_implementations
type: application
status: growing
tags: [pattern, retrieval, llm]
created: 2026-05-10
---

# RAG Pipeline Pattern

## Purpose

A Retrieval-Augmented Generation (RAG) pipeline grounds LLM responses in authoritative documents, reducing hallucination and enabling knowledge updates without model retraining. This note covers the full production pipeline: document loading → chunking → embedding → indexing → hybrid retrieval → reranking → LLM generation, using LlamaIndex and Chroma as the primary implementations.

### Examples

**Document Q&A**: Answer questions over a corpus of 10,000 internal policies; update the index nightly without retraining the LLM.

**Code assistant**: Index a repository's source code, docstrings, and README; retrieve relevant files and functions when the developer asks a question.

---

## Architecture

```
Documents (PDFs, HTML, Markdown, code)
        ↓
  [1] Document loaders (SimpleDirectoryReader, LlamaHub connectors)
        ↓
  [2] Chunking (token-size, sentence, semantic, parent-child)
        ↓
  [3] Embedding (OpenAI ada-002, all-MiniLM, e5-large-v2)
        ↓
  [4] Vector index (Chroma, FAISS, pgvector)
        ↓
  [5] Retrieval: dense ANN + optional sparse BM25 (hybrid)
        ↓
  [6] Reranking (cross-encoder or Cohere Rerank)
        ↓
  [7] LLM generation with retrieved context
```

---

## Setup

```bash
pip install llama-index llama-index-vector-stores-chroma chromadb
pip install llama-index-embeddings-huggingface
pip install sentence-transformers  # for HF embeddings
```

---

## Document Loading and Chunking

```python
from llama_index.core import SimpleDirectoryReader, VectorStoreIndex, Settings
from llama_index.core.node_parser import SentenceSplitter, SemanticSplitterNodeParser
from llama_index.embeddings.huggingface import HuggingFaceEmbedding

# Use a local embedding model (no API cost)
Settings.embed_model = HuggingFaceEmbedding(model_name="BAAI/bge-small-en-v1.5")
Settings.chunk_size    = 512
Settings.chunk_overlap = 50

# Load documents
documents = SimpleDirectoryReader("./docs", recursive=True).load_data()
print(f"Loaded {len(documents)} documents")

# Chunk strategy selection:
# - SentenceSplitter: stable, predictable chunk sizes → general use
# - SemanticSplitterNodeParser: groups semantically similar sentences → better for long docs
splitter = SentenceSplitter(chunk_size=512, chunk_overlap=50)
nodes = splitter.get_nodes_from_documents(documents)
print(f"Created {len(nodes)} chunks")
```

---

## Indexing to Chroma

```python
import chromadb
from llama_index.vector_stores.chroma import ChromaVectorStore
from llama_index.core import StorageContext

# Persistent Chroma client
chroma_client     = chromadb.PersistentClient(path="./chroma_db")
chroma_collection = chroma_client.get_or_create_collection("docs_index")

vector_store    = ChromaVectorStore(chroma_collection=chroma_collection)
storage_context = StorageContext.from_defaults(vector_store=vector_store)

# Build index (embeds all nodes and stores in Chroma)
index = VectorStoreIndex(nodes, storage_context=storage_context, show_progress=True)

# Persist index metadata to disk
index.storage_context.persist(persist_dir="./index_store")
print("Index built and persisted.")
```

---

## Retrieval and Query

```python
from llama_index.core import load_index_from_storage, StorageContext

# Load existing index
storage_context = StorageContext.from_defaults(persist_dir="./index_store",
                                               vector_store=vector_store)
index = load_index_from_storage(storage_context)

# Basic dense retrieval
query_engine = index.as_query_engine(similarity_top_k=5)
response = query_engine.query("What is the refund policy for enterprise customers?")
print(response)
```

---

## Hybrid Search (Dense + BM25)

```python
from llama_index.core.retrievers import BM25Retriever, QueryFusionRetriever
from llama_index.core.query_engine import RetrieverQueryEngine
from llama_index.core import PromptTemplate

# Dense retriever
dense_retriever = index.as_retriever(similarity_top_k=10)

# Sparse BM25 retriever (operates on same nodes)
bm25_retriever = BM25Retriever.from_defaults(nodes=nodes, similarity_top_k=10)

# Fusion: Reciprocal Rank Fusion merges dense + sparse results
fusion_retriever = QueryFusionRetriever(
    retrievers=[dense_retriever, bm25_retriever],
    similarity_top_k=5,
    num_queries=1,      # no query expansion (set to 3-5 to enable)
    mode="reciprocal_rerank",
    use_async=True,
)

query_engine = RetrieverQueryEngine(retriever=fusion_retriever)
response = query_engine.query("How do I cancel my subscription?")
print(response)
```

---

## Cross-Encoder Reranking

```python
from llama_index.core.postprocessor import SentenceTransformerRerank

# Retrieve top-20 candidates, rerank to top-5
reranker = SentenceTransformerRerank(
    model="cross-encoder/ms-marco-MiniLM-L-6-v2",
    top_n=5,
)

query_engine = index.as_query_engine(
    similarity_top_k=20,
    node_postprocessors=[reranker],
)
response = query_engine.query("What are the SLA guarantees in the enterprise tier?")
print(response)
```

---

## Chunk Strategy Selection

| Strategy | Chunk size | Best for |
|---|---|---|
| Fixed token | 256–512 tokens | General documents, predictable latency |
| Sentence-based | ~5–8 sentences | Prose documents, better semantic coherence |
| Semantic splitter | Variable | Long mixed-topic documents |
| Parent-child | Small (256) child, large (1024) parent | Precise retrieval + rich generation context |

**Embedding model selection:**

| Model | Dim | Speed | Quality | Cost |
|---|---|---|---|---|
| `text-embedding-3-small` (OpenAI) | 1536 | Fast (API) | High | $0.02/1M tokens |
| `BAAI/bge-small-en-v1.5` | 384 | Fast (local) | Medium-high | Free |
| `intfloat/e5-large-v2` | 1024 | Slower (local) | High | Free |

---

## Links

**AI Engineering**
- [[06_ai_engineering/04_rag_and_agents/rag_architecture|RAG Architecture]] — naive/advanced/modular RAG theory, HyDE, multi-query, reranking
- [[06_ai_engineering/04_rag_and_agents/vector_stores|Vector Stores]] — FAISS, Chroma, pgvector comparison

**System Patterns**
- [[chroma_vector_store|Chroma Vector Store]] — Chroma-specific CRUD and metadata filtering
- [[vector_database_retrieval|Vector Database Retrieval]] — FAISS vs. Chroma, index types, batch upsert
- [[rag_qa_system|RAG Q&A System]] — end-to-end RAG system with LangChain

**End-to-End Examples**
- [[rag_qa_system|RAG Q&A System]] — complete production RAG system
