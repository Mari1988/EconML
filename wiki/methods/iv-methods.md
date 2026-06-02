---
title: Instrumental-Variable Estimators (OrthoIV, DMLIV, DRIV)
type: method
tags: [iv, orthoiv, dmliv, driv, intent-to-treat, deepiv, method-family]
related: [instrumental-variables, double-machine-learning, doubly-robust-estimation, orthogonal-ml, inference-and-confidence-intervals]
source_files:
  - econml/iv/dml/_dml.py
  - econml/iv/dr/_dr.py
  - econml/iv/sieve/
  - doc/spec/estimation/orthoiv.rst
  - doc/spec/estimation/two_sls.rst
refs:
  - "Syrgkanis et al. 2019, arXiv:1905.10176"
  - "Hartford et al. 2017 (Deep IV)"
  - "Newey & Powell 2003"
---

# Instrumental-Variable Estimators

**Import:** `from econml.iv.dml import OrthoIV, DMLIV, NonParamDMLIV` · `from econml.iv.dr import DRIV, LinearDRIV, SparseLinearDRIV, ForestDRIV, IntentToTreatDRIV, LinearIntentToTreatDRIV` · `from econml.iv.sieve import SieveTSLS, ...`

## What it is

A suite of estimators that recover the [[heterogeneous-treatment-effects|CATE]] when [[unconfoundedness]] **fails** but a valid [[instrumental-variables|instrument]] $Z$ is available. They cast IV effect estimation as minimizing a loss over auxiliary [[nuisance-and-final-models|nuisance]] regressions (of $Y$, $T$, $Z$ on $X,W$) built to satisfy a Neyman [[orthogonal-ml|orthogonality]] condition, so arbitrary ML can be used while parametric final models stay asymptotically normal (Syrgkanis et al. 2019). All take a `Z=` argument in `fit`.

## When to use it

Unobserved confounding of $T$ and $Y$, plus an observed $Z$ satisfying relevance + exclusion + independence. Especially natural for **intent-to-treat** A/B tests: randomize *who is encouraged/recommended* ($Z$) to take an action ($T$) and estimate the effect of the action. See [[instrumental-variables]] and [[choosing-a-method]].

## The two sub-families

**`econml/iv/dml`** — orthogonal DML-IV:
- `OrthoIV` — orthogonal moment, parametric final, analytic CIs; `projection` toggles residualization vs projection of $T$ on $Z$.
- `DMLIV` / `NonParamDMLIV` — DML-style IV with linear / arbitrary final model.

**`econml/iv/dr`** — [[doubly-robust-estimation|doubly robust]] IV (more robust moment; mostly 1-d/binary $T$):
- `DRIV`, `LinearDRIV`, `SparseLinearDRIV`, `ForestDRIV` — DR-IV with nonparametric / linear / sparse / forest final models and corresponding [[inference-and-confidence-intervals|inference]].
- `IntentToTreatDRIV`, `LinearIntentToTreatDRIV` — specialized for binary instrument + binary treatment encouragement designs.

**`econml/iv/sieve`** — `SieveTSLS` and Deep IV style approaches: classical two-stage / sieve nonparametric IV (Newey & Powell 2003; Hartford et al. 2017). `SieveTSLS` handles any treatment type with an assumed-linear structure (`doc/spec/estimation/two_sls.rst`).

## How it maps to code

These extend the [[ortho-learner-engine|_OrthoLearner]] with instrument-aware nuisances. Set `discrete_treatment`/`discrete_instrument` as appropriate; nuisance models follow the naming `model_y_xw`, `model_t_xw`, `model_t_xwz`, `flexible_model_effect`, etc. The detailed comparison matrix (treatment type, CI availability) is in [[choosing-a-method]] / `doc/spec/comparison.rst`.
