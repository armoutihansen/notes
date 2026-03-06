---
layer: 02_modeling
type: concept
status: seed
tags: [computer_vision, detection, yolo, anchor_boxes]
created: 2026-03-02
---

# Object Detection

## Definition

The task of predicting axis-aligned bounding boxes and class labels for all instances of target object categories in an image; requires solving localization and classification jointly.

## Intuition

Unlike classification (one label per image), detection must find where each object is. YOLO-style models predict boxes directly from grid cells; anchor boxes provide shape priors; IoU measures localization quality; NMS removes duplicate detections.

## Formal Description

**Bounding box parameterization:** $(b_x, b_y, b_w, b_h)$ = center + width/height, normalized to image or grid cell dimensions.

**Intersection over Union (IoU):**

$$\text{IoU}(B_1, B_2) = \frac{\text{area}(B_1 \cap B_2)}{\text{area}(B_1 \cup B_2)} \in [0,1]$$

Threshold typically 0.5 for "correct" detection; used in loss and NMS.

**Anchor boxes:** predefined box shapes (aspect ratios) at each grid cell position; each grid cell predicts multiple boxes (one per anchor); box prediction is a delta from the anchor shape; improves detection of elongated objects (cars, pedestrians).

**YOLO (You Only Look Once):** divide image into $S \times S$ grid; each cell predicts $B$ boxes (one per anchor), each box has $(b_x, b_y, b_w, b_h, p_\text{obj})$ + $C$ class probabilities; output tensor: $S \times S \times B \times (5 + C)$; single forward pass — fast enough for real-time.

**Non-Maximum Suppression (NMS):**
1. Discard boxes with $p_\text{obj} < \text{threshold}$
2. For each class, sort remaining by confidence
3. Greedily keep the highest-confidence box
4. Remove all remaining boxes with IoU ≥ threshold against the kept box
5. Repeat

**Loss function (YOLO-style):** sum of localization loss (MSE on box coordinates), confidence loss (BCE on objectness), classification loss (BCE or CE).

## Applications

Autonomous driving (pedestrian/vehicle detection), surveillance, medical imaging (lesion detection), retail (inventory counting).

## Trade-offs

- YOLO trades accuracy for speed vs. two-stage detectors (Faster R-CNN)
- NMS is sequential and hard to parallelize
- Anchor box design is domain-specific (sizes must match typical object scales)
- Recent anchor-free methods (FCOS, DETR) avoid anchor design

## Links

- [[convolution]]
- [[cnn_architecture]]
- [[face_recognition]]
- [[02_modeling/03_model_families/04_neural_networks/index|Neural Networks Index]]
