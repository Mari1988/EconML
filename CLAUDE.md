# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

EconML is a Python package for estimating **heterogeneous treatment effects** (CATE — Conditional Average Treatment Effects) from observational data via machine learning. It measures the causal effect of treatment(s) `T` on outcome `Y`, controlling for features/confounders `X, W` (and optionally instruments `Z`), and how that effect varies with `X`. It is part of the [PyWhy](https://www.pywhy.org/) ecosystem.

## Build & install (editable, with C extensions)

The package contains Cython extensions (see below), so an editable install compiles native code:

```bash
pip install -e .            # base install
pip install -e .[plt]       # + matplotlib/graphviz for plotting & interpreters
pip install -e .[all]       # + dowhy, ray, azure-cli, plotting
```

After editing any `.pyx`/`.pxd` file you must rebuild (`pip install -e .` again) for changes to take effect. The build is driven by `setup.py` (not declarative): it cythonizes any `.pyx` that lacks a committed `.c`, and **treats Cython warnings as errors** (`Options.warning_errors = True`). Most metadata lives in `pyproject.toml`; `_version.py` is the single source of the version number.

## Testing

CI uses `pytest`, but running the full suite locally is extremely slow and some tests need extra environment/deps. Prefer running a targeted subset via `unittest`:

```bash
python -m unittest econml.tests.test_dml                      # one module
python -m unittest econml.tests.test_dml.TestDML              # one class
python -m unittest econml.tests.test_dml.TestDML.test_cate_api  # one method
```

Full run (slow, needs `pytest pytest-xdist pytest-cov coverage[toml]`, plus notebook deps): `python -m pytest`. Pytest config in `pyproject.toml` runs with `-n auto` (xdist) and `--import-mode=importlib`, with custom markers: `slow`, `notebook`, `automl`, `dml`, `serial`, `cate_api`, `treatment_featurization`, `ray`.

Tests live in `econml/tests/`; shared synthetic data generators are in `econml/tests/dgp.py` and `econml/data/dgps.py`.

## Linting

Ruff enforces style (config in `pyproject.toml`): line length 120, numpy docstring convention, rule sets `D/W/E/F/SIM`. Pre-commit runs ruff with `--fix`:

```bash
pre-commit install              # one-time
pre-commit run --all-files      # manual run
ruff check econml/              # direct
```

`prototypes/` and `monte_carlo_tests/` are excluded from linting.

## Architecture

### The estimator class hierarchy

Nearly every estimator inherits a common API defined in `econml/_cate_estimator.py`:

- `BaseCateEstimator` — abstract base: `fit(Y, T, *, X, W, Z, ...)`, `effect(X, T0, T1)`, `effect_interval(...)`, `effect_inference(...)`.
- `LinearCateEstimator` — adds `const_marginal_effect(X)` and its interval/inference variants, for estimators whose effect is linear in treatment.
- Mixins layer in behavior orthogonally: `TreatmentExpansionMixin` (treatment featurization/baseline handling), `LinearModelFinalCateEstimatorMixin`, `StatsModelsCateEstimatorMixin`, `ForestModelFinalCateEstimatorMixin`, and `*DiscreteMixin` variants for discrete treatments. New estimators are typically assembled by combining these mixins rather than overriding `fit`/`effect` wholesale.

### `_OrthoLearner` — the central orchestration

`econml/_ortho_learner.py` (`_OrthoLearner`, extends `TreatmentExpansionMixin, LinearCateEstimator`) implements the generic **double/orthogonal ML** loop shared by most estimators: cross-fitting of first-stage nuisance models (predicting `Y` and `T` from `X, W`), then fitting a "final" model on the orthogonalized residuals. Subclasses (DML, DR, IV variants) just specify which nuisance and final models to use and how to compute residuals — the cross-fitting, CV splitting, sample weighting, and inference plumbing are inherited. Understanding this file is the key to understanding the bulk of the library.

### Method families (one subpackage each, exported from its `__init__.py`)

- `econml/dml/` — Double ML / R-learner: `LinearDML`, `SparseLinearDML`, `NonParamDML`, `DML`, `CausalForestDML`.
- `econml/dr/` — Doubly Robust learners: `DRLearner`, `LinearDRLearner`, `SparseLinearDRLearner`, `ForestDRLearner`.
- `econml/iv/` — Instrumental variables, split into `iv/dml`, `iv/dr`, `iv/sieve` (DeepIV etc.).
- `econml/metalearners/` — `SLearner`, `TLearner`, `XLearner`, `DomainAdaptationLearner`.
- `econml/orf/` — Orthogonal Random Forests (`DMLOrthoForest`, `DROrthoForest`).
- `econml/panel/dml/` — `DynamicDML` for panel/dynamic treatments.
- `econml/policy/` — offline policy learning (`DRPolicyTree`, `DRPolicyForest`).

### Cython forest infrastructure

Three layers of compiled tree code (forked/adapted from scikit-learn) underpin the forest estimators:

- `econml/tree/` — low-level decision tree (`_tree.pyx`, `_splitter.pyx`, `_criterion.pyx`, `_utils.pyx`).
- `econml/grf/` — Generalized Random Forests built on `econml/tree` (`_base_grf.py`, `_criterion.pyx`); backs `CausalForestDML`.
- `econml/policy/_forest/` — policy forest with its own `_criterion.pyx`.

`.pxd` files are Cython headers; changes there ripple into dependent `.pyx` modules and require recompilation.

### Cross-cutting modules

- `econml/inference/` — pluggable inference: `BootstrapInference`, analytic/statsmodels-based intervals. Passed via the `inference=` argument to `fit()`.
- `econml/sklearn_extensions/` — `WeightedLasso`/`WeightedLassoCV`, `model_selection.py` (`ModelSelector`, used by cross-fitting estimators to do first-stage model selection from a list of candidate models or sklearn CV objects).
- `econml/cate_interpreter/` — `SingleTreeCateInterpreter`, `SingleTreePolicyInterpreter` (require `[plt]`).
- `econml/score/` — `RScorer` (causal model selection) and CATE ensembling.
- `econml/validate/` — `DRTester`, sensitivity analysis.
- `econml/dowhy.py` — DoWhy interop (requires `[dowhy]`); has a hardcoded dowhy version check kept in sync with `pyproject.toml`.
- `econml/utilities.py` — shared array/shape helpers (`check_input_arrays`, `shape`, `reshape`, etc.) and `MissingModule` (used to lazily defer ImportErrors for optional deps like ray until first use).
- `econml/federated_learning.py` — `FederatedEstimator` for combining estimators across data partitions.

## "Last Known Good" (LKG) dependencies

`lkg.txt` / `lkg-notebook.txt` pin exact, known-passing versions of the full dependency tree (with per-OS / per-Python-version markers). CI installs these for PRs to keep builds reproducible, but uses unpinned latest deps on nightly runs to catch upstream breakage. They are generated by `.github/workflows/generate_lkg.py` — do not hand-edit; regenerate via CI. When adding support for a new dependency version, the LKG files are typically updated in a follow-up commit.

## Concept wiki

`wiki/` holds an LLM-maintained knowledge base of the **causal-inference / ML concepts** in this repo (CATE, orthogonal ML, cross-fitting, the estimator families, etc.) — synthesized from `doc/spec/*.rst`, docstrings, and the cited papers into interlinked, Obsidian-style markdown pages. Use it to answer conceptual questions and to ground explanations. Start at `wiki/overview.md` and `wiki/index.md`. When working under `wiki/`, follow `wiki/CLAUDE.md`, which defines the maintenance conventions (ingest / query / lint workflows, page format, the rule that the code is authoritative over the wiki).

## Conventions

- Sign-off required: commits must be DCO-signed (`git commit -s`).
- Variable naming follows the causal-inference notation throughout: `Y` (outcome), `T` (treatment), `X` (effect-modifier features), `W` (controls/confounders), `Z` (instrument), `nuisance` models (first stage), `final` model (effect stage).
- Optional dependencies (`graphviz`, `matplotlib`, `dowhy`, `ray`) must be imported lazily/guarded so the base package works without them.
