---
title: Climate futures tutorial
weight: 2
bookToc: true
---

# Forecasting with Real Climate Data

This tutorial walks through combining your own response data with climate data to run the forecast pipeline. You'll use a companion repo to pull together observed and projected climate covariates for your park unit, then run those through the M4MD pipeline to generate forecasts under different climate futures.

If you haven't run a forecast before, the [mock data tutorial]({{< ref "/docs/5-forecasting/running/tutorials/mock-data" >}}) is a great place to start — it covers the same pipeline steps with a simpler synthetic dataset.

## Prerequisites

To follow along you'll need:

- Your response variable training data (e.g. vegetation observations)
- Plot location data (site coordinates)
- A model YAML file for your park unit

## Step 0: What you're working toward

The [climate-futures-covariates](https://github.com/mtaniguchiking/climate-futures-covariates) repo will produce two CSV files that plug directly into the M4MD pipeline:

| File | Used for |
|------|----------|
| Historical covariate CSV | Model fitting (`analysis-pipeline.R`) |
| Future scenarios CSV | Forecasting (`forecast-pipeline.R`) |

The historical file covers observed climate over your training period. The future scenarios file contains projected climate values for a set of GCM model runs, organized by climate future (e.g. hot-dry, warm-wet). Once you have both files, the rest of this tutorial is the same as any other forecast run.

For full details on the CSV format and column names, see the [repo README](https://github.com/mtaniguchiking/climate-futures-covariates).

## Step 1: Generate your climate covariates

Head over to the [climate-futures-covariates](https://github.com/mtaniguchiking/climate-futures-covariates) repo. The repo walks you through four Quarto notebooks that take you from raw climate data downloads all the way to the two pipeline-ready CSVs:

| Notebook | What it does |
|----------|-------------|
| `00-get-prism-loca2.qmd` | Downloads and crops observed (PRISM) and projected (LOCA2) climate data to your park boundary |
| `01-get-climate-futures.qmd` | Classifies GCM model runs into climate future quadrants (e.g. hot-dry, warm-wet) |
| `02-plot-prism-loca2.qmd` | *(Optional)* Visualizes the time-series and spatial summaries of your climate data |
| `03-prep-climate-covariates.qmd` | Extracts site-level climate values and writes the two final CSVs |

Each notebook has a **User config** block near the top — that's where you set your park code, site locations, and which climate variables to extract. The [repo README](https://github.com/mtaniguchiking/climate-futures-covariates) covers configuration in detail.

Once you've run all four notebooks, you'll have your historical covariate CSV and future scenarios CSV ready to go.

## Step 2: Fit the model

With your historical covariate CSV in hand, you're ready to fit. Open your model YAML and point it to the new climate data:

```yaml
# [TODO: fill in the relevant YAML fields that point to the historical covariate CSV]
```

The same guidance from the mock data tutorial applies here — add your covariates (`ppt`, `tmax`, or both) to the `additional covariates` field, consider whether you want a `time effect`, and make sure to pass `--save-forecast-inputs` so you can forecast afterward.

<!-- [TODO: note any differences from the mock data case — e.g. if real data requires extra YAML config] -->

For more on these options, see [fitting a model for forecasting]({{< ref "/docs/5-forecasting/running/guide/fitting-a-model" >}}).

Run the analysis pipeline interactively:

```R
yaml_file <- "path/to/your-model.yml"
save_forecast_inputs <- TRUE
```

Or from the CLI:

```sh
./analysis-pipeline.R path/to/your-model.yml --save-forecast-inputs
```

## Step 3: Run the forecast pipeline

Now point your forecast config at the future scenarios CSV. The key sections:

```yaml
paths:
  fitted_model_dir: path/to/your/fitted-model-output-dir
  scenarios_file: path/to/your/future-scenarios.csv

covariates:
  ppt:           # include whichever covariates you used in fitting
    source: provided
  tmax:
    source: provided
```

<!-- [TODO: add any notes about how scenario names in the CSV map to output folder names] -->

For a full reference of the config options, see [config files]({{< ref "/docs/5-forecasting/running/guide/config-files" >}}).

Run the forecast pipeline interactively:

```R
config_path <- "path/to/your-forecast-config.yaml"
```

Or from the CLI:

```sh
Rscript forecasting/forecast/forecast-pipeline.R \
    --config path/to/your-forecast-config.yaml
```

## Step 4: Reading your outputs

Forecast plots are written to the `04-forecast/` subfolder inside your fitted model's output directory — the same structure as the mock data run. The scenario labels in the output (folder names, plot legends) correspond to the climate future names from your scenarios file (e.g. `hot-dry`, `warm-wet`).

<!-- [TODO: any notes on interpreting ensemble spread or comparing across real climate scenarios] -->

For a complete reference of all output plots, see the [forecast outputs reference]({{< ref "/docs/5-forecasting/running/guide/outputs" >}}).

## Summary

In this tutorial you:

- Generated a historical covariate CSV and future scenarios CSV using the [climate-futures-covariates](https://github.com/mtaniguchiking/climate-futures-covariates) notebooks
- Fit a model using observed climate data
- Ran the forecast pipeline against projected climate scenarios
- Interpreted the outputs by climate future

<!-- [TODO: pointer to next steps, e.g. model diagnostics, config file reference, etc.] -->
