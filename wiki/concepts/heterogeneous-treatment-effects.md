---
title: Heterogeneous Treatment Effects (CATE)
type: concept
tags: [cate, treatment-effect, core]
related: [potential-outcomes, unconfoundedness, api-vocabulary, choosing-a-method]
source_files:
  - econml/_cate_estimator.py
  - doc/spec/motivation.rst
refs:
  - "Künzel et al. 2017, arXiv:1706.03461"
---

# Heterogeneous Treatment Effects (CATE)

The **Conditional Average Treatment Effect (CATE)** is the central object EconML estimates. It answers: *for a unit with features $X=x$, what is the causal effect of moving treatment from $T_0$ to $T_1$ on outcome $Y$?*

$$\tau(x, T_0, T_1) = \mathbb{E}[Y(T_1) - Y(T_0) \mid X = x]$$

where $Y(t)$ is the [[potential-outcomes|potential outcome]] under treatment $t$. Unlike the **Average Treatment Effect (ATE)** $\mathbb{E}[Y(T_1) - Y(T_0)]$, which is a single number, the CATE is a *function* of $x$ — it captures **effect heterogeneity**, how responsiveness varies across the population.

## Why it matters

Most ML predicts $\mathbb{E}[Y \mid X]$ (what *will* happen). Causal estimation targets $\mathbb{E}[Y(t) \mid X]$ (what *would* happen under intervention $t$). A model that predicts outcomes well can still be useless or misleading for deciding *who to treat*. The CATE is exactly what you need for personalized decisions: pricing, targeting, treatment assignment, policy. See `doc/spec/motivation.rst` for worked use cases (A/B testing, pricing, clinical, recommendations).

## Constant marginal CATE

When the effect is linear in the treatment, EconML works with the **constant marginal CATE** $\theta(X)$, the per-unit-of-treatment effect:

$$Y = \theta(X) \cdot T + g(X, W) + \epsilon$$

Then $\tau(x, T_0, T_1) = \theta(x) \cdot (T_1 - T_0)$. This is the quantity returned by `const_marginal_effect`, while `effect` returns $\tau$ for a specified $T_0 \to T_1$ contrast. See [[api-vocabulary]] for the full method surface.

## Identification

The CATE is a causal quantity, so estimating it from observational data requires assumptions. Most EconML estimators assume [[unconfoundedness]] (all common causes of $T$ and $Y$ are observed in $X, W$); when that fails, [[instrumental-variables]] methods can recover it with a valid instrument $Z$. Without one of these, no estimator — however flexible — yields a causal effect.

## How EconML approaches it

EconML treats CATE estimation as a *two-stage* problem: use flexible ML to remove the influence of confounders (the [[nuisance-and-final-models|nuisance]] stage), then fit a final model for $\theta(X)$ on the de-confounded signal. The families differ in *how* they do this — see [[double-machine-learning]], [[doubly-robust-learner]], [[metalearners]], [[orthogonal-random-forest]], [[generalized-random-forest]] — but share the common API in [[api-vocabulary]] and (for most) the [[ortho-learner-engine]].
