---
title: Interpretability (trees, policies, SHAP)
type: guide
tags: [guide, interpretability, shap, tree-interpreter, policy]
related: [heterogeneous-treatment-effects, policy-learning, generalized-random-forest]
source_files:
  - econml/cate_interpreter/_interpreters.py
  - econml/_shap.py
  - doc/spec/interpretability.rst
refs:
  - "Lundberg & Lee 2017 (SHAP), arXiv:1705.07874"
---

# Interpretability

Once you've fit a (possibly black-box) [[heterogeneous-treatment-effects|CATE]] model, EconML offers three ways to understand *what drives the heterogeneity* (`doc/spec/interpretability.rst`). All require the `[plt]` extra for plotting.

## Tree interpreter — *who responds differently?*

`SingleTreeCateInterpreter` fits a single shallow decision tree to the learned effect $\theta(X)$, splitting on the cutoffs that most separate treatment-effect levels. Each leaf is a subgroup with a distinct response. Use it for a presentation-ready summary of the key heterogeneity drivers.

```python
from econml.cate_interpreter import SingleTreeCateInterpreter
intrp = SingleTreeCateInterpreter(include_model_uncertainty=True, max_depth=2, min_samples_leaf=10)
intrp.interpret(est, X)
intrp.plot(feature_names=[...])
```

## Policy interpreter — *whom should we treat?*

`SingleTreePolicyInterpreter` instead splits samples into **treat / don't-treat** groups (optionally net of `sample_treatment_costs`), producing an interpretable assignment policy with a recommended treatment at each leaf. This is the *post-hoc* counterpart to direct [[policy-learning]] (which learns the policy from data without first fitting a CATE).

## SHAP — *which features explain a given effect?*

Every CATE estimator has `shap_values(X)`, returning SHAP explanations of the effect model for each (outcome, treatment) pair (Lundberg & Lee 2017). Invaluable for black-box final models like [[generalized-random-forest|causal forests]] where you can't just read coefficients. EconML dispatches to SHAP's fast model-specific algorithms where possible.

```python
shap_values = est.shap_values(X)
shap.summary_plot(shap_values["Y0"]["T0"])
```

## How it maps to code

`econml/cate_interpreter/_interpreters.py` (tree & policy interpreters), `econml/_shap.py` (SHAP integration). For linear final models, you can also just inspect `est.coef_` / `est.summary()` (see [[inference-and-confidence-intervals]]).
