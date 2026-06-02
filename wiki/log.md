# Log

Append-only timeline of wiki activity. Each entry: `## [YYYY-MM-DD] ingest|query|lint | short title`. Recent history: `grep "^## \[" wiki/log.md | tail -5`.

## [2026-06-01] ingest | Bootstrap of the concept wiki

Initial full bootstrap of the EconML concept wiki (32 pages). Synthesized from `doc/spec/*.rst` (estimation guides for DML, DR, forests, metalearners, orthoIV, dynamic; inference, validation, interpretability, model_selection, comparison, references) and the core docstrings in `econml/_ortho_learner.py`, `econml/_cate_estimator.py`, and the method modules.

Created:
- Schema: `wiki/CLAUDE.md`.
- 12 concept pages (CATE, potential outcomes, unconfoundedness, orthogonal-ml, cross-fitting, nuisance/final, residualization, doubly-robust, propensity, IV, honest-forests, inference).
- 8 method pages (DML, DRLearner, GRF/CausalForest, ORF, metalearners, IV methods, dynamic DML, policy learning).
- 4 architecture pages (api-vocabulary, class-hierarchy, ortho-learner-engine, cython-forest-stack).
- 4 guide pages (choosing-a-method, interpretability, model-selection-and-scoring, validation-and-sensitivity).
- `overview.md`, `glossary.md`, `index.md`.
