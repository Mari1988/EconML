---
title: Nuisance and Final Models
type: concept
tags: [nuisance, final-model, two-stage, core]
related: [orthogonal-ml, cross-fitting, residualization, model-selection-and-scoring, ortho-learner-engine]
source_files:
  - econml/_ortho_learner.py
  - doc/spec/model_selection.rst
refs:
  - "Foster & Syrgkanis 2019, arXiv:1901.09036"
---

# Nuisance and Final Models

EconML estimators are **two-stage**. The vocabulary recurs everywhere, so it gets its own page.

## Nuisance (first-stage) models

**Nuisance functions** are quantities we don't care about for their own sake but must estimate to get at the effect. They are pure *predictive* tasks, so any sklearn-compatible ML model works. The common ones:

- **Outcome model** $q(X,W) = \mathbb{E}[Y\mid X,W]$ — `model_y` (DML) / part of `model_regression` (DR).
- **Treatment model** $f(X,W) = \mathbb{E}[T\mid X,W]$ — `model_t`; for discrete $T$ this is the [[propensity-score]] `model_propensity`, a classifier with `predict_proba`.
- (DR) **outcome-given-treatment** $g_t(X,W) = \mathbb{E}[Y\mid T=t,X,W]$ — `model_regression`.

These are fit in a [[cross-fitting]] manner. Because they are "just prediction," you choose them by predictive performance and can use cross-validated estimators (e.g. `LassoCV`), grid search, lists of candidates, or the `"auto"`/`"automl"` keywords for automatic [[model-selection-and-scoring|model selection]] (`doc/spec/model_selection.rst`).

## Final (second-stage) model

The **final model** fits the [[heterogeneous-treatment-effects|CATE]] $\theta(X)$ from the de-confounded signal produced by the first stage (residuals in [[double-machine-learning|DML]], doubly robust pseudo-outcomes in [[doubly-robust-learner|DR]]). The *choice of final model is what distinguishes the estimators within a family*:

- unregularized linear → `LinearDML` / `LinearDRLearner` (gives [[inference-and-confidence-intervals|analytic CIs]] via statsmodels)
- $\ell_1$ / debiased lasso → `SparseLinearDML` / `SparseLinearDRLearner`
- forest → `CausalForestDML` / `ForestDRLearner`
- arbitrary sklearn regressor → `NonParamDML` / `DRLearner`

## Why the split matters

Putting flexible ML in the nuisance stage and a (often simpler, inference-friendly) model in the final stage is exactly what [[orthogonal-ml]] needs: the orthogonal moment makes the final estimate robust to first-stage error, and [[cross-fitting]] keeps the stages independent. The engine that wires the two stages together generically is the [[ortho-learner-engine|_OrthoLearner]].
