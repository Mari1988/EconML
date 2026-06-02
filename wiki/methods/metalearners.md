---
title: Meta-Learners (S / T / X / Domain Adaptation)
type: method
tags: [metalearners, slearner, tlearner, xlearner, method-family, discrete-treatment]
related: [heterogeneous-treatment-effects, propensity-score, doubly-robust-learner, double-machine-learning, model-selection-and-scoring]
source_files:
  - econml/metalearners/_metalearners.py
  - doc/spec/estimation/metalearners.rst
refs:
  - "Künzel et al. 2017, arXiv:1706.03461"
---

# Meta-Learners

**Import:** `from econml.metalearners import SLearner, TLearner, XLearner, DomainAdaptationLearner`

## What it is

Meta-learners estimate the [[heterogeneous-treatment-effects|CATE]] for **discrete treatments** by combining off-the-shelf ML regressors/classifiers in a black box — they model the response surfaces $Y^{(t)}$ directly and difference them, introducing no new estimation machinery (Künzel et al. 2017). Maximum flexibility and trivial model selection, but generally **no valid analytic [[inference-and-confidence-intervals|confidence intervals]]** (bootstrap only).

## When to use it

When the goal is a low-MSE CATE and easy cross-validated [[model-selection-and-scoring|model selection]] at every stage, and you don't need honest CIs. If you do need inference for discrete treatments, prefer [[doubly-robust-learner|DRLearner]] or [[double-machine-learning|NonParamDML]] (themselves "meta-learners" in the sense of accepting arbitrary ML, but with orthogonal moments). See [[choosing-a-method]].

## The four learners (binary-treatment sketch)

- **S-learner** (`SLearner`): one model $\hat\mu(x,t)$ with treatment as an input feature; $\hat\tau(x)=\hat\mu(x,1)-\hat\mu(x,0)$. Simple; can under-detect effects if the model shrinks the $T$ feature.
- **T-learner** (`TLearner`): separate models $\hat\mu_0,\hat\mu_1$ per treatment arm; $\hat\tau(x)=\hat\mu_1(x)-\hat\mu_0(x)$. No effect-shrinkage, but doesn't pool across arms.
- **X-learner** (`XLearner`): impute each arm's counterfactual with the *other* arm's model, regress the imputed effects ($\hat\tau_0,\hat\tau_1$), and combine via a [[propensity-score|propensity]] weight $g(x)$. Strong when arm sizes are imbalanced.
- **Domain Adaptation learner** (`DomainAdaptationLearner`): an X-learner variant that reweights each arm's training samples by propensity-derived weights $\tfrac{g}{1-g}$ / $\tfrac{1-g}{g}$ to correct for the differing covariate distributions $P(X^0)\neq P(X^1)$ before imputing.

## How it maps to code

`econml/metalearners/_metalearners.py`. Constructor models: `SLearner(overall_model=…)`, `TLearner(models=…)`, `XLearner(models=…, cate_models=…, propensity_model=…)`, `DomainAdaptationLearner(models=…, final_models=…, propensity_model=…)`. Any sklearn regressor/classifier works; pass cross-validated estimators for automatic tuning. Unlike the [[ortho-learner-engine|_OrthoLearner]] family they do **not** cross-fit an orthogonal moment — which is exactly why their inference is bootstrap-only. All extend to multiple categorical treatments.

> Note: features and controls are passed together as `X` to meta-learners (e.g. `fit(Y, T, X=np.hstack([X, W]))`); they don't take a separate `W`.
