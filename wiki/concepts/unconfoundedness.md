---
title: Unconfoundedness
type: concept
tags: [unconfoundedness, identification, confounding, assumptions]
related: [potential-outcomes, instrumental-variables, validation-and-sensitivity, propensity-score]
source_files:
  - doc/spec/estimation/dml.rst
  - doc/spec/validation.rst
refs:
  - "Chernozhukov et al. 2022 (omitted variable bias), NBER 30302"
---

# Unconfoundedness

**Unconfoundedness** (a.k.a. conditional ignorability, selection-on-observables, no unobserved confounders) is the core identifying assumption behind most EconML estimators. It states that, conditional on the observed controls $X, W$, treatment assignment is as good as random:

$$\{Y^{(t)}\}_t \perp T \mid X, W$$

Concretely: there is **no unobserved variable** that simultaneously affects both the treatment $T$ and the outcome $Y$ that isn't captured in $X, W$. A **confounder** is exactly such a common cause; leaving one out biases the estimated effect.

## Why it is required

Observational treatment is *not* randomized — units that received a treatment may differ systematically from those that didn't. Unconfoundedness says all those systematic differences are observed (in $X, W$), so we can adjust for them. Under it, the [[potential-outcomes|counterfactuals]] become recoverable and the [[heterogeneous-treatment-effects|CATE]] is identified.

This is the assumption that lets [[double-machine-learning]], [[doubly-robust-learner]], [[metalearners]], and the forest estimators interpret their output causally. They control for $W$ (and $X$) but **cannot** account for confounders outside the data.

## When it fails

If an important confounder is unobserved, the estimate is biased and no amount of ML flexibility fixes it. Two responses in EconML:

- **Get an instrument.** If you observe a variable $Z$ that affects $T$ but only affects $Y$ through $T$, the [[instrumental-variables]] methods identify the effect *without* unconfoundedness.
- **Quantify the damage.** You can't prove the absence of unobserved confounders, but you can ask how strong one would have to be to overturn your conclusion. EconML's sensitivity analysis (`sensitivity_analysis`, `robustness_value`, `sensitivity_summary` on [[double-machine-learning|DML]]/[[doubly-robust-learner|DRLearner]] subclasses, based on Chernozhukov et al. 2022) does exactly this — see [[validation-and-sensitivity]].

## Controls vs. heterogeneity features

In the EconML API, $W$ are pure controls (confounders to adjust for) and $X$ are the features along which the effect may vary. A variable can be both — pass it in both $X$ and $W$. Unconfoundedness is a statement about $X \cup W$ jointly capturing all confounding. See [[api-vocabulary]].
