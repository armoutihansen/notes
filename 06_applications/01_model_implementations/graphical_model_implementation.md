---
layer: 06_applications
type: application
status: growing
tags: [algorithm, reasoning]
created: 2026-05-10
---

# Graphical Model Implementation (pgmpy)

## Purpose

Practical guide for constructing, fitting, and querying Bayesian networks with `pgmpy`. Covers explicit DAG construction from domain knowledge, maximum-likelihood / Bayesian parameter estimation, and exact Variable Elimination inference — the core workflow for probabilistic reasoning in diagnostic, risk-assessment, and causal modelling problems.

### Examples

**Medical diagnosis**: network of symptoms, diseases, and test results; query `P(disease | symptoms)`.

**Risk assessment**: model of customer features, credit score, and default probability; infer posterior over default given observed attributes.

---

## Architecture

```
Domain knowledge / structure learning
        ↓
   BayesianNetwork (DAG)
        ↓
   Parameter fitting (CPTs via MLE or BayesianEstimator)
        ↓
   Inference engine: VariableElimination or BeliefPropagation
        ↓
   Query: P(target | evidence)
```

---

## Setup

```bash
pip install pgmpy networkx matplotlib
```

---

## Manual Network Construction

```python
from pgmpy.models import BayesianNetwork
from pgmpy.factors.discrete import TabularCPD

# Define DAG: each tuple is a directed edge (parent → child)
model = BayesianNetwork([
    ("Rain",      "WetGrass"),
    ("Sprinkler", "WetGrass"),
    ("Season",    "Rain"),
    ("Season",    "Sprinkler"),
])

# CPD for root node 'Season' (no parents)
cpd_season = TabularCPD(
    variable="Season",
    variable_card=2,          # 0 = dry, 1 = wet
    values=[[0.4], [0.6]],
)

# CPD for 'Rain' conditioned on 'Season'
cpd_rain = TabularCPD(
    variable="Rain",
    variable_card=2,
    values=[
        [0.8, 0.2],   # P(Rain=0 | Season=0), P(Rain=0 | Season=1)
        [0.2, 0.8],   # P(Rain=1 | ...)
    ],
    evidence=["Season"],
    evidence_card=[2],
)

# CPD for 'Sprinkler' conditioned on 'Season'
cpd_sprinkler = TabularCPD(
    variable="Sprinkler",
    variable_card=2,
    values=[[0.6, 0.3], [0.4, 0.7]],
    evidence=["Season"],
    evidence_card=[2],
)

# CPD for 'WetGrass' conditioned on 'Rain' and 'Sprinkler' (4 parent combinations)
cpd_wetgrass = TabularCPD(
    variable="WetGrass",
    variable_card=2,
    values=[
        [0.95, 0.10, 0.05, 0.01],   # P(WetGrass=0 | Rain=0,Spr=0), ..., Rain=1,Spr=1
        [0.05, 0.90, 0.95, 0.99],
    ],
    evidence=["Rain", "Sprinkler"],
    evidence_card=[2, 2],
)

model.add_cpds(cpd_season, cpd_rain, cpd_sprinkler, cpd_wetgrass)
assert model.check_model(), "Model structure or CPDs are invalid"
```

---

## Parameter Estimation from Data

```python
import pandas as pd
from pgmpy.estimators import MaximumLikelihoodEstimator, BayesianEstimator

# Simulate observed data
data = pd.DataFrame({
    "Season":     [0, 1, 0, 1, 1, 0, 1, 0, 0, 1],
    "Rain":       [0, 1, 0, 1, 1, 0, 1, 0, 0, 1],
    "Sprinkler":  [1, 0, 1, 1, 0, 1, 0, 0, 1, 1],
    "WetGrass":   [1, 1, 0, 1, 1, 1, 1, 0, 1, 1],
})

# Maximum likelihood — may overfit with small data (CPT entries can be 0)
model_mle = BayesianNetwork([("Season", "Rain"), ("Season", "Sprinkler"), ("Rain", "WetGrass"), ("Sprinkler", "WetGrass")])
model_mle.fit(data, estimator=MaximumLikelihoodEstimator)

# Bayesian estimation — adds prior pseudocounts to smooth CPT estimates
model_bayes = BayesianNetwork([("Season", "Rain"), ("Season", "Sprinkler"), ("Rain", "WetGrass"), ("Sprinkler", "WetGrass")])
model_bayes.fit(
    data,
    estimator=BayesianEstimator,
    prior_type="BDeu",      # Bayesian Dirichlet equivalent uniform prior
    equivalent_sample_size=5,   # strength of prior (virtual sample count)
)

# Inspect learned CPDs
for cpd in model_bayes.cpds:
    print(cpd)
```

