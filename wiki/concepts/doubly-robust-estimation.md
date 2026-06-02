---
title: Doubly Robust Estimation
type: concept
tags: [doubly-robust, aipw, pseudo-outcome, propensity]
related: [propensity-score, orthogonal-ml, doubly-robust-learner, potential-outcomes, residualization]
source_files:
  - econml/dr/_drlearner.py
  - doc/spec/estimation/dr.rst
refs:
  - "Robins, Rotnitzky & Zhao 1994"
  - "Bang & Robins 2005"
  - "Foster & Syrgkanis 2019, arXiv:1901.09036"
---

# Doubly Robust Estimation

**Doubly robust (DR)** estimation, also called augmented inverse-propensity weighting (AIPW), constructs an unbiased estimate of each [[potential-outcomes|potential outcome]] by combining two models so that the result is correct if *either* one is correct. It is the basis of the [[doubly-robust-learner]] and DR forests, and applies to **discrete (categorical) treatments**.

## Two ingredients, two failure modes it survives

- **Direct/regression model** $g_t(X,W) = \mathbb{E}[Y\mid T=t, X, W]$. Estimating $\theta_t$ from $g_t$ alone (the *direct method*) relies entirely on model-based extrapolation across treatment groups — fragile if the regression is misspecified.
- **[[propensity-score]] model** $p_t(X,W) = \Pr[T=t\mid X,W]$. Reweighting observed outcomes by $1/p_t$ (the *inverse propensity* method) is unbiased but high-variance, especially where some treatment is rare.

## The doubly robust pseudo-outcome

DR fits the direct regression, then **debiases it with an inverse-propensity correction on its own residual**:

$$Y_{i,t}^{DR} = g_t(X_i, W_i) + \frac{Y_i - g_t(X_i, W_i)}{p_t(X_i, W_i)}\,\mathbf{1}\{T_i = t\}$$

Then the [[heterogeneous-treatment-effects|CATE]] is estimated by regressing $Y_{i,t}^{DR} - Y_{i,0}^{DR}$ on $X$.

## Why "doubly robust"

$\mathbb{E}[Y_{i,t}^{DR}\mid X,W] = \mathbb{E}[Y^{(t)}\mid X,W]$ holds if **either** $g_t$ **or** $p_t$ is correct. Stronger still: the error of the final estimate depends only on the **product** of the two nuisance errors. So if both converge faster than $n^{-1/4}$, the final estimate reaches the parametric $n^{-1/2}$ rate and is asymptotically normal. This is a property slightly stronger than generic Neyman [[orthogonal-ml|orthogonality]] (`doc/spec/estimation/dr.rst`).

## Trade-off vs. DML

DR's reliance on $1/p_t$ makes it **higher variance under poor overlap** (regions where a treatment is nearly never assigned). [[double-machine-learning|DML]], which only needs good overlap "on average," can extrapolate better there. On the other hand, DR's final regression remains meaningful even when the true CATE isn't in the model class — it then estimates the **projection** of the CATE onto that class, enabling honest inference on, e.g., the best linear approximation. See [[choosing-a-method]].

## Beyond effect estimation

The same DR pseudo-outcomes are the foundation for offline [[policy-learning]] (Dudík et al. 2014; Athey & Wager 2017), where they give low-variance estimates of the value of a candidate policy.
