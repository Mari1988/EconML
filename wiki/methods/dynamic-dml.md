---
title: Dynamic DML (panel / sequential treatments)
type: method
tags: [dynamic-dml, panel, time-series, sequential-treatment, method-family]
related: [double-machine-learning, orthogonal-ml, cross-fitting, api-vocabulary]
source_files:
  - econml/panel/dml/_dml.py
  - doc/spec/estimation/dynamic_dml.rst
refs:
  - "Lewis & Syrgkanis 2021, arXiv:2002.07285"
---

# Dynamic DML

**Import:** `from econml.panel.dml import DynamicDML`

## What it is

`DynamicDML` extends [[double-machine-learning|DML]] to settings where treatments are assigned **sequentially over time** via an adaptive policy, and earlier treatments can affect later outcomes (and later treatment decisions). It estimates the effect of the treatment in each period on the final outcome, adjusting for dynamic confounding (Lewis & Syrgkanis 2021).

## When to use it

Panel/longitudinal data with a per-unit sequence of (state, treatment, outcome) over $m$ periods, where you want the dynamic treatment effects and all dynamic confounders are observed. Data is passed with a `groups=` argument identifying which rows belong to the same unit.

## Formal methodology

The data is a Markov decision process $\{X_t, W_t, T_t, Y_t\}_{t=1}^m$ with structural assumptions

$$XW_t = A\,T_{t-1} + B\,XW_{t-1} + \eta_t,\quad T_t = p(T_{t-1}, XW_t, \zeta_t),\quad Y_t = \theta_0(X_0)'T_t + \mu' XW_t + \epsilon_t$$

(where $XW$ concatenates $X$ and $W$). It applies the orthogonal DML machinery with period-aware nuisance estimation and **group-aware [[cross-fitting]]** (folds split on whole units, not rows) to learn the effect of treatments across periods on the last-period outcome. (`doc/spec/estimation/dynamic_dml.rst`)

## How it maps to code

`econml/panel/dml/_dml.py::DynamicDML` on the [[ortho-learner-engine|_OrthoLearner]]. Call `est.fit(Y, T, X=X, W=W, groups=groups)`; `model_y`, `model_t`, and `cv` behave as in standard [[double-machine-learning|DML]]. The FAQ in `doc/spec/estimation/dml.rst` applies here too. See [[api-vocabulary]] for the shared method surface.
