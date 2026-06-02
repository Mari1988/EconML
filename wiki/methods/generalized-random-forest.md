---
title: Generalized Random Forests & Causal Forest
type: method
tags: [grf, causal-forest, causalforestdml, het, mse, method-family]
related: [honest-forests, double-machine-learning, orthogonal-random-forest, cython-forest-stack, inference-and-confidence-intervals]
source_files:
  - econml/grf/classes.py
  - econml/dml/causal_forest.py
  - doc/spec/estimation/forest.rst
refs:
  - "Athey, Tibshirani & Wager 2019, Annals of Statistics"
  - "Wager & Athey 2018, JASA 113(523)"
---

# Generalized Random Forests & Causal Forest

**Import:** `from econml.dml import CausalForestDML` · low-level: `from econml.grf import CausalForest, CausalIVForest, RegressionForest, MultiOutputGRF`

## What it is

A **Generalized Random Forest (GRF)** grows an [[honest-forests|honest, subsampled]] forest whose job is to learn an **adaptive similarity kernel** $K_x(X_i)$, then solves a local [[orthogonal-ml|orthogonal]] moment at each target point $x$. EconML's `CausalForestDML` is the double-ML flavor (Athey et al. 2019, §6.1.1): a fully nonparametric [[double-machine-learning|DML]] whose final model is a causal forest. Supports continuous, binary, or multi-valued discrete treatments and delivers [[inference-and-confidence-intervals|confidence intervals]] via Bootstrap-of-Little-Bags.

## When to use it

You have many features, no idea what the heterogeneity looks like, and you want nonparametric effects *with* valid CIs. It performs automatic featurization (no need to specify `featurizer`) and adapts to low-dimensional structure. See [[choosing-a-method]].

## Formal methodology

The CATE at $x$ solves a locally-weighted residual-on-residual problem ([[residualization]]):

$$\hat\theta(x) = \arg\min_\theta \sum_i K_x(X_i)\big(Y_i - \hat q(X_i,W_i) - \theta\,(T_i - \hat f(X_i,W_i))\big)^2$$

where $K_x$ comes from how often $x$ shares a leaf with $X_i$ across the forest. Unlike the [[orthogonal-random-forest|OrthoForest]], the nuisances $\hat q, \hat f$ are fit **globally** (once), and the kernel used for nuisance fitting is decoupled from the final-stage kernel — cheaper, sometimes slightly less accurate.

### Split criteria: `het` vs `mse`

The forest chooses splits to maximize treatment-effect **heterogeneity** between children:
- **`het`** (Athey et al. 2019): maximize $\theta_1^2 + \theta_2^2$ — pure heterogeneity, ignores within-child treatment variation.
- **`mse`** (EconML addition): maximize $\theta_1^2\sum_{S_1}\tilde T_i^2 + \theta_2^2\sum_{S_2}\tilde T_i^2 \approx \theta_1^2|S_1|\mathrm{Var}_n(T\mid S_1) + \dots$ — penalizes splits that leave a child with little treatment variation. Can differ from `het` in finite samples (`doc/spec/estimation/forest.rst`).

## Related GRF predictors

`econml/grf/classes.py` exposes the raw forests as sklearn-style estimators: `CausalForest`, `CausalIVForest` (forest for [[instrumental-variables|IV]]), `RegressionForest` (the honest regression forest behind `ForestDRLearner`), and `MultiOutputGRF`.

## How it maps to code

`CausalForestDML` (`econml/dml/causal_forest.py`) wires a `CausalForest` final model into the [[ortho-learner-engine|DML engine]]; the forest itself is the high-performance [[cython-forest-stack|Cython stack]] in `econml/grf` + `econml/tree`. Key params: `n_estimators`, `criterion` (`'het'`/`'mse'`), `min_samples_leaf`, `max_depth`, `max_samples` (subsample fraction), `min_var_fraction_leaf` (treatment-variance floor), `honest`, `model_y`, `model_t`. Inference: `inference='blb'`.
