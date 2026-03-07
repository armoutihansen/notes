---
layer: 07_applications
type: application
status: growing
tags: [llm, retrieval, workflow]
created: 2026-03-11
---

# RAG Knowledge Assistant

## Problem

Provide employees with a conversational assistant that answers questions using the organisation's internal knowledge base — policies, procedures, technical documentation, past project outputs, HR information — in natural language, with citations. Traditional intranets and wikis are poorly navigated; employees waste significant time searching for answers or asking colleagues. A RAG assistant reduces resolution time and improves consistency of answers.

## Users / Stakeholders

| Role | Use case |
|---|---|
| New employee | Onboarding questions about policies and procedures |
| HR employee | Answer employee benefits and leave policy questions at scale |
| Engineer | Technical documentation lookup, API reference |
| Claims adjuster | Coverage interpretation, underwriting guidelines |
| Sales | Product pricing, eligibility rules, competitive positioning |

## Domain Context

- **Knowledge currency**: Documents in the knowledge base may be outdated. The system must surface document dates and must not present stale answers as current fact. Freshness filtering is critical.
- **Confidentiality tiers**: Some documents (board minutes, personnel files, M&A documents) should not be accessible to all employees. ACL enforcement is non-negotiable.
- **Ambiguous queries**: "What is our holiday policy?" is ambiguous — which country? Which employment type? The system must clarify or provide multiple answers.
- **Hallucination risk**: The LLM must answer only from retrieved context. Without proper grounding, it will fabricate policy details — potentially creating legal exposure.
- **Integration**: Knowledge is spread across SharePoint, Confluence, Google Drive, email archives, PDF repositories. Connectors for each system are required.

## Inputs and Outputs

**Input**:
```
user_query:          Natural language question
conversation_history: Prior turns in the chat session
user_context:        Department, location, role, language
```

**Output**:
```
answer:      Natural language response grounded in retrieved documents
citations:   List of source documents with title, URL, date, relevant excerpt
confidence:  CONFIDENT / UNCERTAIN (based on retrieval quality score)
followup:    Suggested clarifying questions (optional)
```

## Decision or Workflow Role

```
[Knowledge ingestion pipeline] — nightly or event-driven
Connectors (SharePoint / Confluence / Google Drive / PDF upload)
  → Text extraction + OCR → Chunking → Embedding → Upsert to vector store
  → Metadata: source, date, author, ACL group, document_type

[Query pipeline] — real-time
User query → ACL context attached
  → Semantic search: embed query → ANN retrieval (top 20)
  → Metadata filter: ACL check, freshness filter
  → Reranking: cross-encoder top 20 → top 5
  → Prompt assembly: [system prompt] + [top 5 chunks] + [chat history] + [query]
  → LLM generation (GPT-4o-mini / Claude Haiku)
  → Citation extraction + response formatting
  → Deliver to user via Slack / Teams / web UI
  → Log interaction → evaluation pipeline → fine-tuning data
```

## Modeling / System Options

| Decision | Options | Notes |
|---|---|---|
| Embedding model | text-embedding-3-small (cost), bge-large-en-v1.5 (accuracy) | On-prem vs API cost |
| LLM | GPT-4o-mini (cost/quality), Claude Haiku (fast), Llama-3-8B (on-prem) | Privacy requirements drive choice |
| Vector store | pgvector (SQL-native), Qdrant (performance), Chroma (dev) | Scale + ops maturity |
| RAG framework | LangChain (broad connectors), LlamaIndex (retrieval focus) | Connector ecosystem |
| Grounding enforcement | Prompt engineering + "do not hallucinate" instruction, NLI faithfulness check, citation-forced generation | Layered defence |

**Recommended**: LlamaIndex for retrieval pipeline (better chunking strategies). GPT-4o-mini or Claude Haiku for generation. pgvector if team has PostgreSQL expertise.

## Deployment Constraints

- **Privacy and data residency**: Employee knowledge bases contain personal data. Data must stay in the relevant jurisdiction. On-premises LLM or EU-hosted API endpoint required for GDPR compliance.
- **Latency**: Conversational response expected in <5 seconds. Streaming output (token-by-token) improves perceived latency.
- **Access control**: System must call the organisation's identity provider (Okta, Azure AD) to resolve user permissions before retrieval.
- **Confidence signalling**: When retrieved chunks have low relevance scores, the system should say "I couldn't find a confident answer" rather than hallucinate.
- **Observability**: All queries, retrieved chunks, and generated answers must be logged (LangSmith / custom) for quality auditing and compliance.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Policy hallucination** | LLM invents policy details not in the documents | Grounding constraint in system prompt; faithfulness check |
| **ACL bypass** | Confidential document retrieved for unauthorised user | Server-side ACL filter at retrieval stage |
| **Stale knowledge** | Answer based on outdated document version | Freshness filter; document date in answer context |
| **Overconfident wrong answer** | System gives definitive answer when documents are ambiguous | Uncertainty signalling; "consult HR for confirmation" framing |
| **Index coverage gaps** | Question answered with partial context because relevant doc not indexed | Document coverage reporting; connector completeness monitoring |
| **Prompt injection** | Malicious user crafts query to exfiltrate other users' data | Input sanitisation; strict system prompt; output filtering |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| Answer accuracy (human eval) | > 85% correct | Spot check by domain expert |
| Citation accuracy | > 90% of answers have valid citation | Verifiable grounding |
| User satisfaction | > 4.0/5 | Monthly survey |
| Deflection rate | > 40% of HR / helpdesk queries resolved without human | Business efficiency metric |
| Hallucination rate | < 2% of answers | Human adversarial evaluation |
| Time to answer | < 5 seconds | UX SLA |

## References

- Lewis, P. et al. (2020). *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks.* NeurIPS.
- Gao, Y. et al. (2023). *Retrieval-Augmented Generation for Large Language Models: A Survey.*

## Links

**AI Engineering**
- [[06_ai_engineering/04_rag_and_agents/rag_architecture|RAG Architecture]] — design patterns for retrieval systems
- [[06_ai_engineering/04_rag_and_agents/vector_stores|Vector Stores]] — ANN index comparison
- [[06_ai_engineering/08_architecture_and_feedback/index|Architecture and Feedback]] — LLMOps for production

**Reference Implementations**
- [[08_implementations/02_end_to_end_examples/rag_qa_system|RAG Q&A System]]
- [[08_implementations/01_system_patterns/rag_pipeline_pattern|RAG Pipeline Pattern]]
- [[08_implementations/01_system_patterns/langsmith_llm_observability|LangSmith LLM Observability]]

**Adjacent Applications**
- [[semantic_search|Semantic Search]]
- [[07_applications/05_generation_and_assistance/document_summarization|Document Summarization]]
