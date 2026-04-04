---
title: The Big Picture
weight: 1
bookToc: true
draft: true
---

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

## Why MCMC Makes This Tractable

<!-- TODO: The integral above is usually impossible to compute analytically.
     But we already have samples {θ_s} from p(θ | y) (the MCMC draws).
     For each draw s:
       1. Compute the linear predictor: η_s = B0[site,stratum] + X %*% Beta[s,]
       2. Apply the inverse link: μ_s = inv_link(η_s)
       3. Sample one observation: y_s ~ likelihood(μ_s, σ_s)
     The collection {y_s} is a sample from the posterior predictive.

     Map to code: forecast-engine.R:compute_forecast_draws()
-->

## What the Output Represents

<!-- TODO: Explain what the posterior predictive draws mean in practice:
     - mu_median: the median predicted mean (uncertainty in the mean response)
     - y_rep_median: the median predicted observation (includes obs-level noise)
     - The trajectory plots show these over time; the ribbon is the credible interval
     - A wider ribbon = more uncertainty (could be parameter uncertainty, natural
       variability, or both)
-->
