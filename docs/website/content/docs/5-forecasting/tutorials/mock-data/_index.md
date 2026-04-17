---
title: Mock data walkthrough
weight: 1
bookToc: true
---

# Your First Forecast (Mock Data)

This tutorial walks through a simple forecast run using synthetic data. You'll fit a vegetation cover model, generate forecasts under two climate scenarios, and interpret how the model's learned patterns manifest in these forecasts.

## Prerequisites

See the [getting started]({{< ref "/docs/0-getting-started/installation.md" >}}) section if you haven't used the model fitting pipeline before. The model fitting steps (Steps 1 and 2) are pretty brief, so if you're unfamiliar with the model fitting process, the linked guides in each section are a good place to start.

> File/folder paths in this tutoral assume your working directory is set to the repository root directory (i.e. `/M4MD`).

## Step 0: Understand the mock data

This mock dataset represents a park unit (ELDO) with two vegetation strata (A and B), six sites per stratum, and one transect per site. The data is annual from 2000 to 2025. Each transect records how many of 100 sample points hit a plant (`y_hits` out of `n_points = 100`). A single climate covariate, precipitation (ppt), declines gradually over the training period and drives the vegetation response.

There are three data files we'll be using, pre-committed to `assets/_data/`:

| File | What it contains |
|------|-----------------|
| `mock-response.csv` | Transect-level vegetation observations (one row per transect-year) |
| `mock-covariate.csv` | Annual precipitation per site (one row per site-year) |
| `mock-covariate-scenarios.csv` | Future precipitation under two climate scenarios |

{{< hint info >}}
You don't need to generate the data because it's already provided. The script that created it is at `forecasting/mock-data-tutorial/generate-mock-data.R` if you want to play around with generating alternative mock data.
{{< /hint >}}

### What the data looks like

Vegetation hit rate tracks precipitation closely, with stratum A starting at a higher baseline (~75%) than stratum B (~45%). Both strata decline as precipitation falls.

![Vegetation hit rate by stratum over time, with precipitation overlaid](01-stratum-hit-rates.png)

The precipitation–vegetation relationship is highly correlated. This will be learned by the model and carried into the forecasts.

<div style="text-align: center;">
<img src="02-hits-vs-ppt.png" alt="Hit rate vs. precipitation scatter plot" width="500">
</div>

The future scenarios are represented by 1 GCM model each, covering two potential climate futures.

![Climate scenarios: historical and three future trajectories](03-climate-scenarios.png)

{{< hint info >}}
This mock data is intentionally simplified (e.g. the vegetation response to precipitation is exaggerated), so that patterns are easier to see and model outputs are more interpretable.
{{< /hint >}}

## Step 1: Fit the Model

Now that we understand the data, we can fit a model. For our mock model, we run the analysis pipeline with the mock cover YAML file. By default, this YAML is located at `assets/_config/M4MD/ELDO/mock-cover.yml`.

There are two key configurations in this YAML:

1) The `additional covariates` field tells the model which predictors to include. Here we add
a stratum interaction so the model fits a separate precipitation weight per stratum. Covariate interaction
terms are described in the [covariates]({{< ref "/docs/1-guide/b-config-files/iii-analysis-model/covariates" >}})
section.

```yaml
additional covariates:
  - ppt, ppt*stratum
```

2) By default the model includes a linear time trend. We disable it here so forecasts are
driven by precipitation alone, not by an extrapolated time trend:

```yaml
time effect: disabled
```

These configurations discussed in more detail in the [fitting a model for forecasting]({{< ref "/docs/5-forecasting/guide/fitting-a-model" >}}) section.

To run the analysis pipeline interactively, set your `analysis-pipeline.R` configurations as follows.

```R
yaml_file <- "assets/_config/M4MD/ELDO/mock-cover.yml"
            # point to the YAML described above

n_adapt <- 1000
n_update <- 5000
n_iter <- 2500
n_cores <- 1
override_dqs <- FALSE
excl_null_mods <- TRUE # we won't be using the null model
save_mcarray <- FALSE
save_forecast_inputs <- TRUE # so we can forecast later!
pass_errors <- FALSE

this_slice <- c(1)
```

Then source your `analysis-pipeline.R`. Alternatively, run the model fitting pipeline from the CLI.

```sh
./analysis-pipeline.R assets/_config/M4MD/ELDO/mock-cover.yml \
    --save-forecast-inputs --excl-null
```

A successful run should output a completed model initialization and finish with diagnostic trace plots.

```
Initializing model

  |++++++++++++++++++++++++++++++++++++++++++++++++++| 100%
  |**************************************************| 100%
  |**************************************************| 100%
  |**************************************************| 100%

#  ... intermediate outputs ...

Diagnostics: trace plot(s)...
|=======================================================|100% ~0 s remaining
```

## Step 2: Inspect the Output (optional)

We now have a fitted model! Before forecasting, it's worth checking the model output at `assets/_output/M4MD/ELDO/mock-cover/y_hits/binom_inv-logit_b0_hier-site/ppt_pptXstratum/`. You'll notice a `04-forecasting` folder containing posterior draws and model metadata — this is saved because of the `--save-forecast-inputs` flag we passed when fitting.

