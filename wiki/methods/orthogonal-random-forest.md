---
title: Orthogonal Random Forests (ORF)
type: method
tags: [orf, orthoforest, local-nuisance, method-family]
related: [generalized-random-forest, honest-forests, double-machine-learning, doubly-robust-learner, residualization, doubly-robust-estimation]
source_files:
  - econml/orf/_ortho_forest.py
  - econml/orf/_causal_tree.py
  - doc/spec/estimation/forest.rst
refs:
  - "Oprescu, Syrgkanis & Wu 2019, ICML"
  - "Friedberg et al. 2018 (local linear forests)"
---

# Orthogonal Random Forests (ORF)

**Import:** `from econml.orf import DMLOrthoForest, DROrthoForest`

## What it is

ORF combines causal forests with [[orthogonal-ml|double ML]] and adds a twist: it estimates the [[nuisance-and-final-models|nuisance]] functions **locally**, around each target point $x$, using the same forest-induced kernel that it uses for the final effect. This local fitting can improve accuracy over the globally-fit [[generalized-random-forest|CausalForestDML]], at higher compute cost (a nuisance model per prediction point). Estimates are asymptotically normal ⇒ valid [[inference-and-confidence-intervals|CIs]] (Bootstrap-of-Little-Bags).

## When to use it

High-dimensional confounders $W$, low-dimensional heterogeneity features $X$, and you want the most accurate nonparametric CATE with CIs and are willing to pay for local nuisance estimation. Two variants by treatment type (mirroring the two moment families):

- **`DMLOrthoForest`** — continuous or discrete treatment; local [[residualization|residual-on-residual]] moment.
- **`DROrthoForest`** — categorical treatment; local [[doubly-robust-estimation|doubly robust]] moment.

## Formal methodology

For target $x$, ORF minimizes a locally-weighted version of the relevant orthogonal loss. For `DMLOrthoForest`:

$$\hat\theta(x) = \arg\min_\theta \sum_i K_x(X_i)\big(Y_i - \hat q_x(X_i,W_i) - \theta\,(T_i - \hat f_x(X_i,W_i))\big)^2$$

with the crucial difference that $\hat q_x, \hat f_x$ are themselves fit by a **locally-weighted penalized regression** (weights $K_x$) in a [[cross-fitting]] manner, so high-dimensional $W$ is handled locally. EconML also implements the **local-linear correction** of Friedberg et al. 2018 (fit a linear-in-$X$ $\theta$ locally with ridge on the slope). The kernel $K_x$ comes from a forest grown with a residualized causal criterion. `DROrthoForest` swaps in the DR moment with local $g_x$ and $p_{x,t}$ nuisances. (`doc/spec/estimation/forest.rst`)

## ORF vs. CausalForestDML / ForestDRLearner

Same moment equations, different nuisance strategy: ORF fits nuisances **locally per target** and *couples* the nuisance and final kernels; the [[generalized-random-forest|forest DML/DR]] estimators fit nuisances **globally** and decouple the kernels. ORF can be more accurate; the global forests are much cheaper. (`doc/spec/estimation/forest.rst`)

## How it maps to code

`econml/orf/_ortho_forest.py` (`DMLOrthoForest`, `DROrthoForest`, `BaseOrthoForest`) with the per-leaf causal tree in `econml/orf/_causal_tree.py`. Nuisances default to `WeightedLasso`/`WeightedLassoCV` (which accept the sample weights $K_x$); wrap any sklearn model with `WeightedModelWrapper` to add weight support. Key params: `n_trees`, `min_leaf_size`, `max_depth`, `subsample_ratio`, `lambda_reg`, `model_Y`/`model_T` (DML) or `propensity_model`/`model_Y` (DR).
