---
layer: 04_ml_engineering
type: engineering
tool: onnx
status: growing
tags: [compression, quantization, pruning, distillation, onnx, optimization]
created: 2026-03-05
---

# Model Compression for Deployment

## Purpose

Large models trained for accuracy often cannot be deployed directly at the inference latency, memory, or cost targets required in production. Model compression is the set of techniques that reduce model size and computational cost while preserving as much predictive performance as possible. The goal is to move models from research quality to production feasibility.

## Architecture

### Quantization
Reduces the numerical precision of weights and/or activations from 32-bit floats to lower bit-width representations.

**Post-Training Quantization (PTQ)**
Applied after training with no retraining required. Two modes:

- **Dynamic quantization**: Weights are quantized to INT8 statically; activations are quantized dynamically at runtime. Low overhead, good for transformer models (BERT, GPT). Apply via:
  ```python
  import torch
  quantized_model = torch.quantization.quantize_dynamic(
      model, {torch.nn.Linear}, dtype=torch.qint8
  )
  ```

- **Static quantization**: Both weights and activations are quantized using calibration data to determine activation ranges. Requires a representative calibration dataset (100–1000 samples). Faster at inference than dynamic but needs the calibration step.

**FP16 (Half Precision)**
Halves memory footprint with minimal accuracy loss on GPUs that support native FP16 arithmetic (NVIDIA Volta+). Common first step before INT8.

**ONNX Runtime Quantization**
```python
from onnxruntime.quantization import quantize_dynamic, QuantType
quantize_dynamic("model.onnx", "model_quant.onnx", weight_type=QuantType.QInt8)
```
Static quantization via `quantize_static` requires a `CalibrationDataReader` implementation.

### Pruning
Removes redundant parameters (weights, neurons, channels) from a trained network.

**Magnitude-Based Unstructured Pruning**
Zeroes out the smallest-magnitude individual weights. Produces sparse weight matrices. Theoretically reduces FLOPs but requires sparse runtime support for wall-clock speedup (e.g., NVIDIA cuSPARSE, sparse ONNX models). In practice, hardware speedup is limited unless sparsity > 90%.

```python
import torch.nn.utils.prune as prune
prune.l1_unstructured(layer, name='weight', amount=0.3)  # prune 30% of weights
```

**Structured Channel Pruning**
Removes entire convolutional filters or attention heads — entire rows/columns of weight matrices. Results in dense (not sparse) smaller networks that run faster on standard hardware without special runtimes. Requires retraining after pruning to recover accuracy. More effective for CNN deployment on CPU/mobile.

### Knowledge Distillation
Trains a small **student** model to mimic the behaviour of a large **teacher** model, rather than learning from hard labels alone.

**Soft Targets**
The teacher's output probability distribution (logits divided by temperature T > 1) is used as training signal. Soft targets carry inter-class relationship information absent from one-hot labels.

Distillation loss:
```
L = α · L_CE(y_true, y_student) + (1 - α) · T² · KL(softmax(z_teacher/T) || softmax(z_student/T))
```
where `z` are logits, `T` is temperature (commonly 2–5), and `α` balances the two terms (commonly 0.1–0.5).

**Task-Specific Distillation**
Rather than distilling on the original training task, the teacher scores an unlabelled dataset relevant to the deployment domain, and the student learns from those pseudo-labels. Effective for domain adaptation at low labelling cost.

**Examples**: DistilBERT (66% of BERT parameters, 97% of BERT performance on GLUE), TinyBERT, MobileNetV3.

## Implementation Notes

### ONNX Export and Optimization

**Export from PyTorch:**
```python
torch.onnx.export(
    model, dummy_input, "model.onnx",
    opset_version=17,
    input_names=["input"], output_names=["output"],
    dynamic_axes={"input": {0: "batch_size"}}
)
```

**Graph Optimizations (ONNX Runtime):**
```python
from onnxruntime import SessionOptions, GraphOptimizationLevel
opts = SessionOptions()
opts.graph_optimization_level = GraphOptimizationLevel.ORT_ENABLE_ALL
session = onnxruntime.InferenceSession("model.onnx", opts)
```

Key optimizations applied:
- **Operator fusion**: Merges consecutive ops (e.g., Conv + BatchNorm + ReLU → fused op) to reduce memory bandwidth.
- **Constant folding**: Evaluates subgraphs with only constant inputs at graph load time.
- **Graph simplification**: Removes redundant Reshape/Transpose/Identity nodes.

**ONNX Model Optimization Tool:**
```bash
python -m onnxruntime.tools.optimizer_cli --input model.onnx --output model_opt.onnx --model_type bert
```

## Trade-offs

| Technique | Accuracy Loss | Speedup | Effort |
|---|---|---|---|
| FP16 quantization | < 0.5% | 1.5–2× (GPU) | Very low |
| INT8 PTQ dynamic | 0.5–2% | 2–4× (CPU) | Low |
| INT8 PTQ static | 0.5–1.5% | 2–4× (CPU) | Medium (calibration) |
| Unstructured pruning | 0.5–3% | Limited (HW) | Medium |
| Structured pruning | 1–5% | 1.5–3× | High (retrain) |
| Knowledge distillation | 1–5% | 2–10× | High (teacher needed) |

Combining techniques (e.g., distillation + INT8 quantization) is common in production pipelines and can achieve near-additive compression ratios with careful tuning.

## References

- Gou et al., "Knowledge Distillation: A Survey", *IJCV* 2021.
- ONNX Runtime Quantization documentation: `onnxruntime.ai/docs/performance/model-optimizations/quantization`.
- Han et al., "Deep Compression: Compressing DNNs with Pruning, Trained Quantization and Huffman Coding", *ICLR* 2016.
- Sanh et al., "DistilBERT, a distilled version of BERT", *NeurIPS 2019 EMC² Workshop*.

## Links
- [[serving_patterns|Model Serving Patterns]]
- [[rollout_strategies|Model Rollout Strategies]]
- [[offline_evaluation|Offline Evaluation]]
- [[experiment_tracking|Experiment Tracking]]
- [[distributed_training|Distributed Training]]
- [[02_modeling/03_deep_learning/neural_network_fundamentals|Neural Network Fundamentals]]
