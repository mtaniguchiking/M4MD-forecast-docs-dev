---
title: Custom data guide
weight: 3
bookToc: true
---

# Running a Forecast with Custom Data

<!-- TODO: Intro paragraph — this is the full reference for users bringing
     entirely their own data (response, covariates, and scenarios). No
     external tool required. This page is denser than the walkthroughs
     and serves as a reference rather than a tutorial.
-->

## Required Input Files

<!-- TODO: Describe each input file with column schemas:
     1. Response variable CSV — columns, units, required fields
     2. Covariates CSV (training) — columns, format
     3. Scenarios CSV (future) — columns, scenario_id vs model_run_id distinction,
        how multiple GCM runs are represented
     Cross-reference forecasting/getting-started/generate-mock-data.R
     for concrete examples of each format.
-->

## Forecast Config YAML Reference

<!-- TODO: Document every key in the forecast config YAML:
     - Required keys (paths to inputs, output dir, likelihood)
     - Covariate strategy options: provided / hold_last / hold_mean
     - Scenario filtering options
     - n_cores, seed, etc.
     Source: forecasting/forecast/read-forecast-config.R
-->

## Running the Pipeline

```bash
Rscript forecasting/forecast/forecast-pipeline.R \
  --config path/to/forecast-config.yaml
```

<!-- TODO: Note any useful flags (--dry-run, --verbose, etc. if they exist).
     Describe what a successful run looks like in the console. -->

## Output Files

<!-- TODO: Document the output directory layout and key files:
     - Trajectory plots (per-site, per-stratum)
     - Spaghetti plots
     - forecast-site-summaries.csv — describe each column
     - mu_median vs y_rep_median — explain the distinction in plain English
     Source: forecasting/forecast/forecast-plot-engine.R
-->
