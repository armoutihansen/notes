---
layer: 07_applications
type: application
status: growing
tags: [vision, classification, anomaly-detection]
created: 2026-03-11
---

# Quality Control Vision

## Problem

Deploy computer vision systems for real-time quality inspection on manufacturing production lines — detecting surface defects, dimensional deviations, assembly errors, and foreign material contamination. This is the same technical domain as [[07_applications/07_multimodal_systems/visual_inspection|Visual Inspection]] but framed from the operations and supply chain perspective: how does the QC system integrate into the manufacturing process, quality management system, and continuous improvement cycle?

The key operational distinction: QC vision is not just about detecting defects, it is about **preventing defects** by closing the feedback loop into process control.

## Users / Stakeholders

| Role | Decision |
|---|---|
| Quality engineer | Define acceptance criteria; analyse defect root cause |
| Production supervisor | Stop/start line; escalate quality issues |
| Process engineer | Adjust process parameters based on defect data |
| Customer quality manager | Verify outgoing quality; customer complaint reduction |
| Operations director | Quality cost; warranty liability; brand risk |

## Domain Context

- **Six Sigma / SPC integration**: Statistical Process Control (SPC) charts (X-bar, R-chart, CUSUM) are the established quality management methodology. ML vision integrates as the measurement system feeding SPC.
- **First-pass yield vs rework**: QC identifies defective product. Operations decision: scrap or rework? ML can classify defect severity to automate scrap vs rework routing.
- **Incoming goods inspection**: QC vision can inspect received materials from suppliers — reducing incoming QC labour and catching supplier quality issues early.
- **Measurement system analysis (MSA)**: Before deploying a vision system, it must pass a Gauge R&R (Repeatability and Reproducibility) study — the system must be at least as consistent as a human inspector.
- **ISO 9001 / IATF 16949**: Quality management certification requires documented inspection procedures, calibration records, and non-conformance management.
- **Data ownership**: Defect images and quality data are proprietary. Often kept on-premises for IP protection.

## Inputs and Outputs

**Inputs**:
```
Camera images: 1–20 Megapixel, multiple cameras per station (top/bottom/sides)
Trigger: encoder pulse from conveyor or part presence sensor
Reference: golden sample image or 3D CAD model
Metadata: part_number, batch_id, station_id, production_date_time, shift_id
Process parameters: temperature, pressure, tool_wear_count (for root cause correlation)
```

**Output**:
```
pass_fail:         PASS / FAIL / MARGINAL
defect_class:      SCRATCH / CRACK / CONTAMINATION / DIMENSIONAL / ASSEMBLY / COLOUR
defect_location:   Bounding box + pixel mask on original image
severity_score:    Minor / Major / Critical (maps to scrap vs rework decision)
spc_data_point:    Defect rate contribution to SPC chart
audit_record:      Stored image + result for traceability (5–10 year retention)
```

## Decision or Workflow Role

```
Part arrives at inspection station
  ↓
Camera(s) capture image(s) → triggered inspection
  ↓
Real-time inference (edge device, <50ms)
  ↓
PASS → part continues down line
FAIL/Critical → automated reject + defect image stored
FAIL/Marginal → human review station
  ↓
Defect data → real-time SPC chart update
  ↓
SPC out-of-control signal → line stop alert to supervisor
  ↓
Root cause analysis: correlate defect rate with process parameters
  ↓
Process adjustment → improvement verified → model calibration updated
  ↓
Weekly: review defect types → update labelling → retrain if needed
```

## Modeling / System Options

See [[07_applications/07_multimodal_systems/visual_inspection|Visual Inspection]] for detailed model comparison.

**Operational additions for QC**:

| System | Approach | Notes |
|---|---|---|
| Anomaly detection (PatchCore) | No defect labels needed initially | For new product introduction |
| Supervised CNN (EfficientNet) | Higher accuracy when defects are labelled | As defect dataset grows |
| 3D point cloud (lidar/structured light) | Dimensional QC — not just surface | Height, flatness, gap/flush measurements |
| Template matching | Deterministic for assembly presence/absence check | Is component A present and in position? |

## Deployment Constraints

- **Gauge R&R validation**: Must demonstrate repeatability (same part measured multiple times) ≥ 90% agreement and reproducibility (different cameras/shifts) ≥ 90% agreement before certification.
- **Traceability**: Every part inspection must be stored with part ID for product recall capability. IATF 16949 requires full traceability.
- **Integration with MES**: Machine Execution System (Siemens Opcenter, SAP MII) integration for production data, batch records, and non-conformance management.
- **Lighting maintenance**: Inspection lighting degrades over time. Automated lighting calibration check (measure reference card daily) is necessary.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Measurement drift** | Camera/lighting changes → false positive/negative rate changes | Reference card daily calibration; APC (automatic process control) |
| **Product changeover** | New product variant introduced → model not trained → high FPR | Change management process; model update procedure |
| **Rare defect type** | Never-seen defect bypasses detector | Anomaly detection as safety layer |
| **Environmental contamination** | Dust, vibration, temperature affect camera performance | IP65-rated enclosures; vibration isolation |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| Defect escape rate | < 50 PPM | Parts Per Million shipped with defects |
| False positive rate | < 0.5% of production | Reject yield loss cost |
| First-pass yield | Improvement vs manual inspection | % of parts passing first inspection |
| Gauge R&R | < 10% total variation | Measurement system acceptability threshold |
| Inspector FTE reduction | 60–80% | Operational cost saving |
| System uptime | > 99.5% | Reliability requirement |

## References

- Montgomery, D. (2019). *Introduction to Statistical Quality Control.* Wiley.
- Bergmann, P. et al. (2019). *MVTec AD — A Comprehensive Real-World Dataset for Unsupervised Anomaly Detection.* CVPR.

## Links

**Modeling**
- [[03_modeling/04_deep_learning/index|Deep Learning]] — CNN for visual defect detection
- [[03_modeling/02_unsupervised_learning/index|Unsupervised Learning]] — anomaly detection

**Application Cross-links**
- [[07_applications/07_multimodal_systems/visual_inspection|Visual Inspection]] — detailed model technical note
- [[07_applications/03_detection_and_monitoring/anomaly_detection_operations|Anomaly Detection — Operations]]

**Reference Implementations**
- [[08_reference_implementations/01_model_implementations/neural_network_implementation|Neural Network Implementation]]
- [[08_reference_implementations/01_model_implementations/unsupervised_learning_implementation|Unsupervised Learning Implementation]]

**Adjacent Applications**
- [[capacity_planning|Capacity Planning]]
- [[07_applications/08_domain_verticals/05_mobility/predictive_maintenance|Predictive Maintenance]]
