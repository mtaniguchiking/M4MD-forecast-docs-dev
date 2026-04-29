---
title: Outputs
weight: 4
---

**\*WORK IN PROGRESS (this is somewhat empty for now)***

# Forecast Outputs

The forecasting pipeline writes all outputs into `04-forecast/` subdirectories inside your `fitted_model_dir`. The structure depends on how many scenarios and model runs are in your scenarios CSV.

---

## Scenarios and model runs

Each unique combination of `(scenario, model_run)` in your scenarios CSV is treated as a separate forecast run. The pipeline then:

1. Produces **run-level outputs** for each individual run
2. **Ensembles** across model runs within each scenario (if a scenario has more than one model run)
3. **Compares** across scenarios (if there is more than one scenario)

---

## Directory structure

```
04-forecast/
  forecast-inputs/         (written by fitting pipeline — do not modify)
  runs/
    <scenario>/
      <model_run>/         (one folder per model run)
        forecast-site-summaries.csv
        forecast-stratum-summaries.csv
        forecast-bundle.rds
        forecasts/
          [TODO: plot filenames]
      ensemble/            (only when scenario has >1 model run)
        <scenario>-ensemble-site-summaries.csv
        <scenario>-ensemble-stratum-summaries.csv
        <scenario>-ensemble-bundle.rds
        ensemble/
          [TODO: plot filenames]
    comparison/            (only when there is >1 scenario)
      comparison/
        [TODO: plot filenames]
  diagnostics/
    diagnostics/
      [TODO: plot filenames]
```

---

## Summary CSVs

Summary CSVs are written at each spatial level you specified in `summary levels`. The filename pattern is `{output prefix}-{level}-summaries.csv` (e.g., `forecast-site-summaries.csv`).

### Site- and stratum-level columns

| Column | Description |
|---|---|
| `scenario` | Scenario label |
| `model_run` | Model run label |
| `site_id` | Site identifier (site-level only) |
| `stratum_id` | Stratum identifier |
| `cal_year` | Forecast year |
| `mu_mean` | Mean of the posterior predictive mean (probability scale) |
| `mu_median` | Median of the posterior predictive mean |
| `mu_q025` / `mu_q975` | Lower and upper quantiles of the posterior predictive mean (bounds set by `summary quantiles` in config) |
| `cover_pct_mean` | `mu_mean` expressed as a percentage (×100) |
| `y_rep_mean` | Mean of the posterior predictive distribution (observation scale, includes dispersion) |
| `y_rep_q025` / `y_rep_q975` | Credible interval on the observation-scale predictions |

> **`mu` vs `y_rep`**: `mu` columns summarize the expected response (the latent mean), while `y_rep` columns include observation-level noise. For most ecological interpretations, `mu` columns are the primary quantity of interest.

### Ensemble-level columns

Ensemble CSVs (written when a scenario has >1 model run) collapse across model runs:

| Column | Description |
|---|---|
| `scenario`, `stratum_id`, `cal_year` | Grouping identifiers |
| `n_runs` | Number of model runs included in the ensemble |
| `ensemble_mean` | Mean of run-level `mu_median` values across model runs |
| `ensemble_median` | Median of run-level `mu_median` values |
| `ensemble_q025` / `ensemble_q975` | Spread across model runs (reflects climate model uncertainty) |
| `ensemble_min` / `ensemble_max` | Range across model runs |

> The ensemble does not include `y_rep` — combining observation noise across climate model runs conflates two statistically distinct sources of uncertainty.

---

## Plots

The pipeline produces several plot types:

### Run-level plots (in `runs/<scenario>/<model_run>/forecasts/`)

[TODO: describe what these show — trajectory plots per site and/or stratum with historical overlay and credible interval band]

### Ensemble plots (in `runs/<scenario>/ensemble/`)

Generated only when a scenario has more than one model run.

[TODO: describe spaghetti + envelope plot and time-slice violin plot]

### Comparison plots (in `runs/comparison/`)

Generated only when there is more than one scenario.

[TODO: describe cross-scenario overlay plots]

### Diagnostic plots (in `diagnostics/`)

Combines all scenarios and runs. Useful for a high-level view of forecast behavior.

[TODO: describe what diagnostic plots show]

---

## The forecast bundle (`.rds`)

Each output directory also contains a `*-bundle.rds` file — an R list with the summary data frames and scenarios data. This is useful for custom downstream plotting without re-running the pipeline.

```r
bundle <- readRDS("04-forecast/runs/dry/GCM-1/forecast-bundle.rds")
bundle$summaries$site   # site-level summary data frame
bundle$summaries$stratum
```
