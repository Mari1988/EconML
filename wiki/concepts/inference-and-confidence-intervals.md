---
title: Inference & Confidence Intervals
type: concept
tags: [inference, confidence-intervals, bootstrap, debiased-lasso, blb, statsmodels]
related: [orthogonal-ml, cross-fitting, honest-forests, api-vocabulary, double-machine-learning, doubly-robust-learner]
source_files:
  - econml/inference/_inference.py
  - econml/inference/_bootstrap.py
  - doc/spec/inference.rst
refs:
  - "Chernozhukov et al. 2016, arXiv:1608.00060"
  - "Bühlmann & van de Geer 2011 (debiased lasso)"
  - "Athey, Tibshirani & Wager 2019 (BLB)"
---

# Inference & Confidence Intervals

A distinguishing feature of EconML is that many estimators provide **valid confidence intervals and p-values for the CATE**, correctly accounting for the multi-stage ([[nuisance-and-final-models|nuisance]] + final) estimation. Inference is selected via the `inference=` argument to `fit()` and surfaced through `effect_interval`, `effect_inference`, `const_marginal_effect_interval`, etc. (see [[api-vocabulary]]).

The validity rests on [[orthogonal-ml|orthogonality]] + [[cross-fitting]]: those are what make the final estimate asymptotically normal despite biased ML first stages. **Caveat:** intervals are only valid if the final model is correctly specified — a linear final model on a nonlinear truth gives misleadingly tight intervals (`doc/spec/validation.rst`). See [[validation-and-sensitivity]].

## The four inference backends

| `inference=` | Method | Enabled for | Backed by |
|---|---|---|---|
| `'bootstrap'` / `BootstrapInference(...)` | Refit on resamples; quantiles of the estimate distribution | **any** estimator | `econml/inference/_bootstrap.py` |
| `'statsmodels'` (`'auto'`) | OLS asymptotic normality | linear final stage: `LinearDML`, `LinearDRLearner` | `StatsModelsLinearRegression` |
| `'debiasedlasso'` (`'auto'`) | Debiased-lasso normality for sparse high-dim final models | `SparseLinearDML`, `SparseLinearDRLearner` | `DebiasedLasso` (Bühlmann & van de Geer 2011) |
| `'blb'` (`'auto'`) | Bootstrap-of-Little-Bags for honest forests | `CausalForestDML`, `ForestDRLearner`, `DMLOrthoForest`, `DROrthoForest` | [[honest-forests]] / `RegressionForest` |

`inference='auto'` (the default) picks the analytic backend appropriate to the estimator's final model. Setting `inference=None` disables interval machinery for speed.

## Bootstrap: universal but slow

Bootstrap works for *any* estimator (including [[metalearners]] that lack analytic inference) by retraining many clones on resampled data. It is computationally expensive and, for estimators that trade honest inference for flexibility, not guaranteed valid (`doc/spec/comparison.rst`). Prefer an analytic backend when the estimator offers one.

## Inference result objects

`effect_inference(...)` returns a rich `InferenceResults` object (`econml/inference/_inference.py`): `NormalInferenceResults` (analytic), `EmpiricalInferenceResults` (bootstrap), with `summary_frame()`, `population_summary()`, and — for linear parametric final models — a `summary()` on the estimator. These expose standard errors, z-scores, p-values, and intervals at any `alpha`.

## Which estimators deliver analytic CIs

See the matrix in [[choosing-a-method]] (from `doc/spec/comparison.rst`): the `Linear*`, `SparseLinear*`, forest, and IV estimators do; the bare [[metalearners]] and `DML`/`DRLearner`/`NonParamDML` with arbitrary final models generally fall back to bootstrap.
