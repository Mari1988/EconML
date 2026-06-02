---
title: Model Selection & Scoring
type: guide
tags: [guide, model-selection, rscorer, ensemble, automl]
related: [nuisance-and-final-models, double-machine-learning, doubly-robust-learner, ortho-learner-engine, validation-and-sensitivity]
source_files:
  - econml/sklearn_extensions/model_selection.py
  - econml/score/rscorer.py
  - econml/score/ensemble_cate.py
  - doc/spec/model_selection.rst
refs:
  - "Nie & Wager 2017, arXiv:1712.04912"
---

# Model Selection & Scoring

Two distinct selection problems arise: choosing the **first-stage [[nuisance-and-final-models|nuisance]] models** inside one estimator, and choosing **which CATE estimator** to use among several.

## First-stage model selection (inside an estimator)

Any [[ortho-learner-engine|_OrthoLearner]]-based estimator accepts flexible specs for its nuisance models (`doc/spec/model_selection.rst`):

- **An sklearn estimator** — if it self-tunes via CV (e.g. `LassoCV`), tuning happens once and the chosen hyperparameters are reused across [[cross-fitting|folds]]. Custom classes need `fit` + `predict`/`predict_proba`.
- **A keyword string** — `"linear"`, `"poly"`, `"forest"`, `"gbf"`, `"nnet"`, `"automl"` (all of the above; slow), or `"auto"` (a sensible smaller default subset).
- **A list** of any of the above — the library picks the best.
- A `ModelSelector` (internal two-stage select-then-fit interface).

Tip from the docs: selecting first-stage models *outside* EconML (one global `GridSearchCV`) and passing in `best_estimator_` can be faster and statistically more stable than re-selecting inside every fold. Implemented in `econml/sklearn_extensions/model_selection.py`.

## Choosing among CATE estimators — the RScorer

`RScorer` (`econml/score/rscorer.py`) scores fitted CATE models on a validation set using the **R-loss** (Nie & Wager 2017): it residualizes $Y$ and $T$ on $X,W$ and measures how much better a model's $\theta(X)$ explains the residual product than a baseline — a proxy for CATE accuracy when ground-truth effects are unobservable.

```python
scorer = RScorer(model_y=reg(), model_t=clf(), discrete_treatment=True, cv=3)
scorer.fit(Y_val, T_val, X=X_val)
rscore = [scorer.score(m) for m in models]
best, _ = scorer.best_model(models)
ens, _  = scorer.ensemble(models)   # score-weighted ensemble
```

`ensemble()` builds a score-weighted `EnsembleCateEstimator` (`econml/score/ensemble_cate.py`).

## Per-estimator goodness of fit

Every fitted estimator exposes `score_` (in-sample final-stage loss; lower is better) and `score(Y, T, X, W)` (out-of-sample). Inspect first-stage fit through `models_y`/`models_t`/`models_regression` (e.g. `oob_score_` if the nuisance is an OOB random forest). For credibility checks beyond fit, see [[validation-and-sensitivity]].
