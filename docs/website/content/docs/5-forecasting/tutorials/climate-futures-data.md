---
title: Climate futures tutorial
weight: 2
bookToc: true
---

# Forecasting with Real Climate Data

This tutorial walks through combining your own response data with climate data to run the forecast pipeline. You'll use a temporary, external repo to pull together observed and projected climate covariates for your park unit, then run those through the M4MD pipeline to generate forecasts under different climate futures.

If you haven't run a forecast before, the [mock data tutorial]({{< ref "/docs/5-forecasting/tutorials/mock-data" >}}) is a great place to start.

## Prerequisites

To follow along you'll need:

- Annual response variable training data (e.g. vegetation observations)
- Plot location data (site coordinates)
- A model YAML file for your park unit

## Step 0: Generate your climate covariates

Head over to the [climate-futures-covariates](https://github.com/mtaniguchiking/climate-futures-covariates) repository. The [repo README](https://github.com/mtaniguchiking/climate-futures-covariates) is a good place to start. The repo walks you through four Quarto notebooks that take you from raw climate data downloads to pipeline-ready CSVs:

| Notebook | What it does |
|----------|-------------|
| `00-get-prism-loca2.qmd` | Downloads and crops observed (PRISM) and projected (LOCA2) climate data to your park boundary |
| `01-get-climate-futures.qmd` | Classifies GCM model runs into climate future quadrants (e.g. hot-dry, warm-wet) |
| `02-plot-prism-loca2.qmd` | *(Optional)* Visualizes the time-series and spatial summaries of your climate data |
| `03-prep-climate-covariates.qmd` | Extracts site-level climate values and writes the two final CSVs |

Each notebook has a **User config** block near the top. This is where you set your park code, site locations, and which climate variables to extract.

Once you've run all four notebooks, you'll have your historical covariate CSV and future scenarios CSV ready to go.

## Step 1: Fit the model

With your historical covariate CSV in hand, you're ready to fit. We will be editing your YAML to match this CSV, so save a copy of your original YAML if needed! Open your model YAML (or a copy of it) and point it to the new climate data:

```yaml
covariate info:
  file: assets/_data/<fit_output_csv>.csv
  event date info:
    date-time column: <year_col>
    date-time format: Y!
```

The same guidance from the [mock data tutorial]({{< ref "/docs/5-forecasting/tutorials/mock-data/#step-1-fit-the-model" >}}) applies here. That is, add your covariates (`ppt`, `tmax`, or both) to the `additional covariates` field and optionally stratum interactions. Also consider whether you want a `time effect`:

```yaml
additional covariates:
# remove the ppt or the tmax terms if you only selected one of
# them in 03-prep-climate-covariates.qmd

# remove the interaction terms if you don't want to capture
# stratum-specific responses
  - ppt, tmax, ppt*<stratum_col>, tmax*<stratum_col>

time effect: disabled # optional
```

For more on these options, see [fitting a model for forecasting]({{< ref "/docs/5-forecasting/guide/fitting-a-model" >}}).

Once the YAML is ready, review your `analysis-pipeline.R`. You can run the analysis pipeline interactively with the following parameters:

```R
yaml_file <- "path/to/your-model.yml"
            # point to the YAML described above

# assign other parameters according to your specific model + data

excl_null_mods <- TRUE # we won't be using the null model
save_forecast_inputs <- TRUE # so we can forecast later!
```

Or from the CLI:

```sh
./analysis-pipeline.R path/to/your-model.yml \
    --save-forecast-inputs --excl-null
# assign other parameters according to your specific model + data
```

## Step 2: Inspect the output (optional)

Before forecasting, it's worth checking the model fitting output in its respective folder (likely in `assets/_output/...`). There should be a `04-forecasting` folder containing posterior draws and model metadata — this is saved because of the `--save-forecast-inputs` flag we passed when fitting.

If you'd like to evaluate the model, see [model diagnostics]({{< ref "/docs/1-guide/c-outputs/output-dir/01-diagnostics" >}}) and [model checking]({{< ref "/docs/1-guide/c-outputs/output-dir/02-checking" >}}). If the model was not successful in fitting (e.g. failure to converge, poor Bayesian p values), play around with your configs or switch to a new dataset before continuing.

## Step 3: Run the forecast pipeline

Now it's time for your `forecast-pipeline.R` and forecast YAML. A forecast YAML template you can fill in is available at `forecasting/climate-futures-tutorial/forecast-config-template.yaml`. Here are the sections to focus on:

```yaml
# ==== PATHS ==================================================================

paths:
  fitted_model_dir: assets/_output/...
  scenarios_file: assets/_data/<forecast_output_csv> # from your 03-prep-climate-covariates.qmd
  plot_locations_file: assets/_data/<plot_data_csv> # this should match your model fitting YAML

# ==== MODEL ==================================================================

covariates:
  ppt:
    source: provided
  tmax:
    source: provided

```

For a full reference of the config options, see [config files]({{< ref "/docs/5-forecasting/guide/config-files" >}}).

To run the forecast pipeline interactively, set your `forecast-pipeline.R` configuration as follows.

```R
config_path <- "forecasting/climate-futures-tutorial/<your-forecast-config.yaml>"
```

Then source your `forecast-pipeline.R`. Alternatively, run the forecast pipeline from the CLI.

```bash
Rscript forecasting/forecast/forecast-pipeline.R \
    --config forecasting/climate-futures-tutorial/<your-forecast-config.yaml>
```

## Step 4: Reading your outputs

Forecast plots are written to the `04-forecast/` subfolder inside your fitted model's output directory (the same structure as the mock data run). The scenario labels in the output correspond to the climate future names from your scenarios file (e.g. `hot-dry`, `warm-wet`).

Analysis of your particular forecast depends on the underlying model + data. For a complete reference of all output plots, see the [forecast outputs reference]({{< ref "/docs/5-forecasting/guide/outputs" >}}).

## Summary

In this tutorial you:

- Generated a historical covariate CSV and future scenarios CSV using the [climate-futures-covariates](https://github.com/mtaniguchiking/climate-futures-covariates) notebooks
- Fit a model using observed climate data
- Ran the forecast pipeline against projected climate scenarios
- Interpreted the outputs by climate future

<!-- [TODO: pointer to next steps, e.g. model diagnostics, config file reference, etc.] -->
