---
title: Pipeline Walkthrough
weight: 2
---

# Pipeline Walkthrough: `forecast-pipeline.R`

`forecast-pipeline.R` is the orchestrator script for running a forecast. After a user provides a forecast config [YAML file]({{< ref "/docs/5-forecasting/guide/config-files" >}}), it produces forecast summaries, plots, and diagnostics.

> `forecast-pipeline.R` can be thought of as a sister script to `analysis-pipeline.R`. Both scripts expect a user provided config file, and both act as orchestrators that source other scripts to produce their outputs.

It works through 11 steps. The first six are setup (load inputs, line up parameters with covariates, standardize the scenarios); Step 7 is where posterior predictive samples are generated; and Steps 8–11 turn those samples into the CSVs and plots a user looks at.

Each step in this script prints a banner to the console, so if something errors you can usually tell which step you were in. The underlying math, especially during Step 7, is described in the [math section]({{< ref "/docs/5-forecasting/methodology/math" >}}).

---

## Step 1: Load and Validate Configuration

Reads the forecast config YAML and checks if the expected files exist and field values are valid. This includes paths to the fitted model, the scenarios CSV, the covariate strategies, and bookkeeping like `random seed`, `max draws`, `summary levels`, and `summary quantiles`.

It also opens the fitted model's `forecasting-metadata.rds` to pull out the model's likelihood family, link function, and column-name mappings (this file is re-loaded in Step 2, but it's looked at here to validate the config).

## Step 2: Load Forecasting Metadata

Loads the rest of the artifacts the fitting pipeline left behind: site-in-stratum and stratum-id lookup tables, and `covariate-moments.rds`.

This is the step where the forecast pipeline notes the "shape" of the fitted model, such as which sites belong to which strata and how many of each there are.

## Step 3: Load Posterior Draws

Reads `mcmc-draws-full.csv`, the table of posterior parameter samples from the fitted model. If the config sets `max draws`, the script subsamples here to keep memory manageable.

It then scans the column names to figure out which columns are which: site intercepts (`B0[...]`), trend slopes (`B1[...]`), fixed-effect coefficients (`Beta[...]`), and dispersion parameters (`sigma[...]`, `kappa[...]`, or `delta[...]`). The script also checks that the draws contain the dispersion columns the likelihood family expects.

## Step 4: Build Fixed-Effect Lookup

Lines up the model's fixed-effect coefficients (`Beta[1]`, `Beta[2]`, ...) with readable covariate names (e.g. `precipitation`, `temp`).

Interaction terms (like `ppt × Stratification`) are reconstructed here from the metadata, so the user doesn't have to spell them out in the scenarios CSV.

## Step 5: Load and Prepare Scenarios

Reads the user-supplied scenarios CSV, applies any column-name renames declared in the config, and attaches the site/stratum integer indices from Step 2.

The script then identifies the unique `scenario × model_run` combinations present in the data. A "scenario" is a what-if (e.g. "RCP 4.5", "warm-wet"), and a "model run" is a single climate-model realization of that scenario.

## Step 6: Build Covariate Matrix on Scenario Data

Turns the raw scenarios into a numeric design matrix that lines up with `Beta`. Three things happen per covariate:

1. **Source the raw value.** Either from the scenarios CSV (`provided`), the last observed training value (`hold_last`), or the training mean (`hold_mean`). The strategy is set per covariate in the config.
2. **Standardize.** Center and scale using the training-data moments from Step 2, so the future values are on the same scale the model was fit on.
3. **Build interaction columns.** Continuous × stratum-dummy and continuous × continuous interactions are constructed automatically.

## Step 7: Compute Forecasts

This is where the forecasting computation happens. For every `scenario × model_run` combination and every posterior draw, the pipeline computes the linear predictor (site intercept + trend term + covariate effects), applies the inverse link to get a mean response, and then samples one realization from the response distribution. The result is a draw-level posterior predictive distribution for every (site, stratum, year) in the scenarios.

If `--n-cores` is greater than 1, the `scenario × model_run` combinations are distributed across worker processes.

## Step 8: Summarize and Save Run-Level Outputs

For each `scenario × model_run` combination, the raw draws are collapsed into summary statistics (means, medians, and the user-requested credible intervals) at each level requested in the config's `summary levels`. These summaries are saved as CSV files in `04-forecast/runs/<scenario>/<model_run>/`, along with a bundled `.rds` that records the user-defined quantile levels used.

Run-level forecast plots (trajectory bands per site/stratum/park) are also generated and saved alongside the CSVs. Each run is checked for extrapolation (whether its scenario covariates push past the year range the model was trained on).

## Step 9: Ensemble Summarization

If a scenario has more than one model run (e.g. several climate-model realizations of the same climate futures or RCP scenario), the pipeline collapses them into a single ensemble to produce ensemble bands.

Ensemble summaries are saved to `04-forecast/runs/<scenario>/ensemble/`, with their own set of plots. If a scenario has only one model run, no ensemble CSVs or plots are written.

## Step 10: Cross-Scenario Comparison

If the user-provided scenarios CSV contains more than one scenario (e.g. RCP 4.5 vs. RCP 8.5), the pipeline generates comparison plots that overlay the ensemble bands from each scenario side-by-side. If only one scenario exists, this step is skipped.

The output goes to `04-forecast/runs/comparison/`.

## Step 11: Generate Diagnostic Plots

A final set of plots is generated that combines things into one place for sanity-checking.

Diagnostic plots are saved to `04-forecast/diagnostics/`.

---

After Step 11 the pipeline prints `Forecasting Complete!` and exits. At this point the `04-forecast/` directory contains run-level CSVs and plots, ensemble outputs, cross-scenario comparison plots, and diagnostic plots.
