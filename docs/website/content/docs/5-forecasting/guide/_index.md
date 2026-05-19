---
title: Guide
bookCollapseSection: true
weight: 2
---

# The forecasting pipeline

Producing a forecast involves several steps across the model fitting and forecasting pipelines. This page generalizes this process into 4 steps.

---

## Workflow overview

The diagram below sketches the full process. Each step is described in more detail in the checklist that follows.

[TODO: I need to add workflow diagram here]

---

## Step-by-step checklist

### 1. Get your data

Gather two types of data before you start:

- **Training data** — the observations you'll use to fit the model (e.g. response and covariate data)
- **Future scenario data** — projected covariate values (e.g. climate scenarios) for the forecast period, formatted as a scenarios CSV

> For details on response and covariate data requirements, see the [model fitting data]({{< ref "/docs/1-guide/a-data/i-y-info" >}}) guide.  
> For details on future scenario data requirements, see the [forecasting data]({{< ref "/docs/5-forecasting/guide/input-data" >}}) guide.

---

### 2. Fit your model

Configure and run the model fitting pipeline with forecasting in mind.

> For full model fitting YAML configuration details, see [Config files]({{< ref "/docs/1-guide/b-config-files" >}}).  
> For forecasting-specific decisions, see [Fitting a model for forecasting]({{< ref "/docs/5-forecasting/guide/fitting-a-model.md" >}}).

---

### 3. Evaluate your model

Before forecasting, make sure the model is a good fit for your data. For example:

- Review MCMC convergence diagnostics (R-hat, trace plots, effective sample size)
- Check posterior predictive performance — does the model reproduce the observed data?

If the model doesn't fit well, revisit your covariate choices or model structure before forecasting

> For details on available diagnostics and outputs, see [Outputs]({{< ref "/docs/1-guide/c-outputs/output-dir" >}}) in the model fitting guide.

---

### 4. Forecast

With a valid, fitted model, you can run your forecasts.

1. If you haven't already, prepare your [scenarios CSV]({{< ref "/docs/5-forecasting/guide/input-data" >}}) with future covariate values
2. Write a [forecast config YAML]({{< ref "/docs/5-forecasting/guide/config-files.md" >}})
3. Run the forecasting pipeline:
   ```bash
   Rscript forecasting/forecast/forecast-pipeline.R path/to/your-forecast-config.yaml
   ```
4. Inspect the [outputs]({{< ref "/docs/5-forecasting/guide/outputs" >}}). Summary CSVs and plots are written to `04-forecast/` inside your fitted model directory.

> For a guided walkthrough with example data, see the [Mock data tutorial]({{< ref "/docs/5-forecasting/tutorials/mock-data" >}}).
