---
title: Instrumental Variables
type: concept
tags: [instrumental-variables, iv, endogeneity, unobserved-confounding]
related: [unconfoundedness, iv-methods, orthogonal-ml, doubly-robust-estimation]
source_files:
  - econml/iv/dml/_dml.py
  - doc/spec/estimation/orthoiv.rst
refs:
  - "Syrgkanis et al. 2019, arXiv:1905.10176"
  - "Newey & Powell 2003, Econometrica 71(5)"
  - "Hartford et al. 2017 (Deep IV)"
---

# Instrumental Variables

An **instrumental variable (IV)** $Z$ lets you estimate a causal effect **even when [[unconfoundedness]] fails** — i.e. when there are unobserved confounders of $T$ and $Y$. This is the escape hatch when controlling for observed $X, W$ is not enough.

## What makes a valid instrument

$Z$ must satisfy:

1. **Relevance** — $Z$ affects the treatment $T$ (it shifts who gets treated).
2. **Exclusion** — $Z$ affects the outcome $Y$ *only through* $T$, with no direct path.
3. **Independence** — $Z$ is as-good-as-random (not driven by the unobserved confounders).

The canonical source is an actual randomization: an earlier experiment, a lottery, or an **intent-to-treat** encouragement. In A/B tests where you randomize *who gets a recommendation* to take an action but can't force the action, the recommendation is the instrument and the action is the treatment — see [[iv-methods]].

## The intuition

A naive regression of $Y$ on $T$ is biased because part of $T$'s variation is correlated with the unobserved confounder. IV uses **only the variation in $T$ that is driven by $Z$** — which, by independence + exclusion, is clean of the confounder — to identify the effect. Classical two-stage least squares (`SieveTSLS`) does this by predicting $T$ from $Z$ then regressing $Y$ on the prediction.

## EconML's orthogonal IV

EconML's modern IV estimators ([[iv-methods]]) recast IV effect estimation as minimizing a loss that depends on several auxiliary [[nuisance-and-final-models|nuisance]] regressions (of $Y$, $T$, and $Z$ on $X,W$), built so the loss satisfies a Neyman [[orthogonal-ml|orthogonality]] condition. This lets arbitrary ML fit the nuisances while the effect remains robust and (for parametric final models) asymptotically normal — the same recipe as [[double-machine-learning|DML]]/[[doubly-robust-learner|DR]], extended to the IV moment (Syrgkanis et al. 2019). The DRIV variants additionally use a [[doubly-robust-estimation|doubly robust]] IV moment.

## Cost

IV identifies a more limited quantity (often a local effect for "compliers") and is statistically less efficient — you're using only the instrument-driven slice of treatment variation. Use it when you genuinely cannot observe all confounders; otherwise the [[unconfoundedness]]-based methods are more powerful.
