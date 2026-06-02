---
title: Orthogonal / Double Machine Learning (the principle)
type: concept
tags: [orthogonal-ml, neyman-orthogonality, double-ml, core]
related: [cross-fitting, nuisance-and-final-models, residualization, doubly-robust-estimation, ortho-learner-engine]
source_files:
  - econml/_ortho_learner.py
  - doc/spec/estimation/dml.rst
refs:
  - "Chernozhukov et al. 2016, arXiv:1608.00060"
  - "Foster & Syrgkanis 2019, arXiv:1901.09036"
  - "Mackey et al. 2017, arXiv:1711.00342"
---

# Orthogonal / Double Machine Learning (the principle)

**Orthogonal (a.k.a. double) machine learning** is the theoretical principle underpinning most of EconML. It lets you plug *arbitrary, biased* ML models into the steps of a causal estimator while still getting a final effect estimate that is $\sqrt{n}$-consistent and asymptotically normal (so you can build [[inference-and-confidence-intervals|confidence intervals]]). This is the idea the [[ortho-learner-engine|_OrthoLearner]] base class operationalizes for the whole library.

## The problem it solves

A causal estimator depends on **[[nuisance-and-final-models|nuisance functions]]** $h$ — e.g. $\mathbb{E}[Y\mid X,W]$ and $\mathbb{E}[T\mid X,W]$ — that we must estimate with ML. ML models trade bias for variance (regularization, early stopping), so $\hat h$ carries first-order bias. Naively plugging $\hat h$ into the effect estimate propagates that bias at rate $O(\|\hat h - h\|)$, which for flexible ML is too slow for valid inference.

## Neyman orthogonality

The fix is to estimate the effect $\theta$ as the solution to a moment condition $\mathbb{E}[\psi(V; \theta, h)] = 0$ that is **Neyman orthogonal**: its gradient with respect to the nuisance $h$, evaluated at the truth, is zero.

$$\left. \partial_h\, \mathbb{E}[\psi(V; \theta, h)] \right|_{h=h_0} = 0$$

When this holds, a small error in $\hat h$ has only a *second-order* effect on $\hat\theta$. The estimation error in $\theta$ scales with $\|\hat h - h\|^2$, so nuisance models that converge at merely $n^{-1/4}$ (achievable by many ML methods) still yield a $\theta$ that converges at the parametric $n^{-1/2}$ rate. See Chernozhukov et al. 2016, Foster & Syrgkanis 2019.

## Where the orthogonality comes from

In [[double-machine-learning|DML]], orthogonality is achieved by **[[residualization]]**: regress out $X,W$ from both $Y$ and $T$ and relate the residuals $\tilde Y = \theta(X)\tilde T + \epsilon$. The least-squares moment for this residual-on-residual regression is automatically orthogonal to errors in the two conditional-expectation nuisances.

In [[doubly-robust-estimation|DR]] methods, orthogonality comes from the augmented (doubly robust) pseudo-outcome, which is orthogonal to errors in *both* the outcome regression and the [[propensity-score]] — in fact slightly stronger than Neyman orthogonality (only the *product* of the two nuisance errors matters).

## The second ingredient: cross-fitting

Orthogonality removes first-order *bias* but not the *own-observation* overfitting bias that arises from using the same data to fit $\hat h$ and to evaluate the moment. [[cross-fitting]] (fit nuisances on one fold, evaluate on the held-out fold) eliminates that, completing the recipe. Orthogonality + cross-fitting together are what make "double ML" work.

## In code

`econml/_ortho_learner.py::_OrthoLearner` implements the generic loop: cross-fit the nuisances, then minimize the orthogonal plugin loss for the final model. Every DML, DR, and IV estimator is a thin subclass that only specifies *which* nuisances and *which* orthogonal moment to use — see [[ortho-learner-engine]].
