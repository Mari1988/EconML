---
title: Policy Learning
type: method
tags: [policy-learning, drpolicy, policy-tree, policy-forest, method-family]
related: [doubly-robust-estimation, doubly-robust-learner, interpretability, heterogeneous-treatment-effects]
source_files:
  - econml/policy/_drlearner.py
  - econml/policy/_base.py
  - econml/policy/_forest/
refs:
  - "Athey & Wager 2017 (efficient policy learning), arXiv:1702.02896"
  - "Dudík et al. 2014"
---

# Policy Learning

**Import:** `from econml.policy import DRPolicyTree, DRPolicyForest, PolicyTree, PolicyForest`

## What it is

Policy learning directly learns a **treatment-assignment rule** $\pi(X) \to$ recommended treatment, optimizing the expected outcome of *following* the rule — rather than first estimating a [[heterogeneous-treatment-effects|CATE]] and thresholding it. EconML's policy learners use [[doubly-robust-estimation|doubly robust]] value estimates to do this offline from observational/experimental data (Athey & Wager 2017; Dudík et al. 2014).

## When to use it

When the deliverable is the *decision* ("treat whom?"), not the effect size — and especially when you want an **interpretable** policy. The tree learners output a shallow, human-readable decision tree of treatment recommendations. Treatment costs can be incorporated so the policy only treats where the effect exceeds cost.

## How it works

`DRPolicyTree` / `DRPolicyForest` construct doubly robust per-unit outcome estimates (an outcome regression + [[propensity-score]] correction, as in the [[doubly-robust-learner|DRLearner]]), then fit a policy tree/forest that maximizes the DR-estimated policy value. `PolicyTree` / `PolicyForest` are the underlying optimizers that take precomputed value estimates. The forest variant averages many honest policy trees; the single tree is fully interpretable and `plot()`-able. Compare with the *post-hoc* `SingleTreePolicyInterpreter`, which extracts a policy from an already-fitted CATE model — see [[interpretability]].

## How it maps to code

`econml/policy/_drlearner.py` (`DRPolicyTree`, `DRPolicyForest`), `econml/policy/_base.py`, and the [[cython-forest-stack|Cython]] policy forest in `econml/policy/_forest/` (its own `_criterion.pyx`). Fit with `policy.fit(Y, T, X=X, W=W)`; then `predict(X)` for recommended treatment, `feature_importances_`, and `plot()` (needs the `[plt]` extra). Key params: `max_depth`, `min_impurity_decrease`, `honest`.
