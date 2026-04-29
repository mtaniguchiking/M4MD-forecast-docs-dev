---
title: Input Data
weight: 2
---

# Forcast Input data

The forecasting pipeline requires three inputs:

1. A **fitted model directory** — data written by the model fitting pipeline when run with `--save-forecast-inputs` (see [Fitting a model for forecasting]({{< ref "/docs/5-forecasting/guide/fitting-a-model.md" >}}))
2. A **scenarios CSV** — future covariate projections you supply **(described on this page)**
3. A **forecast config YAML** — a file specifying input paths, covariate handling, and output options (see [Config files]({{< ref "/docs/5-forecasting/guide/config-files.md" >}}))

---

## Scenarios CSV

The scenarios CSV provides the covariate values the pipeline uses to construct future predictions. Each row represents one site in one year under one scenario and model run.

### Required columns

| Column | Description |
|---|---|
| `scenario` | Scenario label (e.g., `"dry"`, `"wet"`) |
| `model_run` | Model run label (e.g., `"ACCESS-CM2"`) |
| `site_id` | Site identifier — must match sites in the fitted model |
| `stratum_id` | Stratum identifier — must match strata in the fitted model |
| `cal_year` | Forecast year (integer) |
| *covariate columns* | One column per `provided` covariate, named to match the covariate (see below) |

> **Column names**: If your fitted model used non-standard column names (e.g., `Plot_ID` instead of `site_id`), the pipeline resolves these automatically using the column mappings stored in the model metadata. Use the same column names as in your original data.

### Covariate columns

Include one column for each covariate declared as `source: provided` in your forecast config. For example, if precipitation is `provided`, include a `ppt` column with projected values.

> Covariates using `hold_last` or `hold_mean` strategies do **not** need a column in the scenarios CSV — the pipeline fills those values automatically from training data.

### Scenarios and model runs

The `scenario` and `model_run` columns allow you to encode multiple climate scenarios and multiple model runs per scenario in a single file. The pipeline processes each unique `(scenario, model_run)` combination as a separate forecast run.

### Example (from [mock data tutorial]({{< ref "/docs/5-forecasting/tutorials/mock-data" >}}))

| `scenario` | `model_run` | `site_id` | `stratum_id` | `cal_year` | `ppt` |
|---|---|---|---|---|---|
| dry | Mock-GCM-1 | A1 | A | 2026 | 337 |
| dry | Mock-GCM-1 | A1 | A | 2027 | 303 |
| dry | Mock-GCM-2 | A1 | A | 2026 | 290 |
| dry | Mock-GCM-2 | A1 | A | 2027 | 315 |
| wet | Mock-GCM-1 | A1 | A | 2026 | 480 |
