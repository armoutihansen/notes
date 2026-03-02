---
layer: 01_foundations
type: concept
status: seed
tags: [orthogonalization, ml-process, debugging, early-stopping]
created: 2026-03-02
---

# Orthogonalization

## Definition

A design principle for ML development where each intervention targets exactly one failure mode, keeping the effects of different controls independent.

## Intuition

Like orthogonal dials on a mixing board — turning one should not affect others; if fixing bias also changes variance, you lose diagnostic clarity; early stopping is an example of a non-orthogonal tool (it entangles optimization and regularization).

## Formal Description

The ML development chain has four distinct failure modes, each with its own orthogonal set of fixes:

| Step | Failure mode | Orthogonal fixes |
|------|-------------|-----------------|
| 1 | Does not fit training set well | Increase model capacity, tune optimizer, add features |
| 2 | Does not generalize to dev set | More data, regularization, reduce model complexity |
| 3 | Does not generalize to test set | Larger/better dev set (possible overfit to dev) |
| 4 | Does not work in real world | Revisit dev/test distribution, re-examine metrics |

Applying a fix from step 2 to a step 1 problem is wasteful and can mask the real issue.

**Early stopping is non-orthogonal:** stopping early reduces training error (step 1) while simultaneously affecting generalization (step 2), making both harder to diagnose independently.

## Applications

Structuring the ML iteration loop; debugging stalled training; deciding which knob to turn next.

## Trade-offs

- Orthogonality is an ideal — real interventions always have some side effects
- Useful as a heuristic, not a rigid rule

## Links
- [[bias_variance_analysis]]
- [[evaluation_metrics]]
- [[error_analysis]]
