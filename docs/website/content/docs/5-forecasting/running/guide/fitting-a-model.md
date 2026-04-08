---
title: Model fitting
weight: 1
---

# Fitting a model for forecasting

Before forecasting, you must fit a Bayesian hierarchical model that is configured with forecasting in mind. This page covers the key considerations and configurations for doing so. It assumes familiarity with the model fitting pipeline (detailed the other model fitting guide).

## `analysis-pipeline.R` command-line arguments

You must set the `save_forecast_inputs` argument to `TRUE`. This saves the model metadata and full posterior draws, which are the required inputs to the forecasting pipeline.

Also be mindful of the model scenario(s) you are selecting. This is determined by the `this_slice` variable and the `excl_null_mods` argument. For example, if you want to forecast with a non-null model containing a specific set of covariates, ensure your `this_slice` variable correctly selects that model.

## YAML configurations

*I may move parts of these YAML configuration explanations to the model fitting guide, but I'll keep them here for now*

### Per-stratum covariate responses

Choosing your covariate set matters for both model fitting and forecasting. Beyond *which* covariates to include, you also need to decide the *scope* of their effect: whether each covariate has a single shared coefficient across all strata, or a separate coefficient per stratum.

For covariates that you believe affect different strata to different degrees, add an interaction term between the covariate and your strata column. The model fitting guide describes how to define interaction terms in your YAML config [here]({{< ref "/docs/1-guide/b-config-files/iii-analysis-model/covariates.md" >}}).

**Example — per-stratum climate coefficients**

Suppose you want to fit and forecast a model with precipitation (`ppt`) as a covariate, and your response variable has two strata (defined in the stratum column `stratum_column`) where one responds to precipitation much stronger than the other. To capture this difference, add an interaction term in your model fitting YAML.

```yaml
additional covariates:
     - ppt, ppt*stratum_column
```

If you wanted to also capture per-stratum maximum temperature (`tmax`) responses, you could update to:

```yaml
additional covariates:
     - ppt, tmax, ppt*stratum_column, tmax*stratum_column
```

These interaction terms allow the model to learn separate per-stratum slopes for each covariate, which are then realized when projecting responses onto future scenario data.

### The linear time term

Another important decision is whether to include a linear time term in your model. A linear time term captures residual temporal trend — the portion of change in your response variable over time that your covariates don't explain. You might include it when you believe there are drivers of change not already captured by your covariates.

Including a time term for forecasting carries a strong assumption: **the linear trend continues at the same rate into the future.** This creates several problems:

- **Unbounded extrapolation** — a time trend projected 50 years forward may predict vegetation cover above 100% or below 0%.
- **Collinearity with climate** — if a covariate is also trending (e.g., warming or drying), the model cannot cleanly attribute change to covariate versus time.
- **Mechanism conflated with trend** — a climate-covariate forecast may be designed to isolate a specific question: "how does vegetation respond to a climatic trend?" A time term undermines this by attributing part of the projected change to a continuation of historical trend rather than to climate forcing.

Thus, when fitting a model for forecasting, you should include the time term only if there is a known secular process that will continue through the forecast horizon. For example, if your system is demonstrably not at equilibrium and has/will grow linearly over both the historical training *and* forecast periods.
