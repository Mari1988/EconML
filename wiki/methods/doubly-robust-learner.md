---
title: Doubly Robust Learner (DRLearner)
type: method
tags: [dr, doubly-robust, propensity, method-family, discrete-treatment]
related: [doubly-robust-estimation, propensity-score, double-machine-learning, generalized-random-forest, metalearners, inference-and-confidence-intervals]
source_files:
  - econml/dr/_drlearner.py
  - doc/spec/estimation/dr.rst
refs:
  - "Foster & Syrgkanis 2019, arXiv:1901.09036"
  - "Bang & Robins 2005"
  - "Chernozhukov et al. 2016"
---

# Doubly Robust Learner (DRLearner)

**Import:** `from econml.dr import DRLearner, LinearDRLearner, SparseLinearDRLearner, ForestDRLearner`

## What it is

The DRLearner estimates the [[heterogeneous-treatment-effects|CATE]] for **categorical treatments** under [[unconfoundedness]] by building [[doubly-robust-estimation|doubly robust pseudo-outcomes]] and regressing them on $X$. It needs two first-stage models — an outcome regression and a [[propensity-score]] — and is correct if *either* is correct.

## When to use it

Discrete/categorical treatment, all confounders observed, and you want a method whose final regression is meaningful even under misspecification (it estimates the *projection* of the CATE onto the final model class — enabling honest inference on, e.g., a best linear approximation). Prefer [[double-machine-learning|DML]] instead when overlap is poor, since DR's $1/p_t$ weighting inflates variance there. See [[choosing-a-method]].

## Formal methodology

Under potential outcomes with $g_t(X,W)=\mathbb{E}[Y\mid T=t,X,W]$ and $p_t(X,W)=\Pr[T=t\mid X,W]$:

$$Y_{i,t}^{DR} = g_t(X_i,W_i) + \frac{Y_i - g_t(X_i,W_i)}{p_t(X_i,W_i)}\mathbf{1}\{T_i=t\}, \qquad \theta_t(X)=\mathbb{E}[Y_{i,t}^{DR}-Y_{i,0}^{DR}\mid X]$$

Effects are relative to a **baseline treatment 0**. The DR moment is orthogonal — in fact only the *product* of the outcome- and propensity-model errors enters the final error, so both converging faster than $n^{-1/4}$ yields parametric-rate, asymptotically normal estimates. Requires [[cross-fitting]]. (`doc/spec/estimation/dr.rst`)

## The variants (chosen by final model)

| Class | Final model | Inference |
|---|---|---|
| `LinearDRLearner` | unregularized linear | statsmodels (analytic) |
| `SparseLinearDRLearner` | debiased lasso | debiased-lasso |
| `ForestDRLearner` | subsampled honest `RegressionForest` | Bootstrap-of-Little-Bags |
| `DRLearner` | any sklearn regressor (`model_final=`) | bootstrap only — also a [[metalearners|meta-learner]] |

## How it maps to code

`econml/dr/_drlearner.py` implements the family on the [[ortho-learner-engine|_OrthoLearner]]. Nuisances: `model_regression` (outcome given $T,X,W$) and `model_propensity` (classifier with `predict_proba`); propensities are trimmed for overlap (see [[propensity-score]]). `ForestDRLearner` is closely related to the [[orthogonal-random-forest|DROrthoForest]] but fits nuisances globally rather than locally (`doc/spec/estimation/forest.rst`).

## Knobs worth knowing

`model_regression`, `model_propensity`, `model_final`, `cv`, `featurizer`, `min_propensity` (trimming floor). Effects via `effect(X, T0, T1)` or per-treatment `const_marginal_effect(X)`; `coef__interval(T=...)` for the linear variants. Subclasses of `DRLearner` support sensitivity analysis — see [[validation-and-sensitivity]].
