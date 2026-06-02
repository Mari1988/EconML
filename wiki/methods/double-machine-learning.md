---
title: Double Machine Learning (DML)
type: method
tags: [dml, rlearner, residualization, method-family]
related: [orthogonal-ml, residualization, cross-fitting, doubly-robust-learner, generalized-random-forest, inference-and-confidence-intervals, ortho-learner-engine]
source_files:
  - econml/dml/dml.py
  - econml/dml/_rlearner.py
  - econml/dml/causal_forest.py
  - doc/spec/estimation/dml.rst
refs:
  - "Chernozhukov et al. 2016, arXiv:1608.00060"
  - "Nie & Wager 2017 (R-learner), arXiv:1712.04912"
  - "Chernozhukov et al. 2017, 2018 (high-dim)"
---

# Double Machine Learning (DML)

**Import:** `from econml.dml import LinearDML, SparseLinearDML, NonParamDML, DML, CausalForestDML, KernelDML`

## What it is

DML (a.k.a. the R-learner) estimates the [[heterogeneous-treatment-effects|CATE]] under [[unconfoundedness]] by [[residualization|partialling out]] the controls. It reduces causal estimation to two prediction tasks — predict $Y$ from $X,W$ and predict $T$ from $X,W$ — then relates the residuals. Works for **any treatment type** (continuous, binary, categorical).

## When to use it

When you observe all confounders, they're high-dimensional or have nonlinear effects, and you want flexible ML in the first stage with (often) valid [[inference-and-confidence-intervals|confidence intervals]]. DML tolerates poor overlap better than [[doubly-robust-learner|DR]] (it needs good overlap only on average), so for continuous treatments it is usually the default. See [[choosing-a-method]].

## Formal methodology

Structural model: $Y = \theta(X)\,T + g(X,W) + \epsilon$, $\;T = f(X,W) + \eta$, with $\mathbb{E}[\epsilon\eta\mid X,W]=0$. Subtracting conditional expectations gives the [[residualization|residual]] relation

$$\tilde Y = \theta(X)\,\tilde T + \epsilon, \qquad \tilde Y = Y - \mathbb{E}[Y\mid X,W],\ \ \tilde T = T - \mathbb{E}[T\mid X,W]$$

so $\hat\theta = \arg\min_\theta \mathbb{E}_n[(\tilde Y - \theta(X)\tilde T)^2]$. The least-squares moment is Neyman-[[orthogonal-ml|orthogonal]] to first-stage error; with [[cross-fitting]], $\hat\theta$ is $\sqrt n$-consistent and asymptotically normal even when the nuisances converge as slowly as $n^{-1/4}$.

## The variants (chosen by final model)

| Class | Final model | Inference | Notes |
|---|---|---|---|
| `LinearDML` | unregularized linear | statsmodels (analytic) | low-dim $\phi(X)$; default CIs |
| `SparseLinearDML` | debiased lasso | debiased-lasso | high-dim features |
| `KernelDML` | RKHS via random Fourier features | — | approximates Nie & Wager RKHS |
| `DML` | any sklearn linear (`model_final=`) | — | fits on $\tilde T \otimes \phi(X)$ |
| `NonParamDML` | any sklearn regressor | — | 1-d/binary $T$; weighted regression, target $\tilde Y/\tilde T$, weight $\tilde T^2$ |
| `CausalForestDML` | causal forest | Bootstrap-of-Little-Bags | nonparametric; see [[generalized-random-forest]] |

## How it maps to code

`econml/dml/_rlearner.py::_RLearner` is the private parent implementing the residual-on-residual fit on top of the [[ortho-learner-engine|_OrthoLearner]]; `econml/dml/dml.py` wraps sklearn models into `DML`/`LinearDML`/`SparseLinearDML`/`KernelDML`/`NonParamDML`; `CausalForestDML` lives in `econml/dml/causal_forest.py`. Nuisances are `model_y` and `model_t`; `featurizer` shapes $\theta(X)$; `discrete_treatment=True` switches `model_t` to a classifier. See [[api-vocabulary]].

## Knobs worth knowing

`cv` (folds; raise to 5–6 for small data), `model_y`/`model_t` (any sklearn estimator, CV estimator, list, or `"auto"` — see [[model-selection-and-scoring]]), `featurizer` (e.g. `PolynomialFeatures` for polynomial heterogeneity, `OneHotEncoder` for fixed effects), `treatment_featurizer` (nonlinear dose response). Inspect fit quality via `score_`, `models_y`, `models_t`. Subclasses of `DML` also support sensitivity analysis (see [[validation-and-sensitivity]]).
