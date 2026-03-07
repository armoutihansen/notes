---
layer: 07_applications
type: application
status: growing
tags: [vision, llm, workflow]
created: 2026-03-11
---

# Document Intelligence

## Problem

Automatically extract structured information from semi-structured or unstructured documents — invoices, insurance forms, contracts, identity documents, medical records — that arrive as scanned images or PDFs. Manual data entry is slow, expensive, and error-prone. Document intelligence replaces data entry with automated extraction, classification, and validation.

Common sub-tasks:
1. **Document classification**: What type is this document? (Invoice, FNOL form, driver's licence)
2. **Key-value extraction**: Extract named fields (invoice number, date, total amount)
3. **Table extraction**: Extract line items from invoice tables
4. **NER (Named Entity Recognition)**: Identify and tag entities (names, addresses, amounts)
5. **Form understanding**: Map handwritten or printed form fields to schema

## Users / Stakeholders

| Role | Use case |
|---|---|
| Finance / AP team | Automated invoice processing; 3-way match |
| Claims handler | Auto-extract from FNOL, police reports, medical records |
| KYC / compliance | ID document verification, AML due diligence |
| Loan officer | Income verification from bank statements, payslips |
| Insurance underwriter | Extract risk data from survey reports, questionnaires |

## Domain Context

- **Document variety**: Within a single organisation, hundreds of document templates from different vendors, countries, or years. Zero-shot or few-shot approaches (GPT-4V, document foundation models) outperform template-based systems that break with new layouts.
- **OCR quality**: Scanned documents have varying quality. Handwriting, low-resolution scans, stamps over text, and rotated pages create OCR challenges. Pre-processing pipeline is critical.
- **Extraction confidence**: Low-confidence extractions must be flagged for human review. The goal is not 100% automation but a STP (Straight-Through Processing) rate of 70–90% with a human review queue for exceptions.
- **Regulatory**: PII in financial and medical documents is subject to GDPR, HIPAA (US health), FCA consumer data rules. Data minimisation: extract only what is needed; don't retain raw document images beyond legal requirement.
- **Multi-page documents**: Contracts may be 50+ pages. Line item tables span multiple pages. Document segmentation and cross-page aggregation are necessary.

## Inputs and Outputs

**Input**:
```
document:        PDF, scanned image (JPEG/PNG/TIFF), or Word file
document_type:   Known type or unknown (auto-classify)
schema:          Target extraction schema (field names + data types + validation rules)
```

**Output**:
```
extracted_fields: {
  "invoice_number": {"value": "INV-2024-00123", "confidence": 0.99, "bbox": [...]}
  "invoice_date":   {"value": "2024-11-01", "confidence": 0.97, "bbox": [...]}
  "line_items":     [{"description": "...", "qty": 5, "unit_price": 99.50, ...}]
  "total_amount":   {"value": 497.50, "confidence": 0.95, "bbox": [...]}
}
stp_decision:     AUTO_PROCESS / HUMAN_REVIEW_REQUIRED
validation_errors: ["Total does not match sum of line items"]
```

## Decision or Workflow Role

```
Document received (email attachment / upload / scan)
  ↓
Ingestion: format normalisation → OCR (Tesseract / Azure Document Intelligence)
  ↓
Document classification: Invoice / Form / Contract / ID / Other
  ↓
Extraction model: field extraction by document type
  ↓
Validation: business rules (date logic, amount reconciliation, field completeness)
  ↓
Confidence assessment:
  all fields > threshold → AUTO_PROCESS → downstream system (ERP, CRM)
  any field < threshold → HUMAN_REVIEW queue with highlighted uncertain fields
  ↓
Human review outcome → label for retraining
  ↓
Processed document metadata stored (audit trail)
```

## Modeling / System Options

| Approach | Strength | Weakness | When to use |
|---|---|---|---|
| GPT-4V / Claude 3.5 (vision) | Zero-shot; handles novel layouts; high quality | Cost; latency; data privacy | Low volume; high variety; prototyping |
| Azure Document Intelligence (Form Recognizer) | Managed; pre-built models for invoices/IDs; easy | Vendor lock-in; limited customisation | Standard document types; quick deployment |
| LayoutLM / LayoutLMv3 | State-of-art form understanding; incorporates layout | Fine-tuning required; not zero-shot | High volume with consistent templates |
| Donut (end-to-end OCR-free) | No OCR preprocessing; direct image-to-text | Smaller model; may miss novel fields | Clean scanned documents |
| PaddleOCR + NER pipeline | Open-source; multilingual; fast | Lower accuracy; pipeline complexity | On-premises; cost-sensitive |
| Template matching (regex + anchor detection) | Fast; deterministic; auditable | Breaks with layout changes; high maintenance | Fixed-format documents (legacy) |

**Recommended**: Azure Document Intelligence for invoice/ID/receipt standard types. GPT-4V or Claude for high-value, high-variety documents (legal contracts). LayoutLMv3 fine-tuned for high-volume proprietary form types.

## Deployment Constraints

- **Accuracy threshold**: 95%+ field-level accuracy at >70% STP rate. Lower thresholds increase human review cost.
- **Privacy**: Never send PII-containing documents to external API without DPA. Consider on-premises solutions (LayoutLM + local OCR) for sensitive documents.
- **Latency**: Async acceptable — most extraction workflows tolerate 5–30 seconds processing time.
- **Audit trail**: All automated extractions must be stored with source document reference and extraction confidence for regulatory audit.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **OCR failure on poor scan** | Low-quality scan → garbled text → extraction failure | Image pre-processing; quality score before extraction |
| **Novel template** | New vendor invoice layout → field extraction fails | Confidence-based human review queue |
| **Amount transcription error** | Incorrect amount extracted → financial loss | Amount cross-validation; approver review for large amounts |
| **PII leakage** | Document sent to external API; data breach | On-premises models; DPA; data classification |
| **Format drift** | Template changes year-over-year → gradual accuracy decline | Field-level accuracy monitoring; drift alerts |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| Field accuracy | > 95% per field type | Measured against human validation |
| STP rate | > 75% | Straight-through without human review |
| Processing time | < 30s per document | Async pipeline SLA |
| Human review queue clearance | > 95% within 4h | Operational throughput |
| Cost per document | < £0.10 | vs manual processing ~£2–5 per document |
| Error rate in downstream system | < 0.5% | PO matching failures, payment errors |

## References

- Xu, Y. et al. (2020). *LayoutLM: Pre-Training of Text and Layout for Document Image Understanding.* KDD.
- Kim, G. et al. (2022). *OCR-Free Document Understanding Transformer.* (Donut)

## Links

**AI Engineering**
- [[06_ai_engineering/00_foundation_models/index|Foundation Models]] — GPT-4V, Claude 3.5 Vision capabilities
- [[06_ai_engineering/03_prompt_engineering/index|Prompt Engineering]] — structured extraction prompts

**Modeling**
- [[03_modeling/04_deep_learning/index|Deep Learning]] — LayoutLM, CNN-based document models

**Reference Implementations**
- [[08_implementations/01_system_patterns/instructor_structured_outputs|Instructor Structured Outputs]]
- [[06_ai_engineering/05_finetuning/transformer_finetuning_implementation|Transformer Fine-tuning Implementation]]

**Adjacent Applications**
- [[visual_inspection|Visual Inspection]]
- [[07_applications/08_domain_verticals/01_insurance/claims_automation|Claims Automation]]
- [[07_applications/05_generation_and_assistance/document_summarization|Document Summarization]]
