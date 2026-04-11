---
title: Forecasting
bookCollapseSection: true
weight: 5
---

# Forecasting

This section documents the forecasting extension developed by the [Eric and Wendy Schmidt Center for Data Science & Environment](https://dse.berkeley.edu/) in collaboration with the National Park Service's Inventory & Monitoring Division.
The forecasting pipeline takes in a fitted model and produces posterior predictive forecasts under user-specified future climate scenarios.

In context of this repository, "model fitting" refers to running `analysis-pipeline.R`, which fits a Bayesian hierarchical model to historical data via MCMC (as described in the previous sections of this guide). "Forecasting" refers to running `forecast-pipeline.R`, which reads a fitted model's posterior draws and projects the response variable forward.

This forecasting documentation contains two subsections:

---

{{< hint info >}}
**Run a forecast**

- Start here for your first forecast with a step-by-step walkthrough and mock data 
- Learn how to forecast with your own data

→ [Running a Forecast]({{< ref "/docs/5-forecasting/running" >}})
{{< /hint >}}

{{< hint warning >}}
**Understand a forecast**

- Go here to understand the underlying posterior predictive framework
- Explore what each step of the forecast pipeline is doing

→ [Understanding a Forecast]({{< ref "/docs/5-forecasting/understanding" >}})
{{< /hint >}}
