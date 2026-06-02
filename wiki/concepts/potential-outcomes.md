---
title: Potential Outcomes
type: concept
tags: [potential-outcomes, identification, core]
related: [heterogeneous-treatment-effects, unconfoundedness, doubly-robust-estimation]
source_files:
  - doc/spec/estimation/dr.rst
  - doc/spec/causal_intro.rst
refs:
  - "Hernán & Robins 2010, Causal Inference (book)"
---

# Potential Outcomes

The **potential outcomes** (Neyman–Rubin) framework is the language causal inference uses to define effects. For each unit and each possible treatment value $t$, there is a potential outcome $Y^{(t)}$ — the outcome that *would* be observed if the unit received treatment $t$.

The treatment effect for a unit is a contrast of potential outcomes, e.g. $Y^{(1)} - Y^{(0)}$. The [[heterogeneous-treatment-effects|CATE]] is the conditional average of this contrast:

$$\theta_t(X) = \mathbb{E}[Y^{(t)} - Y^{(0)} \mid X]$$

## The fundamental problem of causal inference

For any unit we observe only **one** potential outcome — the one corresponding to the treatment actually received. The others are counterfactual and never seen. So a treatment effect is never directly observed in data; it must be *identified* under assumptions and then estimated.

## What makes the counterfactuals recoverable

EconML's outcome-modeling estimators ([[doubly-robust-learner]], the DR forests) make these assumptions explicit (`doc/spec/estimation/dr.rst`):

$$Y^{(t)} = g_t(X, W) + \epsilon_t, \quad \mathbb{E}[\epsilon \mid X, W] = 0$$
$$\Pr[T = t \mid X, W] = p_t(X, W) \quad \text{(the [[propensity-score]])}$$
$$\{Y^{(t)}\} \perp T \mid X, W \quad \text{([[unconfoundedness]])}$$

The conditional-independence (unconfoundedness) assumption is what lets us treat units with the same $X, W$ but different observed treatments as exchangeable, and thereby fill in the missing counterfactuals — either by modeling $g_t$ directly, by reweighting with the [[propensity-score]], or by combining both ([[doubly-robust-estimation]]).

## Relation to the DML structural form

[[double-machine-learning|DML]] expresses the same content as a structural equation $Y = \theta(X)\cdot T + g(X,W) + \epsilon$ rather than via separate $Y^{(t)}$ surfaces. The two views coincide for the effect parameter; DR-style methods reason in potential outcomes because they construct explicit counterfactual pseudo-outcomes (see [[doubly-robust-estimation]]).