---

## Structure Learning

```python
from pgmpy.estimators import PC, HillClimbSearch, BicScore

# Constraint-based (PC algorithm): tests conditional independence
pc = PC(data)
model_pc = pc.estimate(significance_level=0.05)
print("PC edges:", model_pc.edges())

# Score-based (Hill Climb + BIC): searches over DAG space
hc = HillClimbSearch(data)
model_hc = hc.estimate(scoring_method=BicScore(data))
print("HC edges:", model_hc.edges())
```

---

## Inference with Variable Elimination

```python
from pgmpy.inference import VariableElimination

# Use the manually constructed and CPD-fitted model
infer = VariableElimination(model)

# Query: P(Rain | WetGrass=1) — observe wet grass, infer rain probability
q1 = infer.query(variables=["Rain"], evidence={"WetGrass": 1})
print(q1)
# +----------+-------------+
# | Rain     | phi(Rain)   |
# +==========+=============+
# | Rain(0)  | 0.3050      |
# | Rain(1)  | 0.6950      |
# +----------+-------------+

# MAP query: most likely state for multiple variables given evidence
map_query = infer.map_query(variables=["Rain", "Sprinkler"], evidence={"WetGrass": 1})
print("MAP:", map_query)  # {'Rain': 1, 'Sprinkler': 0}
```

---

## Conditional Independence Testing

```python
# Check if model satisfies d-separation constraints
# Is 'Rain' independent of 'Sprinkler' given 'Season'?
print(model.local_independencies("Rain"))
print(model.get_independencies())
```

---

## Worked Example: Student Grade Network

Classic BN from Koller & Friedman (2009):

```python
# D → G ← I, I → S, G → L
student_model = BayesianNetwork([
    ("Difficulty", "Grade"),
    ("Intelligence", "Grade"),
    ("Intelligence", "SAT"),
    ("Grade", "Letter"),
])

cpd_d = TabularCPD("Difficulty",    2, [[0.6], [0.4]])
cpd_i = TabularCPD("Intelligence",  2, [[0.7], [0.3]])
cpd_g = TabularCPD("Grade", 3, [[0.3, 0.05, 0.9, 0.5],
                                 [0.4, 0.25, 0.08, 0.3],
                                 [0.3, 0.70, 0.02, 0.2]],
                   evidence=["Difficulty", "Intelligence"], evidence_card=[2, 2])
cpd_s = TabularCPD("SAT",    2, [[0.95, 0.2], [0.05, 0.8]], evidence=["Intelligence"], evidence_card=[2])
cpd_l = TabularCPD("Letter", 2, [[0.1, 0.4, 0.99], [0.9, 0.6, 0.01]], evidence=["Grade"], evidence_card=[3])

student_model.add_cpds(cpd_d, cpd_i, cpd_g, cpd_s, cpd_l)
assert student_model.check_model()

infer_s = VariableElimination(student_model)
# P(Intelligence | Letter=1, Grade=1)
result = infer_s.query(["Intelligence"], evidence={"Letter": 1, "Grade": 1})
print(result)
```

---

## Links

**Modeling**
- [[02_modeling/03_model_families/08_graphical_models/graphical_models|Graphical Models]] — Bayesian networks, MRFs, HMMs, d-separation theory
- [[01_foundations/03_probability_and_statistics/bayesian_inference|Bayesian Inference]] — prior, likelihood, posterior framework
- [[01_foundations/03_probability_and_statistics/probability_theory|Probability Theory]] — conditional probability and chain rule

**Applications**
- [[probabilistic_models_implementation|Probabilistic Models Implementation]] — GMM, HMM, Naive Bayes with scikit-learn
