---
title: The _OrthoLearner Engine
type: architecture
tags: [architecture, ortho-learner, cross-fitting, engine, core]
related: [orthogonal-ml, cross-fitting, nuisance-and-final-models, estimator-class-hierarchy, double-machine-learning, doubly-robust-learner]
source_files:
  - econml/_ortho_learner.py
refs:
  - "Foster & Syrgkanis 2019, arXiv:1901.09036"
  - "Chernozhukov et al. 2016, arXiv:1608.00060"
---

# The `_OrthoLearner` Engine

`econml/_ortho_learner.py::_OrthoLearner` is the single most important class to understand in the codebase: it implements the generic [[orthogonal-ml|orthogonal ML]] loop that the bulk of EconML's estimators ([[double-machine-learning|DML]], [[doubly-robust-learner|DR]], [[iv-methods|IV]], [[dynamic-dml|dynamic]]) inherit. Subclasses supply *what* to estimate; `_OrthoLearner` supplies *how*.

## The generic recipe

It targets any CATE that is the minimizer of a loss (or solution to a moment) depending on [[nuisance-and-final-models|nuisance]] functions $h$:

$$\theta(X) = \arg\min_\theta\ \mathbb{E}[\ell(V; \theta(X), h(V))]$$

and estimates it in three steps:

1. **Cross-fit the nuisances.** For a $K$-fold partition, fit $\hat h_t$ on the training folds and evaluate it on the held-out fold to get out-of-fold nuisance values $\hat U_i = \hat h(V_i)$ for every sample ([[cross-fitting]]).
2. **Fit the final model** by minimizing the plugin loss $\frac1n\sum_i \ell(V_i; \theta(X_i), \hat U_i)$ over the samples that have a nuisance value.
3. **Predict** $\theta(X)$ from the fitted final model.

Because the moment is Neyman-[[orthogonal-ml|orthogonal]] and the nuisances are out-of-fold, $\hat\theta$ inherits $\sqrt n$-rate, asymptotic normality, and valid [[inference-and-confidence-intervals|inference]] even with biased ML nuisances.

## What subclasses provide

A child class implements just two pieces (the docstring spells this out):

- a **nuisance model** object that can `fit` $\hat h$ on a set of samples and `predict` the nuisance value on others, and
- a **final model** object that takes the data plus their estimated nuisance values and fits $\theta(X)$.

Everything else — the fold management, discrete-treatment one-hot expansion, sample weighting, `mc_iters` Monte-Carlo repetition, optional Ray parallelism, and the inference scaffolding — lives in the base class. This is why adding a new estimator is mostly choosing nuisances + a final model rather than writing new cross-fitting code.

## Key constructor parameters (inherited everywhere)

`discrete_treatment`, `discrete_outcome`, `discrete_instrument`, `treatment_featurizer`, `categories` (first = control), `cv` (fold count / splitter), `random_state`, `mc_iters` + `mc_agg` (variance reduction across split re-runs), `allow_missing`, `use_ray`.

## How families specialize it

- **[[double-machine-learning|DML]]** (`_RLearner` → `DML`): nuisances $\mathbb{E}[Y\mid X,W]$, $\mathbb{E}[T\mid X,W]$; final = residual-on-residual regression ([[residualization]]).
- **[[doubly-robust-learner|DR]]**: nuisances = outcome regression + [[propensity-score]]; final = regression on [[doubly-robust-estimation|DR pseudo-outcomes]].
- **[[iv-methods|IV]]**: extra nuisances involving the instrument $Z$; final = orthogonal IV moment.

See [[estimator-class-hierarchy]] for how the inference mixins layer on top.
