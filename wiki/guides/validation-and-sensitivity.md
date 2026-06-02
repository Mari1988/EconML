---
title: Validation & Sensitivity Analysis
type: guide
tags: [guide, validation, sensitivity, drtester, robustness, dowhy]
related: [unconfoundedness, inference-and-confidence-intervals, double-machine-learning, doubly-robust-learner, model-selection-and-scoring]
source_files:
  - econml/validate/drtester.py
  - econml/validate/sensitivity_analysis.py
  - doc/spec/validation.rst
refs:
  - "Chernozhukov et al. 2022 (OVB), NBER 30302"
  - "Lundberg & Lee 2017"
---

# Validation & Sensitivity Analysis

Causal estimates can't be validated against ground truth (the counterfactual is unobservable), so EconML offers *credibility checks* rather than accuracy tests (`doc/spec/validation.rst`).

## Sensitivity to unobserved confounding

The biggest threat is a violated [[unconfoundedness]] assumption. You can't prove no unobserved confounder exists, but you can ask **how strong one would have to be** to change your conclusion. Subclasses of [[double-machine-learning|DML]] and [[doubly-robust-learner|DRLearner]] expose (based on Chernozhukov et al. 2022):

- **`sensitivity_analysis`** — confidence interval for the ATE under a hypothesized level of unobserved confounding.
- **`robustness_value`** — the minimum confounding strength that would move the ATE interval to include a null value (0 by default). Higher = more robust.
- **`sensitivity_summary`** — a combined report.

Implemented in `econml/validate/sensitivity_analysis.py`.

## DRTester — did the model capture real heterogeneity?

`DRTester` (`econml/validate/drtester.py`) validates a fitted CATE model on held-out data using doubly-robust scores:

- **Best Linear Predictor (BLP)** — tests whether the model's $\theta(X)$ has genuine predictive slope for the true effect.
- **Calibration R²** — whether predicted effect magnitudes match realized ones across groups.
- **Uplift curves (AUTOC / QINI)** — whether targeting by the model beats random assignment.

A passing BLP/calibration gives confidence the heterogeneity is real, not overfit noise.

## Inference as a surface check

Inspecting p-values / [[inference-and-confidence-intervals|confidence intervals]] is a first-pass check — but **only valid under correct specification** (a linear model on a nonlinear truth gives misleadingly tight intervals). Treat significant p-values as necessary, not sufficient.

## DoWhy refutation tests

The companion DoWhy library (via `econml[dowhy]`, `econml/dowhy.py`) adds refutation tests — re-estimate under data perturbations (placebo treatment, random common cause, subset removal) and check the estimate is stable.

## Recommended workflow

1. Pick a method ([[choosing-a-method]]) and select first-stage models ([[model-selection-and-scoring]]).
2. Check inference + `score_`.
3. Run `DRTester` on a validation split to confirm captured heterogeneity.
4. Run sensitivity analysis to bound exposure to unobserved confounding.
5. Optionally cross-check with DoWhy refutations.
