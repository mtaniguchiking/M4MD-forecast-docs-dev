---
title: Interpreting uncertainty
weight: 5
---

# Interpreting forecast uncertainty

In this pipeline, there are three sources of uncertainty during a forecast. This page explains the three sources and how to interpret them.

## The three sources

|   | Source name | Also called | Shown on plots as |
|---|---------------------|-------------|-------------------|
| 1 | **Parameter uncertainty** | estimation uncertainty | Credible interval |
| 2 | **Observation uncertainty** | process uncertainty, sampling uncertainty | Prediction interval |
| 3 | **Future covariate uncertainty** | model uncertainty, structural uncertainty | Ensemble spread across runs |

Sources 1 and 3 are *epistemic*, meaning they reflect what we don't know. They shrink with more data and/or better models. Source 2 is *aleatoric*, meaning it reflects how the world itself varies.

> It's completely valid for a forecast to have small parameter uncertainty and small future covariate uncertainty, yet still produce largely variable individual observations as a result of large observation uncertainty.

These sources are explained in more detail below.

## Parameter uncertainty

The model is fit with MCMC, thus it doesn't return a single "best" set of parameters. Rather, it returns a whole posterior distribution of plausible parameter values, stored as posterior draws.

This uncertainty is propagated through the forecast. Because the model isn't sure of the exact parameter values, it isn't sure of the exact average response values those parameters imply. Parameter uncertainty therefore shows up in a forecast as uncertainty about where the average response is headed.

- **Where it comes from:** the range of posterior draws from the fitted model are used to predict a range of response variable values.
- **What it represents:** how confident the model is about the average predicted response values.
- **How it is represented on a plot:** a credible interval.

## Observation uncertainty

Even if the true parameter values were known, a real measurement isn't guaranteed to land on its average. Instead, measurements scatter around the average. This scatter is observation uncertainty and it does not go away with more data.

The model measures this scatter during fitting, and the likelihood family (Beta, Poisson, negative-binomial, etc) determines its shape. This is why a forecast predicts not just the average response, but also how far an individual observation is likely to fall from it.

- **Where it comes from:** the scatter the model measured during fitting.
- **What it represents:** how much an individual measurement could differ from the average (answers *"where would an actual measurement land?"*).
- **How it is represented on a plot:** a prediction interval.

> The prediction interval is always equal to or wider than the credible interval. This is because each `y_rep` value carries parameter uncertainty **plus** the observation scatter on top. So the prediction band is the credible band with the scatter layer added.

## Future covariate uncertainty

A forecast depends on the future covariate values that are provided by the user and therefore the methods that are used to produce these future covariate values. These predictions can differ because the future is unknown. Collecting multiple runs into an *ensemble* and looking at how far apart they fall gives you future covariate uncertainty.

- **Where it comes from:** the user-supplied future covariate values, which aggregate across model runs within a scenario.
- **What it represents:** how much confidence we have in the future values of our covariate(s).
- **How it is represented on a plot:** an ensemble envelope (the spread across runs).

> The ensemble envelope deliberately leaves out observation uncertainty. This is because mixing observation/measurement scatter with the disagreement between runs would blur two very different sources together.

## Future scenario "uncertainty"

Unlike the three sources above, a scenario (e.g. a wetter vs. drier future) isn't something the model is uncertain about, but rather an assumption the user makes to ask "what if the future looks like this?" Thus, scenarios are not collapsed into an uncertainty band. Instead, they are shown side by side as separate trajectories. Treat the scenario axis as *"what happens under each assumed future,"* and the three sources above as the uncertainty within each of those answers.
