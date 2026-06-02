---
title: EconML Concept Wiki — Overview
type: guide
tags: [overview, landing, map]
related: [heterogeneous-treatment-effects, orthogonal-ml, choosing-a-method, api-vocabulary]
source_files:
  - README.md
  - doc/spec/overview.rst
  - doc/spec/motivation.rst
---

# EconML Concept Wiki — Overview

A knowledge base of the **causal-inference / ML concepts** implemented in EconML, synthesized from the codebase, the `doc/spec` user guide, and the underlying papers. Start here, then follow the wiki-links between pages. See [[index]] for the full catalog and `CLAUDE.md` for how the wiki is maintained.

## What EconML does

EconML estimates **heterogeneous treatment effects** — the causal effect of a treatment $T$ on an outcome $Y$, and how it varies with features $X$ — from observational or experimental data, using machine learning while preserving causal interpretation and (often) valid confidence intervals. The central object is the [[heterogeneous-treatment-effects|CATE]] $\theta(X)$.

## The big idea

Everything rests on two foundations:

1. **[[orthogonal-ml|Orthogonal / double ML]]** — frame effect estimation as an orthogonal moment so that biased ML [[nuisance-and-final-models|nuisance models]] don't contaminate the effect estimate.
2. **[[cross-fitting]]** — fit nuisances out-of-fold so the stages are independent.

Together they let arbitrary ML power the first stage while the [[heterogeneous-treatment-effects|CATE]] stays $\sqrt n$-consistent and asymptotically normal. The [[ortho-learner-engine|_OrthoLearner]] engine implements this once; every method family is a thin specialization.

## Map of the wiki

**Foundational concepts** — [[potential-outcomes]] · [[heterogeneous-treatment-effects]] · [[unconfoundedness]] · [[instrumental-variables]] · [[orthogonal-ml]] · [[cross-fitting]] · [[nuisance-and-final-models]] · [[residualization]] · [[doubly-robust-estimation]] · [[propensity-score]] · [[honest-forests]] · [[inference-and-confidence-intervals]]

**Method families** — [[double-machine-learning]] · [[doubly-robust-learner]] · [[generalized-random-forest]] · [[orthogonal-random-forest]] · [[metalearners]] · [[iv-methods]] · [[dynamic-dml]] · [[policy-learning]]

**Architecture (concept → code)** — [[api-vocabulary]] · [[estimator-class-hierarchy]] · [[ortho-learner-engine]] · [[cython-forest-stack]]

**Task guides** — [[choosing-a-method]] · [[interpretability]] · [[model-selection-and-scoring]] · [[validation-and-sensitivity]]

**Reference** — [[glossary]]

## How to use it

- *Picking an estimator?* → [[choosing-a-method]].
- *Learning the API?* → [[api-vocabulary]].
- *Understanding the theory?* → start at [[orthogonal-ml]] and follow links.
- *A term you don't know?* → [[glossary]].
