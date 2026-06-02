---
title: Cross-fitting
type: concept
tags: [cross-fitting, sample-splitting, core]
related: [orthogonal-ml, nuisance-and-final-models, ortho-learner-engine, honest-forests]
source_files:
  - econml/_ortho_learner.py
  - doc/spec/estimation/dml.rst
refs:
  - "Chernozhukov et al. 2016, arXiv:1608.00060"
---

# Cross-fitting

**Cross-fitting** (cross-fitted / sample-split nuisance estimation) is the second pillar of [[orthogonal-ml]], alongside Neyman orthogonality. It removes the bias that arises from using the same observations to both *fit* the [[nuisance-and-final-models|nuisance]] models and *evaluate* them in the final stage.

## The mechanism

Partition the data into $K$ folds (the `cv` parameter). For each fold $t$:

1. Fit the nuisance models $\hat h_t$ (e.g. $\hat{\mathbb{E}}[Y\mid X,W]$, $\hat{\mathbb{E}}[T\mid X,W]$) on **all folds except $t$**.
2. Evaluate $\hat h_t$ on fold $t$ to produce the nuisance values $\hat U_i = \hat h_t(V_i)$ for $i$ in fold $t$.

Every observation thus gets a nuisance value computed by a model that **never saw it**. The final-stage model for $\theta(X)$ is then fit on these out-of-fold nuisance values (the union of the held-out folds). This is exactly the loop in `econml/_ortho_learner.py::_OrthoLearner` (see step 2 of its docstring).

## Why it is needed

If $\hat h$ were fit and evaluated on the same points, $\hat h(V_i)$ would be biased toward $V_i$ itself (overfitting), and this bias contaminates the moment condition in a way that doesn't vanish fast enough — breaking the $\sqrt n$-rate and the validity of [[inference-and-confidence-intervals|confidence intervals]]. Sample splitting makes $\hat h_t$ and the evaluation point independent, so the only remaining nuisance error is the genuine estimation error, which Neyman orthogonality already renders second-order.

## Practical notes (EconML)

- **`cv`** sets the number of folds. Default is small (2–3); larger values (5–6) give greater statistical stability on small datasets at higher compute cost (`doc/spec/estimation/dml.rst` FAQ).
- Discrete treatments use `StratifiedKFold`; continuous use `KFold` (with shuffling). Grouped/panel data ([[dynamic-dml]]) uses group-aware splits.
- **`mc_iters`** reruns the whole cross-fitting several times with different splits and aggregates (`mc_agg` = `'mean'`/`'median'`) to reduce the variance introduced by a particular random split.
- The fitted per-fold nuisance models are retained (`models_y`, `models_t`, `models_regression`, …) so you can inspect first-stage goodness of fit.

## Relation to honesty in forests

The forest estimators use a closely related idea — **[[honest-forests|honesty]]** — where the data used to choose tree splits is separated from the data used to estimate leaf values. Both are forms of sample splitting that buy valid inference.
