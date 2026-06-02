---
title: The Cython Forest Stack
type: architecture
tags: [architecture, cython, forests, grf, tree, performance]
related: [generalized-random-forest, honest-forests, orthogonal-random-forest, policy-learning]
source_files:
  - econml/tree/
  - econml/grf/
  - econml/policy/_forest/
---

# The Cython Forest Stack

EconML's forest estimators are backed by a three-layer, high-performance **Cython** implementation (forked and adapted from scikit-learn's tree code). Editing any `.pyx`/`.pxd` requires recompiling (`pip install -e .`); see the root `CLAUDE.md`.

## The three layers

1. **`econml/tree/`** — the low-level decision tree: `_tree.pyx` (tree builder), `_splitter.pyx` (split search), `_criterion.pyx` (split scoring), `_utils.pyx`. This is generic infrastructure supporting [[honest-forests|honest]] splitting.
2. **`econml/grf/`** — Generalized Random Forests built on `econml/tree`. `_base_grf.py`, `_base_grftree.py`, a GRF `_criterion.pyx`, and the sklearn-style predictors in `classes.py`: `CausalForest`, `CausalIVForest`, `RegressionForest`, `MultiOutputGRF`. Backs [[generalized-random-forest|CausalForestDML]] and (via `RegressionForest`) `ForestDRLearner`.
3. **`econml/policy/_forest/`** — the policy forest with its own `_criterion.pyx`, used by [[policy-learning|DRPolicyForest/PolicyForest]].

## Why a separate forest implementation

Standard random forests minimize prediction error of $Y$. Causal forests instead need to:
- score splits by a **causal criterion** (`het` heterogeneity vs `mse` treatment-variance-penalized — see [[generalized-random-forest]]), solving a local moment at each candidate split;
- enforce **honesty** (split-set / value-set separation) and **subsampling** for valid inference;
- support the **Bootstrap-of-Little-Bags** variance estimate.

None of these fit sklearn's tree API, hence the fork. The `.pxd` files are Cython headers shared across the `.pyx` modules; a change there ripples into dependents and forces recompilation.

## Where the forests surface

The Cython predictors are wrapped by Python CATE estimators that add the [[cross-fitting]] / orthogonal-moment layer: `CausalForestDML` (in `econml/dml/causal_forest.py`), `ForestDRLearner`, and the locally-fit [[orthogonal-random-forest|OrthoForest]] estimators in `econml/orf/`.
