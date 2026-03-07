---
layer: 07_applications
type: application
status: growing
tags: [llm, generation, workflow]
created: 2026-03-11
---

# Document Summarization

## Problem

Automatically generate concise, accurate summaries of long-form enterprise documents — policies, contracts, research reports, meeting transcripts, incident reports, regulatory filings — so that employees can extract key information without reading the full document. The challenge is faithfulness (no hallucinated facts), relevance (summary matches reader intent), and scalability (thousands of documents processed daily).

## Users / Stakeholders

| Role | Use case |
|---|---|
| Legal / compliance officer | Summarise contract clauses, regulatory guidance |
| Claims adjuster | Extract key facts from medical records, police reports |
| Analyst | Summarise research reports, earnings calls |
| Customer support agent | Summarise customer ticket history |
| Executive | Board-ready executive summaries of technical reports |

## Domain Context

- **Document types vary widely**: PDFs, Word documents, scanned images (requiring OCR), HTML pages, structured forms. Each has different extraction complexity.
- **Context window limits**: Long documents (100+ pages) exceed LLM context windows. Chunking + map-reduce summarisation or retrieval-based selection is necessary.
- **Faithfulness is critical**: In legal and medical contexts, a hallucinated fact in a summary can have serious consequences. Faithfulness evaluation (NLI-based) is mandatory.
- **Regulatory**: Processing personal data in documents triggers GDPR. Legal professional privilege may apply to legal document summaries — data handling must comply with attorney-client privilege rules.
- **Language diversity**: Enterprise documents may be multilingual. Model must handle target languages (GPT-4 handles 95+ languages; smaller models may not).

## Inputs and Outputs

**Input**:
```
document:          Raw text (extracted from PDF/DOCX/HTML) or markdown
document_type:     CONTRACT / POLICY / REPORT / TRANSCRIPT / MEDICAL_RECORD
user_intent:       FREE_TEXT or STRUCTURED_QUERY ("what are the termination clauses?")
length_target:     SHORT (3 sentences) / MEDIUM (1 paragraph) / DETAILED (1 page)
output_format:     PROSE / BULLET_POINTS / STRUCTURED_JSON
```

**Output**:
```
summary:           Generated text summary matching length and format target
key_points:        Extracted bullet points of most important facts
entities:          Named entity list (people, organisations, dates, amounts)
citations:         Source text spans that support key summary claims
confidence:        Faithfulness score (NLI-based) if available
```

## Decision or Workflow Role

```
Document arrives (upload / email / API push)
  ↓
Ingestion: format detection → text extraction → OCR if needed
  ↓
Chunking: recursive character split (512–2048 tokens with overlap)
  ↓
For short docs (<16K tokens): direct summarisation prompt
For long docs: 
  → Map: summarise each chunk independently
  → Reduce: summarise the chunk summaries into final summary
  ↓
Post-processing: faithfulness check, entity extraction, formatting
  ↓
Delivery: push to CRM / SharePoint / email / API response
  ↓
User feedback (thumbs up/down) → log → fine-tuning dataset
```

## Modeling / System Options

| Approach | Strength | Weakness | When to use |
|---|---|---|---|
| Prompt + LLM (GPT-4o, Claude) | Flexible; no training; high quality | Cost; latency; no fine-tuning | General enterprise, low volume |
| Fine-tuned smaller model (Llama, Mistral) | Lower cost; on-premises; domain adaptation | Training cost; quality ceiling | High volume; data privacy; domain-specific |
| Extractive summarisation (BERT) | Faithful (no hallucination risk); fast | Not abstractive; robotic output | Faithfulness-critical; compliance context |
| Map-reduce chain (LangChain) | Handles arbitrary document length | Coherence loss in reduce step | Long documents (>50K tokens) |
| RAG-based (retrieval + generation) | Query-specific summaries; better for targeted questions | Infrastructure cost | When user specifies what to summarise |

**Recommended**: GPT-4o-mini or Claude Haiku for cost-effective production. Fine-tune Mistral-7B for high-volume, domain-specific (e.g., medical records). Always include faithfulness evaluation.

## Deployment Constraints

- **Latency**: Async acceptable for most use cases — process in background, deliver via notification. Interactive use (user waiting) requires <10s for short documents.
- **Cost**: GPT-4o at $5/1M output tokens. A 10-page document = ~2K output tokens = ~$0.01. At 100K docs/day = $1K/day. Cost model must be validated before scaling.
- **Privacy**: Documents often contain PII. On-premises models (vLLM + Llama) or data processing agreements with API providers required. Avoid sending raw documents to external APIs without legal review.
- **Faithfulness testing**: Deploy NLI faithfulness scorer (TRUE, SummaC) on a sample of outputs. Alert if faithfulness drops below threshold.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Hallucination** | LLM generates facts not in the source document | Faithfulness evaluation; extractive baseline; citation requirement |
| **Information loss** | Key sections not captured in summary | Length-adaptive prompting; user feedback loop |
| **Context window overflow** | Long documents truncated without warning | Chunking strategy; document length metadata |
| **PII leakage** | Summaries include sensitive data sent externally | On-prem model; data classification before routing |
| **Language quality degradation** | Non-English documents summarised with errors | Evaluate per language; language-specific model routing |
| **Format drift** | Model changes output format across versions | Schema validation; versioned prompts pinned to model version |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| Faithfulness score (NLI) | > 0.85 | No hallucinated facts; measured on a human-labelled test set |
| ROUGE-L | > 0.35 | Overlap with human reference summary (when available) |
| User satisfaction | > 4.2/5 | Survey-based; primary quality signal |
| Time to insight (user reported) | 60–80% reduction | Business impact metric |
| Processing throughput | 100+ docs/minute | Operational requirement |
| Cost per document | < $0.05 | Financial viability |

## References

- Goyal, T. et al. (2022). *News Summarization and Evaluation in the Era of GPT-3.* EMNLP.
- Laban, P. et al. (2022). *SummaC: Re-Visiting NLI-based Models for Inconsistency Detection in Summarization.* TACL.

## Links

**AI Engineering**
- [[06_ai_engineering/03_prompt_engineering/index|Prompt Engineering]] — chain-of-thought prompting, map-reduce chains
- [[06_ai_engineering/04_rag_and_agents/rag_architecture|RAG Architecture]] — query-specific document retrieval
- [[06_ai_engineering/00_foundation_models/index|Foundation Models]] — LLM capabilities and context window limits

**ML Engineering**
- [[05_ml_engineering/06_deployment_and_serving/index|Deployment and Serving]] — async document processing pipeline

**Reference Implementations**
- [[08_implementations/01_system_patterns/rag_pipeline_pattern|RAG Pipeline Pattern]]
- [[08_implementations/02_end_to_end_examples/rag_qa_system|RAG Q&A System]]

**Adjacent Applications**
- [[code_generation_assistant|Code Generation Assistant]]
- [[07_applications/06_search_and_retrieval/rag_knowledge_assistant|RAG Knowledge Assistant]]
