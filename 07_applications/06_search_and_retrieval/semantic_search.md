---
layer: 07_applications
type: application
status: growing
tags: [retrieval, llm, workflow]
created: 2026-03-11
---

# Semantic Search

## Problem

Enable users to find relevant documents, passages, or data records using natural language queries — without requiring exact keyword matches. Traditional keyword search fails when the user's query vocabulary differs from document vocabulary (synonyms, acronyms, paraphrases, cross-language queries). Semantic search uses dense vector embeddings to represent query and document meaning in a shared semantic space, enabling relevance matching based on meaning rather than string overlap.

## Users / Stakeholders

| Role | Use case |
|---|---|
| Knowledge worker | Find relevant policy, procedure, or research document |
| Customer support agent | Surface relevant knowledge base articles during a support call |
| Researcher / analyst | Explore large document corpus for relevant evidence |
| Product user | Search product catalogue with natural language |
| Compliance officer | Find all documents relevant to a specific regulatory topic |

## Domain Context

- **Vocabulary mismatch**: "How do I cancel my subscription?" ≠ "account termination procedure" in keyword search. Dense retrieval bridges this gap.
- **Hybrid search**: BM25 keyword search and dense retrieval are complementary. BM25 excels at exact match (product codes, names, numbers); dense retrieval excels at semantic similarity. Hybrid with Reciprocal Rank Fusion (RRF) outperforms either alone.
- **Embedding drift**: Document embeddings generated with model version V1 are not comparable with query embeddings from model version V2. Re-indexing required on model updates.
- **Multilingual**: Enterprise documents may be in multiple languages. Multilingual embeddings (mE5, multilingual-e5) enable cross-lingual search.
- **Access control**: Search must respect document-level permissions. User A should not retrieve documents they are not authorised to read. ACL filtering at retrieval time is mandatory.
- **Scale**: Enterprise corpora range from 10K to 100M documents. ANN (Approximate Nearest Neighbour) search is required above ~50K documents.

## Inputs and Outputs

**Input**:
```
query:              Natural language search query (free text)
user_context:       Department, role, language, location (for personalisation/filtering)
access_control:     User's permitted document ACL groups
filters:            Optional structured filters (date range, document type, source)
n_results:          Number of results requested
```

**Output**:
```
results:  [
  {
    doc_id:    "policy-123",
    title:     "Account Termination Policy v3.2",
    snippet:   "...accounts may be terminated by contacting support@...",
    score:     0.87,
    metadata:  {source, date, author, access_group}
  },
  ...
]
```

## Decision or Workflow Role

```
[Indexing pipeline] (offline, nightly or event-driven)
Document corpus → Extract text → Chunk (512 tokens, 50 overlap)
→ Embed (text-embedding-3-small / bge-large-en-v1.5)
→ Upsert to vector store (Chroma / pgvector / Qdrant) with metadata

[Query pipeline] (online, <200ms)
User query → Embed query (same model)
→ ANN search: dense retrieval (top 50)
→ BM25 search: keyword (top 50)
→ RRF fusion: merge and re-rank results
→ Cross-encoder reranker (top 50 → top 10)
→ ACL filter: remove unauthorized results
→ Return top 10 with snippets
```

## Modeling / System Options

| Component | Options | Trade-off |
|---|---|---|
| Dense embedding | text-embedding-3-small (OpenAI), bge-large-en-v1.5 (BAAI), E5-large | Cost vs quality vs on-premises |
| Sparse retrieval | BM25 (Elasticsearch, OpenSearch, BM25s library) | Exact match quality |
| Fusion | RRF (no tuning), weighted linear (requires tuning) | Simplicity vs performance |
| Reranker | cross-encoder/ms-marco-MiniLM (fast), Cohere Rerank API | Latency vs quality |
| Vector store | pgvector (SQL integration), Qdrant (performance), Chroma (dev) | Operational complexity vs features |

## Deployment Constraints

- **Latency**: Query pipeline P99 < 200ms. Reranker adds 30–80ms.
- **Index freshness**: New documents should appear in search within 1 hour. Streaming index updates via event-driven pipeline.
- **ACL enforcement**: Filter must happen at vector store level (metadata filtering) or post-retrieval — never rely solely on post-generation filtering.
- **Embedding model pinning**: Pin model version. Log query + results for evaluation. Never change embedding model without re-indexing.
- **Scale**: pgvector handles <10M vectors well. Qdrant or Weaviate for larger corpora.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Relevance drift** | Query distribution shifts; older embeddings less relevant | Periodic offline evaluation with human judgements |
| **ACL bypass** | Privileged document returned to unauthorised user | Server-side ACL filter; security audit |
| **Embedding model version mismatch** | Query/document embeddings from different models | Version pinning; re-index on upgrade |
| **Snippet extraction** | Snippet shows irrelevant context around the match | Sentence-boundary aware snippet extraction |
| **Hallucination from snippets** | User misreads snippet as authoritative answer | Clear citation display; link to source |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| MRR@10 | > 0.6 | Mean Reciprocal Rank; click-based or human relevance |
| NDCG@10 | > 0.5 | Normalised Discounted Cumulative Gain |
| Query success rate | > 80% (user finds relevant result) | Survey or implicit feedback |
| Latency P99 | < 200ms | Operational SLA |
| Coverage | 100% of authorised documents indexed | Index completeness |

## References

- Karpukhin, V. et al. (2020). *Dense Passage Retrieval for Open-Domain Question Answering.* (DPR paper)
- Ma, X. et al. (2022). *Fine-Tuned Language Models are Zero-Shot Learners.* (E5 model)

## Links

**AI Engineering**
- [[06_ai_engineering/04_rag_and_agents/vector_stores|Vector Stores]] — ANN index comparison
- [[06_ai_engineering/04_rag_and_agents/rag_architecture|RAG Architecture]] — retrieval pipeline design

**Reference Implementations**
- [[08_implementations/01_system_patterns/vector_database_retrieval|Vector Database Retrieval]]
- [[08_implementations/01_system_patterns/chroma_vector_store|Chroma Vector Store]]

**Adjacent Applications**
- [[rag_knowledge_assistant|RAG Knowledge Assistant]]
- [[07_applications/02_recommendation_and_ranking/product_recommendation|Product Recommendation]]
