---
title: The Big Picture
weight: 1
---

**\*WORK IN PROGRESS (this is empty for now)***

# The Big Picture: Posterior Predictive Forecasting

<!-- PREREQUISITE: This page assumes familiarity with the M4MD model — in
     particular the hierarchical Bayesian structure described in the original
     paper (link TBD). -->

## What Does the Model Know?

<!-- TODO: Intuition section — after fitting, the model has learned a posterior
     distribution over all parameters (intercepts, slopes, variance terms).
     This distribution encodes everything the data told us about vegetation
     response to climate. Forecasting asks: given new climate conditions,
     what does the model expect to see?
-->

## The Posterior Predictive Distribution

<!-- TODO: Introduce the integral:
     p(y_new | y) = ∫ p(y_new | θ) p(θ | y) dθ

     Explain each piece in plain English:
     - p(θ | y): the posterior — what we learned from the data
     - p(y_new | θ): the likelihood — how the model generates new data
     - p(y_new | y): the posterior predictive — full uncertainty about new observations

     Note that this is a distribution, not a point estimate. The result reflects
     both parameter uncertainty (from MCMC) and observation-level noise.
-->