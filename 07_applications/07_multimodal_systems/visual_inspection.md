---
layer: 07_applications
type: application
status: growing
tags: [vision, anomaly-detection, classification]
created: 2026-03-11
---

# Visual Inspection

## Problem

Automatically detect defects, anomalies, or quality issues in physical products or environments using camera images or video — replacing or augmenting manual visual inspection on a production line, in a warehouse, or in field operations. Visual inspection ML must achieve human-level accuracy at machine speed, and the cost of a false negative (defective product shipped to customer) typically far exceeds the cost of a false positive (acceptable product rejected).

## Users / Stakeholders

| Role | Decision |
|---|---|
| Quality control inspector | Review flagged items; approve borderline cases |
| Production line manager | Stop/continue production; escalate to maintenance |
| Quality engineer | Defect root cause analysis; process improvement |
| Customer quality team | Defect rate reporting; warranty claim analysis |

## Domain Context

- **Real-time constraint**: Products on a moving line pass the camera at fixed intervals. Inference must complete before the product exits the inspection zone (typically 50–200ms budget).
- **Limited labelled defects**: Defects are rare by definition. Collecting labelled defect images requires running the line until defects occur. Semi-supervised and anomaly detection approaches are valuable.
- **Defect taxonomy**: Cracks, scratches, chips, contamination, incorrect assembly, missing components. Each defect type may require a different model or threshold.
- **Lighting and sensor variation**: Image quality depends on lighting, camera calibration, lens contamination. Distribution shift from sensor degradation is a significant production risk.
- **Edge deployment**: Cameras and inference must often run at the edge (no cloud round-trip). Edge hardware: NVIDIA Jetson, Intel OpenVINO, Google Coral. Model compression (quantization, pruning) required.
- **ISO 9001 / IATF 16949**: Quality management standards require documented inspection procedures. ML-based inspection requires validation records showing equivalence to manual inspection.

## Inputs and Outputs

**Input**:
```
images:        High-resolution camera frames (1MP–12MP), greyscale or RGB
trigger:       Line encoder pulse triggers capture
metadata:      product_id, batch_id, station_id, timestamp, shift
reference:     Golden sample image for comparison (optional)
```

**Output**:
```
decision:         PASS / FAIL / MARGINAL (human review)
defect_class:     CRACK / SCRATCH / CONTAMINATION / MISSING_COMPONENT / ...
defect_location:  Bounding box or segmentation mask on image
confidence:       P(defect) ∈ [0, 1]
audit_record:     Image + annotation stored for traceability
```

## Decision or Workflow Role

```
Camera captures product image (triggered by conveyor encoder)
  ↓
Image preprocessing: normalisation, crop to ROI, white balance correction
  ↓
Inference on edge device (<50ms budget)
  ↓
PASS:     Product continues down line
FAIL:     Pneumatic reject gate activates
MARGINAL: Image queued for human review within 30s
  ↓
Review outcome logged → retraining dataset
  ↓
Weekly: retrain cycle with new confirmed defect examples
```

## Modeling / System Options

| Approach | Strength | Weakness | When to use |
|---|---|---|---|
| CNN classification (ResNet, EfficientNet) | High accuracy with sufficient labelled data | Requires many defect examples | Mature product line with rich defect history |
| Anomaly detection (PatchCore, PADIM) | No defect labels required; detects novel defects | Higher FPR; harder to classify defect type | Few labelled defects; novel defect types |
| Object detection (YOLO) | Locates and classifies multiple defects in one pass | Requires bounding box annotations | Multiple defect types in one image |
| Semantic segmentation (U-Net) | Pixel-level defect mapping | Annotation cost; slower inference | Precise area measurement required |
| Traditional CV (template matching, edge detection) | No training needed; fast; interpretable | Fragile to variation; limited to simple defects | Simple go/no-go checks; quick deployment |

**Recommended**: PatchCore or PaDiM for initial deployment (no defect labels needed). Transition to fine-tuned EfficientNet-B4 as defect dataset grows. YOLO for multi-defect detection on complex assemblies.

## Deployment Constraints

- **Inference latency**: < 50ms at the edge. Use TensorRT or OpenVINO for optimisation.
- **Edge hardware**: NVIDIA Jetson AGX Orin (50–100ms), Intel NUC + OpenVINO (30–80ms), Google Coral (~5ms for small models).
- **Model size**: Quantise to INT8 for edge. Accuracy drop typically <1% vs FP32.
- **Connectivity**: Edge inference with cloud sync for retraining data. Must function offline.
- **Traceability**: Every inspection decision must be stored with product ID for quality audits. Retention: typically 3–10 years.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Lighting change** | Sensor/bulb degradation changes image distribution → model fails | Automated lighting calibration; distribution shift monitoring |
| **Novel defect type** | New defect not in training data → classified as PASS | Anomaly detection as safety layer; low-confidence → MARGINAL |
| **Edge device failure** | Inference device crashes → line runs without inspection | Hardware watchdog; fallback to manual inspection alert |
| **Overfit to training camera** | Model doesn't generalise to new cameras after equipment upgrade | Domain adaptation; camera-specific fine-tuning |
| **Class imbalance** | Very few defect examples → model biased toward PASS | Oversampling; synthetic defect augmentation |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| Defect detection recall | > 99% | Miss rate drives customer escapes |
| False positive rate | < 0.5% | Reject yield loss: economic cost |
| Inference latency P99 | < 50ms | Line speed constraint |
| Customer escape rate | < 10 PPM | Parts Per Million defective at customer |
| Inspector workload reduction | > 80% | Operational efficiency |
| Model uptime | > 99.5% | Reliability SLA |

## References

- Bergmann, P. et al. (2020). *Uninformed Students: Student-Teacher Anomaly Detection.* CVPR.
- Roth, K. et al. (2022). *Towards Total Recall in Industrial Anomaly Detection.* (PatchCore)

## Links

**Modeling**
- [[03_modeling/04_deep_learning/index|Deep Learning]] — CNN architectures, transfer learning
- [[03_modeling/02_unsupervised_learning/index|Unsupervised Learning]] — anomaly detection

**Reference Implementations**
- [[03_modeling/04_deep_learning/01_mlp_and_representation_learning/neural_network_implementation|Neural Network Implementation]]
- [[03_modeling/02_unsupervised_learning/01_clustering/unsupervised_learning_implementation|Unsupervised Learning Implementation]]

**Adjacent Applications**
- [[document_intelligence|Document Intelligence]]
- [[07_applications/03_detection_and_monitoring/anomaly_detection_operations|Anomaly Detection — Operations]]
- [[07_applications/08_domain_verticals/06_operations/quality_control_vision|Quality Control Vision]]
