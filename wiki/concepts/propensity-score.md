---
title: Propensity Score
type: concept
tags: [propensity-score, overlap, trimming, discrete-treatment]
related: [doubly-robust-estimation, unconfoundedness, nuisance-and-final-models]
source_files:
  - econml/dr/_drlearner.py
  - doc/spec/estimation/dr.rst
refs:
  - "Crump et al. 2009, Biometrika 96(1)"
---

# Propensity Score

The **propensity score** is the probability of receiving a given (discrete) treatment conditional on the controls:

$$p_t(X, W) = \Pr[T = t \mid X, W]$$

It is the [[nuisance-and-final-models|nuisance]] estimated by `model_propensity` (a classifier exposing `predict_proba`) in the [[doubly-robust-learner|DRLearner]] family and the DR forests. It plays two roles: reweighting observed outcomes to recover [[potential-outcomes|counterfactuals]], and (via the inverse-propensity correction) debiasing the outcome regression in [[doubly-robust-estimation]].

## Overlap and why small propensities hurt

The DR pseudo-outcome divides by $p_t(X,W)$. Where some treatment is **rarely assigned** ($p_t$ near 0) the weight $1/p_t$ blows up, inflating variance. This is the **overlap** (a.k.a. positivity / common support) condition: every treatment must have non-negligible probability across the relevant $X, W$ region. Poor overlap is the main reason to prefer [[double-machine-learning|DML]] over DR (DML needs overlap only on average). See [[choosing-a-method]].

## Trimming

To control the variance from extreme propensities, EconML clips estimated propensities away from 0 and 1 (a threshold in the spirit of Crump et al. 2009). This trades a little bias for much lower variance. The behavior lives in the DR learner's nuisance construction in `econml/dr/_drlearner.py`.

## Relation to unconfoundedness

The propensity score is only a valid adjustment if [[unconfoundedness]] holds — it models treatment assignment from the *observed* controls. If an unobserved confounder drives assignment, no propensity model corrects for it; you need an [[instrumental-variables|instrument]] instead.
