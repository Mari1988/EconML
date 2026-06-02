---
title: Residualization (Frisch–Waugh–Lovell)
type: concept
tags: [residualization, partialling-out, dml]
related: [double-machine-learning, orthogonal-ml, nuisance-and-final-models]
source_files:
  - econml/dml/_rlearner.py
  - doc/spec/estimation/dml.rst
refs:
  - "Chernozhukov et al. 2016, arXiv:1608.00060"
  - "Nie & Wager 2017, arXiv:1712.04912"
---

# Residualization (partialling-out)

**Residualization** is the trick at the heart of [[double-machine-learning|DML]] and the R-learner. To estimate the effect of $T$ on $Y$ while controlling for $X, W$, you first *partial out* the influence of the controls from both:

$$\tilde Y = Y - \mathbb{E}[Y\mid X, W], \qquad \tilde T = T - \mathbb{E}[T\mid X, W]$$

Under the DML structural model $Y = \theta(X) T + g(X,W) + \epsilon$, subtracting conditional expectations cancels the confounding term $g(X,W)$ and leaves a clean relationship between the residuals:

$$\tilde Y = \theta(X)\cdot \tilde T + \epsilon$$

So $\theta(X)$ can be recovered by regressing $\tilde Y$ on $\tilde T$ (interacted with features of $X$). This is the classical **Frisch–Waugh–Lovell** partialling-out idea, with the two conditional expectations estimated by arbitrary ML ([[nuisance-and-final-models|nuisance models]]) instead of OLS.

## Why it gives orthogonality

The residual-on-residual least-squares moment is automatically Neyman-[[orthogonal-ml|orthogonal]] to errors in the two conditional-expectation nuisances $q=\mathbb{E}[Y\mid X,W]$ and $f=\mathbb{E}[T\mid X,W]$: a small mistake in either residualization has only a second-order effect on $\hat\theta$. This is precisely why DML can use biased ML first stages and still deliver $\sqrt n$-rate, asymptotically normal effect estimates — provided the residuals are computed out-of-fold ([[cross-fitting]]).

## The treatment residual is the "experiment"

Intuitively, $\tilde T$ is the part of the treatment that is *not* predictable from the controls — the conditionally-exogenous variation. Regressing on $\tilde T$ uses only that variation, mimicking a randomized experiment within strata of $X, W$. This is also why DML degrades under poor overlap: if $T$ is nearly deterministic given $X,W$, $\tilde T \approx 0$ and there is little variation to learn from.

## In code

The residual-on-residual fit is implemented in `econml/dml/_rlearner.py::_RLearner`, the private parent of [[double-machine-learning|DML]]. The [[doubly-robust-estimation|doubly robust]] methods use a different (pseudo-outcome) construction to achieve orthogonality rather than residualization.
