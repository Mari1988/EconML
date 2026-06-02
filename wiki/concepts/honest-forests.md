---
title: Honest Forests & Subsampling
type: concept
tags: [honesty, forests, subsampling, bootstrap-of-little-bags]
related: [generalized-random-forest, orthogonal-random-forest, cross-fitting, inference-and-confidence-intervals, cython-forest-stack]
source_files:
  - econml/grf/classes.py
  - doc/spec/estimation/forest.rst
refs:
  - "Wager & Athey 2018, JASA 113(523)"
  - "Athey, Tibshirani & Wager 2019, Annals of Statistics"
---

# Honest Forests & Subsampling

EconML's forest-based estimators ([[generalized-random-forest]], [[orthogonal-random-forest]], `CausalForestDML`, `ForestDRLearner`) rely on **honesty** and **subsampling** to deliver point estimates *and valid [[inference-and-confidence-intervals|confidence intervals]]* from a non-parametric model. These are what separate a *causal* forest from an ordinary random forest.

## Honesty

A tree is **honest** if the data used to *choose the splits* (the structure) is disjoint from the data used to *estimate the values in the leaves*. Each tree splits its subsample in two halves for these two jobs (a forest analogue of [[cross-fitting]]).

Why it matters: if the same points chose the split *and* set the leaf value, the leaf estimate is biased toward those points (the splits "chase" noise), which invalidates inference. Honesty removes that adaptivity bias, making leaf estimates approximately unbiased and the forest's predictions asymptotically normal (Wager & Athey 2018).

## The forest as an adaptive kernel

A causal forest isn't predicting $Y$ — it's learning a **similarity weight** $K_x(X_i)$: how often target point $x$ lands in the same leaf as training point $X_i$. The [[heterogeneous-treatment-effects|CATE]] at $x$ is then a *locally weighted* solution of the [[orthogonal-ml|orthogonal]] moment (residual-on-residual for DML forests, [[doubly-robust-estimation|DR]] pseudo-outcomes for DR forests). The splitting criterion is chosen to make this kernel group together points with *similar treatment effects* — see the `het` vs `mse` criteria in [[generalized-random-forest]].

## Subsampling and Bootstrap-of-Little-Bags

Trees are grown on **subsamples** (without replacement), not full bootstrap samples. This subsample structure enables the **Bootstrap-of-Little-Bags (BLB)** variance estimator (Athey et al. 2019): by organizing trees into little groups grown on shared half-samples, BLB separates the sampling variance of the forest estimate from Monte-Carlo noise across trees, yielding honest standard errors. This is the `inference='blb'` option for the forest estimators (see [[inference-and-confidence-intervals]]).

## Implementation

The honest, subsampled forests are implemented in a high-performance Cython stack — see [[cython-forest-stack]]. `econml/grf/classes.py` exposes `CausalForest`, `CausalIVForest`, `RegressionForest`, and `MultiOutputGRF` as sklearn-style predictors.