If you'd like to evaluate the model, see [model diagnostics]({{< ref "/docs/1-guide/c-outputs/output-dir/01-diagnostics" >}}) and [model checking]({{< ref "/docs/1-guide/c-outputs/output-dir/02-checking" >}}). You'll find that the test statistics in `mod-summary.csv` and the convergence diagnostics in `convergence-diagnostics.txt` indicate a successful fit.

## Step 3: Run the Forecast Pipeline

Now we're ready to forecast. The configuration YAML for this run is at `forecasting/mock-data-tutorial/mock-forecast-config.yaml`. The first sections are provided here with comments.

```YAML
# ==== PATHS ==================================================================

paths:
  # the directory written by the fitting pipeline
  fitted_model_dir: assets/_output/M4MD/ELDO/mock-cover/y_hits/binom_inv-logit_b0_hier-site/ppt_pptXstratum

  # the two future climate scenarios (wet, dry) generated by generate-mock-data.R
  scenarios_file: assets/_data/mock-covariate-scenarios.csv

# ==== COVARIATES =============================================================

# declare how each covariate used in the fitted model will be sourced for
# the forecast period
# we only used precipitation (ppt) and we provide it in the scenarios_file
covariates:
  ppt:
    source: provided
```

For more on the configuration options, see the [config files]({{< ref "/docs/5-forecasting/guide/config-files" >}}) section.

Similar to `analysis-pipeline.R`, `forecast-pipeline.R` points to its YAML config and can be run interactively or from the CLI.

To run the forecast pipeline interactively, set your `forecast-pipeline.R` configuration as follows.

```R
config_path <- "forecasting/mock-data-tutorial/mock-forecast-config.yaml"
```

Then source your `forecast-pipeline.R`. Alternatively, run the forecast pipeline from the CLI.

```bash
Rscript forecasting/forecast/forecast-pipeline.R \
    --config forecasting/mock-data-tutorial/mock-forecast-config.yaml
```

<!-- TODO: update forecasting path when pipeline moves -->

Forecasting computation should take under a minute on most modern machines. You'll know it's complete when the output prints ` Forecasting Complete!`

## Step 4: Reading Your Outputs

All forecast plots are written under the fitted model's output directory, inside the `04-forecast/` subfolder:

```
assets/_output/M4MD/ELDO/mock-cover/y-hits/.../04-forecast/
├── diagnostics/                  ← health-check plots
└── runs/
    ├── comparison/               ← cross-scenario plots
    ├── dry/
    │   └── Mock-GCM-1/
    │       └── forecasts/        ← per-run plots
    └── wet/
        └── Mock-GCM-1/
            └── forecasts/        ← per-run plots
```

For a complete reference of these plot, see the [forecast outputs reference]({{< ref "/docs/5-forecasting/guide/outputs" >}}).

### Scenario comparison

We'll now walk through a couple of plots to finish the tutorial. As you look at them, two things should stand out: the scenarios should visibly diverge after 2026, and stratum A should show a wider gap between them than stratum B.

Our first stop is the parkwide ensemble trajectories. This shows the park average behavior under our two future scenarios. The `dry` and `wet` labels map directly to the `scenario_name` column in `mock-covariate-scenarios.csv`.

![Park-wide ensemble trajectories under both scenarios](ensemble-trajectories-parkwide.png)

The two scenarios visibly diverge from 2026 onward: `wet` tracks upward while `dry` continues the historical downward trend. This is the expected result; the model learned a positive precipitation–vegetation relationship during training, so higher future precipitation translates to higher forecast cover.

{{< hint info >}}
If we had provided more than one model for our scenario(s), this plot would include an interval band representing inter-model variation (i.e. model uncertainty)
{{< /hint >}}

The stratum-level comparison plots tell the same story split by stratum:

<div style="display: flex; gap: 1rem;">
  <figure style="flex: 1; margin: 0;">
    <img src="ensemble-trajectories-stratum-A.png" alt="Ensemble trajectories for stratum A">
    <figcaption>Stratum A</figcaption>
  </figure>
  <figure style="flex: 1; margin: 0;">
    <img src="ensemble-trajectories-stratum-B.png" alt="Ensemble trajectories for stratum B">
    <figcaption>Stratum B</figcaption>
  </figure>
</div>

### Stratum-level response differences

Notice that the two scenarios (dry vs wet) diverge more in stratum A than in stratum B (stratum A's trajectories spread further apart over time). This is also expected. The driver–response scatter plot from training captures why:

![Driver–response relationship by stratum](driver-response-comparison.png)

Stratum A has a steeper ppt–cover slope historically — it occupies higher cover values at high precipitation and drops further when precipitation is low. Because the model fits a stratum-specific precipitation effect (via the `ppt*stratum` interaction term), it learns this stronger sensitivity and carries it into the forecast. The result is that the same change in precipitation produces a larger swing in predicted cover for stratum A than for B.

## Summary

In this tutorial you:

- Fit a binomial cover model with a precipitation covariate and a stratum-specific interaction
- Generated forecasts under two climate scenarios (dry and wet) from 2026–2040
- Interpreted ensemble trajectory plots to see how scenarios diverge from the historical trend
- Used the driver–response relationship to understand why stratum A responds more strongly than stratum B

If you're ready to apply this to your own data, the next tutorial walks through forecasting with real-world climate scenario files: [climate futures data tutorial]({{< ref "/docs/5-forecasting/tutorials/climate-futures-data" >}}).
