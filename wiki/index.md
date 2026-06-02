# Index

Catalog of every page in the wiki. Read this first when answering a query, then drill into the relevant pages. Start at [[overview]]; look up terms in [[glossary]].

## Concepts (cross-cutting theory)

- [[heterogeneous-treatment-effects]] — the CATE $\theta(X)$, EconML's central estimand.
- [[potential-outcomes]] — the Neyman–Rubin framework and the fundamental problem of causal inference.
- [[unconfoundedness]] — the core "no unobserved confounders" identifying assumption.
- [[instrumental-variables]] — identifying effects via an instrument $Z$ when unconfoundedness fails.
- [[orthogonal-ml]] — Neyman orthogonality; why biased ML nuisances are OK. The key principle.
- [[cross-fitting]] — out-of-fold nuisance estimation; the second pillar.
- [[nuisance-and-final-models]] — the two-stage (first-stage / final) structure and vocabulary.
- [[residualization]] — partialling-out (Frisch–Waugh–Lovell), the DML trick.
- [[doubly-robust-estimation]] — augmented pseudo-outcomes correct if either model is right.
- [[propensity-score]] — $\Pr[T=t\mid X,W]$, overlap, and trimming.
- [[honest-forests]] — honesty, subsampling, and the adaptive-kernel view of causal forests.
- [[inference-and-confidence-intervals]] — bootstrap / statsmodels / debiased-lasso / BLB backends.

## Methods (estimator families)

- [[double-machine-learning]] — DML / R-learner: residualize then regress. Any treatment.
- [[doubly-robust-learner]] — DRLearner family for discrete treatments.
- [[generalized-random-forest]] — GRF & CausalForestDML; `het` vs `mse` criteria.
- [[orthogonal-random-forest]] — ORF; local nuisance estimation (DML & DR variants).
- [[metalearners]] — S / T / X / Domain-Adaptation learners.
- [[iv-methods]] — OrthoIV, DMLIV, DRIV, intent-to-treat, SieveTSLS/DeepIV.
- [[dynamic-dml]] — DML for sequential/panel treatments.
- [[policy-learning]] — directly learn treatment-assignment rules (DR policy trees/forests).

## Architecture (concept → code)

- [[api-vocabulary]] — Y/T/X/W/Z and the `effect`/`const_marginal_effect`/`marginal_effect`/`ate` methods.
- [[estimator-class-hierarchy]] — BaseCateEstimator → LinearCateEstimator → mixins.
- [[ortho-learner-engine]] — `_OrthoLearner`, the generic cross-fitting + orthogonal-plugin engine.
- [[cython-forest-stack]] — the `tree/` → `grf/` → `policy/_forest/` Cython layers.

## Guides (task-oriented)

- [[choosing-a-method]] — decision flow + the full estimator comparison matrix.
- [[interpretability]] — tree interpreter, policy interpreter, SHAP.
- [[model-selection-and-scoring]] — first-stage selection, RScorer, ensembling.
- [[validation-and-sensitivity]] — DRTester, sensitivity analysis, DoWhy refutation.

## Reference

- [[overview]] — landing page / domain map.
- [[glossary]] — term → definition with links.
