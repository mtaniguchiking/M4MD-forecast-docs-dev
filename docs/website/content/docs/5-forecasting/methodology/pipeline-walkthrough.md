---
title: Pipeline Walkthrough
weight: 2
---

# Pipeline Walkthrough: `forecast-pipeline.R`

`forecast-pipeline.R` is the entry point for running a forecast. You give it one thing — a forecast configuration YAML file — and it takes a fitted model and a set of future scenarios and turns them into forecast summaries, plots, and diagnostics.

```bash
Rscript forecasting/forecast/forecast-pipeline.R my-forecast-config.yaml
```

Internally it works through 11 steps. The first six are setup (load inputs, line up parameters with covariates, standardize the scenarios). Step 7 is the heavy lift where posterior predictive samples are actually generated. Steps 8–11 turn those samples into the CSVs and plots a user looks at. Outputs land in `<fitted_model_dir>/04-forecast/`, organized by scenario and model run.

The script is designed to be readable top-to-bottom. Each step prints a banner to the console, so when something fails you can usually tell which step you were in. The math behind what is happening at each step (the link function, the predictive sampler, etc.) lives in *The Math*; this page is just the tour.

---

## Step 1: Load and Validate Configuration

Reads the forecast config YAML and checks that everything it points to actually exists and makes sense. This includes paths to the fitted model, the scenarios CSV, the covariate strategies (per covariate: is it `provided`, `hold_last`, or `hold_mean`?), and bookkeeping like `random seed`, `max draws`, summary quantiles, and the output prefix.

It also opens the fitted model's `forecasting-metadata.rds` to pull out the model's likelihood family, link function, and column-name mappings. These are needed for nearly every later step, so the script fails fast here if any of them are missing.

## Step 2: Load Forecasting Metadata

Loads the rest of the artifacts the fitting pipeline left behind: site-in-stratum and stratum-id lookup tables (the maps between human-readable IDs and the integer indices the model uses internally), and `covariate-moments.rds` (the training-data mean and SD for each covariate, used later to put future values on the same scale the model was fit on).

This is the step where the forecast pipeline picks up the "shape" of the fitted model — which sites belong to which strata, how many of each there are, etc.

## Step 3: Load Posterior Draws

Reads `mcmc-draws-full.csv`, the table of posterior parameter samples from the fitted model. Each row is one MCMC iteration; each column is one parameter. If the config sets `max draws`, the script subsamples here to keep memory in check.

It then scans the column names to figure out which columns are which: site intercepts (`B0[...]`), trend slopes (`B1[...]` — present or absent depending on how the model was fit), fixed-effect coefficients (`Beta[...]`), and dispersion parameters (`sigma[...]`, `kappa[...]`, or `delta[...]`, depending on the likelihood family). The script enforces that site-level intercepts exist — the forecast pipeline currently requires them.

## Step 4: Build Fixed-Effect Lookup

Lines up the model's fixed-effect coefficients (`Beta[1]`, `Beta[2]`, ...) with covariate names that mean something to a human (e.g., `fe_ppt`, `fe_pptXMaster_Stratificationgrassland`). This is the alignment that makes the dot product in Step 7 give the right answer — `Beta[2]` only means anything if you know which covariate column it multiplies.

Interaction terms (like `ppt × Master_Stratification`) are reconstructed here from the metadata, so the user doesn't have to spell them out in the scenarios CSV.

## Step 5: Load and Prepare Scenarios

Reads the user-supplied scenarios CSV, applies any column-name renames declared in the config, and attaches the site/stratum integer indices from Step 2. Each row of the scenarios CSV represents one combination of (site, stratum, year, scenario, model run).

The script then identifies the unique `scenario × model_run` combinations present in the data. A "scenario" is a what-if (e.g., RCP 4.5), and a "model run" is a single climate-model realization of that scenario. Each combination is forecasted independently and bundled into an ensemble in Step 9.

## Step 6: Build Covariate Matrix on Scenario Data

Turns the raw scenarios into a numeric design matrix that lines up with `Beta`. Three things happen per covariate:

1. **Source the raw value.** Either from the scenarios CSV (`provided`), the last observed training value (`hold_last`), or the training mean (`hold_mean`). The strategy is set per covariate in the config.
2. **Standardize.** Center and scale using the training-data moments from Step 2, so the future values are on the same scale the model was fit on.
3. **Build interaction columns.** Continuous × stratum-dummy and continuous × continuous interactions are constructed automatically.

The script also checks that the draws contain the dispersion columns the likelihood family expects (e.g., `kappa[k]` for negative-binomial-simple). If they're missing it fails here rather than 100,000 predictive samples later.

## Step 7: Compute Forecasts

This is where the actual forecasting happens. For every `scenario × model_run` combination and every posterior draw, the pipeline computes the linear predictor (site intercept + trend term + covariate effects), pushes it through the inverse link to get a mean response on the natural scale, and then samples one realization from the response distribution. The result is a draw-level posterior predictive distribution for every (site, stratum, year) in the scenarios.

If `--n-cores` is greater than 1, the `scenario × model_run` combinations are distributed across worker processes via the `parallel` package — they share no state, so this parallelizes cleanly.

## Step 8: Summarize and Save Run-Level Outputs

For each `scenario × model_run` combination, the raw draws are collapsed into summary statistics — means, medians, and the user-requested credible intervals — at the site, stratum, and park levels (whichever the config asks for). These summaries are saved as CSV files in `04-forecast/runs/<scenario>/<model_run>/`, along with a bundled `.rds` that records the exact quantile levels used.

Run-level forecast plots (trajectory bands per site/stratum/park) are also generated and saved alongside the CSVs. Each run is also checked here for extrapolation — whether its scenario covariates push past the year range the model was trained on, with a warning if so.

## Step 9: Ensemble Summarization

If a scenario has more than one model run (e.g., several climate-model realizations of the same RCP), the pipeline collapses them into a single ensemble. Mean predictions are pooled across runs to produce ensemble bands; predictive draws (`y_rep`) are intentionally *not* pooled, since their meaning is run-specific.

Ensemble summaries are saved to `04-forecast/runs/<scenario>/ensemble/`, with their own set of plots. If a scenario has only one model run, this step is skipped.

## Step 10: Cross-Scenario Comparison

If the run produced more than one scenario (e.g., RCP 4.5 vs. RCP 8.5), the pipeline generates comparison plots that overlay the ensemble bands from each scenario side-by-side. With only one scenario, this step is skipped.

The output goes to `04-forecast/runs/comparison/`.

## Step 11: Generate Diagnostic Plots

A final set of plots is generated that combines everything — all scenarios, all model runs, all summary levels — into one place for sanity-checking. To keep memory in check, this step reuses the run-level summaries from Step 9 rather than re-reading the full predictive draws.

Diagnostic plots are saved to `04-forecast/diagnostics/`.

---

After Step 11 the pipeline prints `Forecasting Complete!` and exits. At this point the `04-forecast/` directory contains everything: run-level CSVs and plots, ensemble outputs, cross-scenario comparison plots, and diagnostic plots.
