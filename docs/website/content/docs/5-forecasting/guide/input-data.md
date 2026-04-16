---
title: Input Data
weight: 2
---

# Forcast Input data

The forecasting pipeline requires two inputs:

1. A **fitted model directory** — written by the model fitting pipeline when run with `--save-forecast-inputs` (see [Fitting a model for forecasting](./fitting-a-model))
2. A **scenarios CSV** — future covariate projections you supply, described on this page

---

## Scenarios CSV

The scenarios CSV provides the covariate values the pipeline uses to construct future predictions. Each row represents one site in one year under one scenario and model run.

### Required columns

| Column | Description |
|---|---|
| `scenario_id` | Integer identifier for the climate scenario (e.g., `1`, `2`) |
| `scenario_name` | Human-readable scenario label (e.g., `"dry"`, `"wet"`) |
| `model_run_id` | Integer identifier for the individual model run within the scenario (e.g., a GCM ensemble member) |
| `model_run_name` | Human-readable model run label (e.g., `"ACCESS-CM2"`) |
| `site_id` | Site identifier — must match sites in the fitted model |
| `stratum_id` | Stratum identifier — must match strata in the fitted model |
| `cal_year` | Forecast year (integer) |
| *covariate columns* | One column per `provided` covariate, named to match the covariate (see below) |

> **Column names**: If your fitted model used non-standard column names (e.g., `Plot_ID` instead of `site_id`), the pipeline resolves these automatically using the column mappings stored in the model metadata. Use the same column names as in your original data.

### Covariate columns

Include one column for each covariate declared as `source: provided` in your forecast config. For example, if precipitation is `provided`, include a `ppt` column with projected values.

> Covariates using `hold_last` or `hold_mean` strategies do **not** need a column in the scenarios CSV — the pipeline fills those values automatically from training data.

### Scenarios and model runs

The four ID columns (`scenario_id`, `scenario_name`, `model_run_id`, `model_run_name`) allow you to encode multiple climate scenarios and multiple model runs per scenario in a single file. The pipeline processes each unique `(scenario_id, model_run_id)` combination as a separate forecast run.

For a single GCM projection under one scenario, you only need one unique combination (e.g., `scenario_id = 1`, `model_run_id = 1`).

For how multiple model runs and scenarios affect the output structure, see [Outputs](./outputs.md).

### Example

```csv
scenario_id,scenario_name,model_run_id,model_run_name,site_id,stratum_id,cal_year,ppt
1,dry,1,Mock-GCM-1,A1,A,2026,337
1,dry,1,Mock-GCM-1,A1,A,2027,303
1,dry,2,Mock-GCM-2,A1,A,2026,290
1,dry,2,Mock-GCM-2,A1,A,2027,315
2,wet,1,Mock-GCM-1,A1,A,2026,480
```
