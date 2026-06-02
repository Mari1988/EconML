---
title: Glossary
type: guide
tags: [glossary, reference, terminology]
related: [overview, api-vocabulary]
source_files:
  - doc/spec/comparison.rst
  - doc/spec/references.rst
---

# Glossary

Quick definitions; each links to its full page where one exists.

- **ATE** — Average Treatment Effect, $\mathbb{E}[Y(T_1)-Y(T_0)]$; the population-average of the [[heterogeneous-treatment-effects|CATE]].
- **BLB (Bootstrap-of-Little-Bags)** — variance estimator for [[honest-forests|honest subsampled forests]]; the `inference='blb'` backend. See [[inference-and-confidence-intervals]].
- **BLP (Best Linear Predictor)** — validation test for whether a model captures real heterogeneity. See [[validation-and-sensitivity]].
- **CATE** — Conditional Average Treatment Effect, $\theta(X)$ / $\tau(X)$. The central estimand → [[heterogeneous-treatment-effects]].
- **Confounder** — a common cause of treatment and outcome; must be observed for [[unconfoundedness]] to hold.
- **`const_marginal_effect`** — the per-unit-treatment effect $\theta(X)$; see [[api-vocabulary]].
- **Cross-fitting** — out-of-fold nuisance estimation → [[cross-fitting]].
- **Debiased lasso** — inference method for sparse high-dim linear final models → [[inference-and-confidence-intervals]].
- **DML / Double ML / R-learner** — [[double-machine-learning]]; principle in [[orthogonal-ml]].
- **Doubly robust (DR / AIPW)** — combine outcome + propensity models → [[doubly-robust-estimation]].
- **`effect` / `marginal_effect`** — CATE method surface → [[api-vocabulary]].
- **GRF** — Generalized Random Forest → [[generalized-random-forest]].
- **Heterogeneity features ($X$)** vs **controls ($W$)** — see [[api-vocabulary]], [[unconfoundedness]].
- **Honesty** — split-set / value-set separation in forests → [[honest-forests]].
- **Instrument ($Z$)** — variable affecting $Y$ only through $T$ → [[instrumental-variables]].
- **Intent-to-treat** — encouragement design where the randomized recommendation is the instrument → [[iv-methods]].
- **Meta-learner** — black-box CATE estimator (S/T/X/DA) → [[metalearners]].
- **Neyman orthogonality** — moment insensitive (first-order) to nuisance error → [[orthogonal-ml]].
- **Nuisance model** — first-stage predictive model → [[nuisance-and-final-models]].
- **Overlap / positivity** — every treatment has non-trivial probability across $X,W$ → [[propensity-score]].
- **Potential outcome ($Y^{(t)}$)** — outcome under treatment $t$ → [[potential-outcomes]].
- **Projection (of the CATE)** — best approximation of the true CATE within the final model class; what DR linear models estimate under misspecification → [[doubly-robust-estimation]], [[choosing-a-method]].
- **Propensity score** — $\Pr[T=t\mid X,W]$ → [[propensity-score]].
- **R-loss / RScorer** — residual-based CATE model-selection score → [[model-selection-and-scoring]].
- **Residualization / partialling-out** — subtract conditional expectations → [[residualization]].
- **Robustness value** — minimum unobserved-confounding strength to overturn a result → [[validation-and-sensitivity]].
- **SHAP** — feature attribution for the effect model → [[interpretability]].
- **Unconfoundedness / ignorability** — no unobserved confounders → [[unconfoundedness]].
- **Y, T, X, W, Z** — outcome, treatment, heterogeneity features, controls, instrument → [[api-vocabulary]].
