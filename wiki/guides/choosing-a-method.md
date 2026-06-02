---
title: Choosing a Method
type: guide
tags: [guide, comparison, flowchart, model-choice]
related: [double-machine-learning, doubly-robust-learner, metalearners, generalized-random-forest, orthogonal-random-forest, iv-methods, instrumental-variables, inference-and-confidence-intervals]
source_files:
  - doc/spec/comparison.rst
  - doc/spec/flowchart.rst
---

# Choosing a Method

A decision aid distilled from `doc/spec/flowchart.rst` and the comparison matrix in `doc/spec/comparison.rst`.

## First question: can you observe all confounders?

- **No, but you have an instrument $Z$** → [[iv-methods]] (see [[instrumental-variables]]).
- **Yes ([[unconfoundedness]] plausible)** → DML / DR / meta-learner / forest, per below.
- **Not sure** → pick an unconfoundedness method, then stress-test it with [[validation-and-sensitivity|sensitivity analysis]].

## Second question: treatment type?

- **Continuous treatment** → [[double-machine-learning|DML]] family (DR and meta-learners are discrete-only).
- **Discrete/categorical** → any of [[double-machine-learning|DML]], [[doubly-robust-learner|DR]], [[metalearners]].

DML vs DR for discrete treatment: DR's final regression is robust to misspecification (estimates a *projection* of the CATE) but is **higher variance under poor overlap** (small [[propensity-score|propensities]]). DML tolerates poor overlap better. See [[doubly-robust-estimation]].

## Third question: do you need confidence intervals?

- **Yes, few heterogeneity features** → `LinearDML` / `LinearDRLearner` (analytic, statsmodels).
- **Yes, many features (sparse)** → `SparseLinearDML` / `SparseLinearDRLearner` (debiased lasso).
- **Yes, nonparametric / unknown heterogeneity** → `CausalForestDML` / `ForestDRLearner` / [[orthogonal-random-forest|OrthoForest]] (Bootstrap-of-Little-Bags).
- **No, just want low MSE + easy model selection** → [[metalearners]], or `DRLearner`/`NonParamDML` with cross-validated final models.

See [[inference-and-confidence-intervals]] for the backends.

## Fourth question: do you know the shape of heterogeneity?

- **Assume linear in $X$** → `LinearDML`/`LinearDRLearner` (+ `featurizer=PolynomialFeatures(...)` for known nonlinearity).
- **High-dim features, sparse effect** → `SparseLinear*`.
- **No idea / nonparametric** → forest estimators or [[orthogonal-random-forest|ORF]].

## The comparison matrix (from `doc/spec/comparison.rst`)

| Estimator | Treatment | Needs $Z$ | Analytic CIs | Linear het. | Multi-outcome | Multi-treatment | High-dim $X$ |
|---|---|---|---|---|---|---|---|
| `LinearDML` | Any | – | ✓ | assumed | ✓ | ✓ | – |
| `SparseLinearDML` | Any | – | ✓ | assumed | ✓ | ✓ | ✓ |
| `CausalForestDML` | Any | – | ✓ | flexible | ✓ | ✓ | ✓ |
| `DML` | Any | – | – | assumed | ✓ | ✓ | ✓ |
| `NonParamDML` | 1-d/Binary | – | – | flexible | ✓ | – | ✓ |
| `LinearDRLearner` | Categorical | – | ✓ | projected | – | ✓ | – |
| `SparseLinearDRLearner` | Categorical | – | ✓ | projected | – | ✓ | ✓ |
| `ForestDRLearner` | Categorical | – | ✓ | flexible | – | ✓ | ✓ |
| `DRLearner` | Categorical | – | – | flexible | – | ✓ | ✓ |
| `DMLOrthoForest` | Any | – | ✓ | flexible | – | ✓ | ✓ |
| `DROrthoForest` | Categorical | – | ✓ | flexible | – | ✓ | ✓ |
| metalearners | Categorical | – | – | flexible | ✓ | ✓ | ✓ |
| `OrthoIV` / `DMLIV` | Any | ✓ | ✓ / – | assumed | ✓ | ✓ | – / ✓ |
| `DRIV` / `LinearDRIV` / `ForestDRIV` | 1-d/Binary | ✓ | ✓ | flexible / projected | – | – | ✓ |
| `(Linear)IntentToTreatDRIV` | Binary | ✓ | ✓ | – / projected | – | – | ✓ |
| `SieveTSLS` | Any | ✓ | – | assumed | ✓ | ✓ | – |

*"projected"* = fits a flexible CATE then projects onto a linear model (a best linear approximation); *"assumed"* = imposes linearity; *"flexible"* = fully nonparametric.

Once chosen, validate the fit with [[validation-and-sensitivity]] and select first-stage models per [[model-selection-and-scoring]].
