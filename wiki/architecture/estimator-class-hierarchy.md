---
title: Estimator Class Hierarchy
type: architecture
tags: [architecture, class-hierarchy, mixins, sklearn]
related: [ortho-learner-engine, api-vocabulary, inference-and-confidence-intervals]
source_files:
  - econml/_cate_estimator.py
  - econml/_ortho_learner.py
---

# Estimator Class Hierarchy

Almost every EconML estimator is assembled from a small set of base classes and **mixins** in `econml/_cate_estimator.py`, rather than implementing `fit`/`effect` from scratch. Understanding the layering explains how features (inference, treatment expansion, summaries) appear "for free" across the library.

## The spine

- **`BaseCateEstimator`** — the abstract root. Defines the [[api-vocabulary|shared API]]: `fit`, `effect`, `effect_interval`, `effect_inference`, `ate`, and the `inference=` plumbing.
- **`LinearCateEstimator`** — for estimators whose effect is linear in the treatment. Adds `const_marginal_effect(X)` ($\theta(X)$) and its interval/inference variants. This is the parent most concrete estimators ultimately derive from.

## Behavior mixins (composed in, orthogonally)

- **`TreatmentExpansionMixin`** — handles baseline/control treatment, one-hot expansion of discrete treatments, and `treatment_featurizer`; makes `effect` work for arbitrary $T_0\to T_1$ contrasts.
- **`LinearModelFinalCateEstimatorMixin`** and its `*Discrete` sibling — for a linear final model; expose `coef_`, `intercept_`, `summary()`.
- **`StatsModelsCateEstimatorMixin`** / `StatsModelsCateEstimatorDiscreteMixin` — wire in OLS/statsmodels analytic [[inference-and-confidence-intervals|inference]].
- **`DebiasedLassoCateEstimatorMixin`** (+ discrete) — debiased-lasso inference for sparse final models.
- **`ForestModelFinalCateEstimatorMixin`** (+ discrete) — Bootstrap-of-Little-Bags inference for forest final models.

A concrete class like `LinearDML` is then roughly *"`_OrthoLearner` (engine) + a linear final model + `StatsModelsCateEstimatorMixin` (inference)"*. Swapping the final-model mixin is what produces the `Linear` / `SparseLinear` / `Forest` / `NonParam` variants within each family.

## The engine layer

Most estimators sit on top of **`_OrthoLearner`** (`econml/_ortho_learner.py`), which itself is `TreatmentExpansionMixin, LinearCateEstimator`. It implements the generic [[cross-fitting]] + orthogonal-plugin loop so subclasses only specify nuisances and the final model — see [[ortho-learner-engine]]. The exceptions are the [[metalearners]] (which combine ML models directly, not via the orthogonal loop) and the [[orthogonal-random-forest|OrthoForest]] estimators (which add a local-nuisance layer, `BaseOrthoForest`).

## sklearn compatibility

Estimators follow sklearn conventions (`fit`/`predict`-style, cloneable, `get_params`). First-stage models are any sklearn-compatible estimators; the forest predictors in [[cython-forest-stack|`econml/grf`]] are themselves sklearn-style. This is why `GridSearchCV`, `Pipeline`, and cross-validated models slot directly into the nuisance slots (see [[model-selection-and-scoring]]).
