---
title: API Vocabulary (Y, T, X, W, Z and the effect methods)
type: architecture
tags: [api, vocabulary, effect, const-marginal-effect]
related: [estimator-class-hierarchy, heterogeneous-treatment-effects, inference-and-confidence-intervals, unconfoundedness]
source_files:
  - econml/_cate_estimator.py
---

# API Vocabulary

EconML estimators share one API, defined on `econml/_cate_estimator.py::BaseCateEstimator` and `LinearCateEstimator`. Learn it once and it transfers across every method family.

## The input variables

| Symbol | Argument | Meaning |
|---|---|---|
| $Y$ | `Y` | **Outcome(s)** — what you measure; shape `(n,)` or `(n, d_y)` for multiple outcomes. |
| $T$ | `T` | **Treatment(s)** — the intervention; continuous, binary, or categorical. `(n,)` or `(n, d_t)`. |
| $X$ | `X=` | **Effect-modifier features** — variables the [[heterogeneous-treatment-effects|CATE]] $\theta(X)$ may vary along. Optional (no `X` ⇒ estimate an ATE). |
| $W$ | `W=` | **Controls / confounders** — adjusted for but not sources of heterogeneity. |
| $Z$ | `Z=` | **Instrument(s)** — only for [[iv-methods|IV]] estimators. See [[instrumental-variables]]. |

$X$ vs $W$: both are conditioned on for [[unconfoundedness]]; the difference is that $\theta$ is reported as a function of $X$ only. A variable that both confounds *and* drives heterogeneity goes in **both**.

The canonical call: `est.fit(Y, T, X=X, W=W)` (add `Z=Z` for IV, `groups=` for [[dynamic-dml|panel]]). Common constructor flags: `discrete_treatment`, `discrete_outcome`, `discrete_instrument`, `categories` (first category = baseline/control), `cv` ([[cross-fitting|folds]]), `mc_iters`, `featurizer`, `treatment_featurizer`.

## The output methods

| Method | Returns |
|---|---|
| `const_marginal_effect(X)` | $\theta(X)$, the per-unit-treatment effect (for treatment-linear models). Shape `(m, d_y, d_t)`. |
| `effect(X, T0, T1)` | $\tau = \theta(X)\cdot(T_1-T_0)$, effect of moving $T_0\to T_1$. Defaults $T_0=0, T_1=1$. |
| `marginal_effect(T, X)` | $\partial\tau/\partial T$ at treatment level $T$ (matters when the treatment is featurized nonlinearly). |
| `ate(X, T0, T1)` / `marginal_ate(...)` | population averages of the above. |
| `*_interval(..., alpha=)` | lower/upper confidence bounds for any of the above. |
| `*_inference(...)` | rich `InferenceResults` (std err, z, p, summaries). |
| `shap_values(X)` | SHAP explanation of the effect model (see [[interpretability]]). |
| `score(Y, T, X, W)` | out-of-sample final-stage loss (lower is better). |

The `*_interval` / `*_inference` variants are gated by the `inference=` argument at `fit` time — see [[inference-and-confidence-intervals]].

## Treatment featurization vs. heterogeneity featurization

- `featurizer=` transforms $X$ to build a richer model of *how the effect varies* (e.g. `PolynomialFeatures` ⇒ polynomial $\theta(X)$). Coefficients are then on $T \otimes \phi(X)$.
- `treatment_featurizer=` transforms $T$ to model a *nonlinear dose response*, while keeping `marginal_effect` expressed in the original treatment units.

These are handled by `TreatmentExpansionMixin` in the [[estimator-class-hierarchy]].
