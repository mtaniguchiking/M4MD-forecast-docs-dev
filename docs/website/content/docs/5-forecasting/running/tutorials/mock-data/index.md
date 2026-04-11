---
title: Mock data walkthrough
weight: 1
bookToc: true
---

# Your First Forecast (Mock Data)

This tutorial walks through a simple forecast run using synthetic data. By the end you will have fit a model, used this model to forecast under two climate scenarios, and explored the forecast's outputs.

## Prerequisites

See [Getting started]({{< ref "/docs/0-getting-started/installation.md" >}}) if you haven't already.

## Step 0: Understand the mock data

This mock dataset represents a park unit (ELDO) with two vegetation strata (A and B), six sites per stratum, and one transect per site. The data is annual from 2000 to 2025. Each transect records how many of 100 sample points hit a plant (`y_hits` out of `n_points = 100`). A single climate covariate, precipitation (ppt), declines gradually over the training period and drives the vegetation response.

There are three files, all pre-committed to `assets/_data/`:

| File | What it contains |
|------|-----------------|
| `pg-hits.csv` | Transect-level vegetation observations (one row per transect-year) |
| `pg-covariates.csv` | Annual precipitation per site (one row per site-year) |
| `pg-covariates-scenarios.csv` | Future precipitation under two climate scenarios |

{{< hint info >}}
You don't need to generate the data — it's already in the repo. The script that created it is at `forecasting/getting-started/generate-mock-data.R` if you want to play around with generating alternative mock data.
{{< /hint >}}

### Training observations (`pg-hits.csv`)

Each row is one transect in one year. The model fits to `y_hits` as a binomial outcome.

| stratum | park_unit | event_year | transect | site | n_points | y_hits |
|---------|-----------|------------|----------|------|----------|--------|
| A | ELDO | 2000 | 1 | A1 | 100 | 94 |
| A | ELDO | 2001 | 1 | A1 | 100 | 62 |
| A | ELDO | 2002 | 1 | A1 | 100 | 72 |
| … | … | … | … | … | … | … |

### Climate covariates (`pg-covariates.csv`)

One precipitation value (mm) per site per year.

| park_unit | stratum | site | event_year | ppt |
|-----------|---------|------|------------|-----|
| ELDO | A | A1 | 2000 | 534 |
| ELDO | A | A1 | 2001 | 480 |
| ELDO | A | A1 | 2002 | 497 |
| … | … | … | … | … |

### Future scenarios (`pg-covariates-scenarios.csv`)

Two climate trajectories (dry and wet), each with one "GCM" model run covering 2026–2040.

| scenario_id | scenario_name | model_run_id | model_run_name | park_unit | stratum | site | event_year | ppt |
|-------------|---------------|--------------|----------------|-----------|---------|------|------------|-----|
| 1 | dry | 1 | Mock-GCM-1 | ELDO | A | A1 | 2026 | 384 |
| 1 | dry | 1 | Mock-GCM-1 | ELDO | A | A1 | 2027 | 330 |
| 1 | dry | 1 | Mock-GCM-1 | ELDO | A | A1 | 2028 | 319 |
| … | … | … | … | … | … | … | … |

### What the data looks like

Vegetation hit rate tracks precipitation closely, with stratum A starting at a higher baseline (~75%) than stratum B (~45%). Both strata decline as precipitation falls.

![Vegetation hit rate by stratum over time, with precipitation overlaid](01-stratum-hit-rates.png)

The precipitation–vegetation relationship is what the model learns and uses to generate forecasts.

<div style="text-align: center;">
<img src="02-hits-vs-ppt.png" alt="Hit rate vs. precipitation scatter plot" width="500">
</div>

The future scenarios are represented by 1 GCM model each, covering two plausible climate futures.

![Climate scenarios: historical and three future trajectories](03-climate-scenarios.png)

## Step 1: Fit the Model

Now that we understand the data, we can perform the model fitting. See [fitting a model for forecasting]({{< ref "/docs/5-forecasting/running/tutorials/climate-futures-data" >}}) for more details on this step. For our mock model, run:

```sh
Rscript analysis-pipeline.R assets/_config/M4MD/ELDO/mock-cover.yml \
  --save-forecast-inputs
```

Two key configurations in the YAML:

```yaml
# this allows us to capture the stratum-specific response to precipitation
additional covariates:
  - ppt, ppt*stratum

... # other configs

# this disables the linear time term in the model so our forecasts are
# driven by precipitation only
time effect: disabled
```

## Step 2: Inspect the Output (optional)

Before forecasting, it's worth checking the model output at `assets/_output/M4MD/ELDO/mock-cover/y-hits/binom_inv-logit_b0_hier-site/ppt_pptXstratum/`. You'll notice a `04-forecasting` folder containing posterior draws and model metadata — this is saved because of the `--save-forecast-inputs` flag we passed when fitting.

If you'd like to evaluate the model, see [model diagnostics]({{< ref "/docs/1-guide/c-outputs/output-dir/01-diagnostics" >}}) and [model checking]({{< ref "/docs/1-guide/c-outputs/output-dir/02-checking" >}}). For example, the test statistics in `mod-summary.csv` and the convergence diagnostics in `convergence-diagnostics.txt` indicate a successful fit.

## Step 3: Run the Forecast

Now we're ready to forecast. The config for this run is at `forecasting/getting-started/mock-forecast-config.yaml`. For more on the configuration options, see the [config files]({{< ref "/docs/5-forecasting/running/guide/config-files" >}}) section.

```bash
Rscript forecasting/forecast/forecast-pipeline.R \
  --config forecasting/getting-started/mock-forecast-config.yaml
```

<!-- TODO: update forecasting path when pipeline moves -->

## Reading Your Outputs

All forecast plots are written under the fitted model's output directory, inside a `04-forecast/` subfolder:

```
assets/_output/M4MD/ELDO/mock-cover/y-hits/.../04-forecast/
├── diagnostics/
│   └── diagnostics/              ← health-check plots
└── runs/
    ├── comparison/
    │   └── comparison/           ← cross-scenario plots
    ├── dry/
    │   └── Mock-GCM-1/
    │       └── forecasts/        ← per-run plots
    └── wet/
        └── Mock-GCM-1/
            └── forecasts/        ← per-run plots
```

For a complete reference of every plot — what each panel shows, how to interpret uncertainty bands, and what to look for — see the [forecast outputs reference]({{< ref "/docs/5-forecasting/running/guide/outputs" >}}).

### Scenario comparison

We'll now investiate a couple specific plots the finish the tutorial.

Our first stop is the parkwide ensemble trajectories. This shows us what our forecast expects the park average behavior under our two future scenarios to look like.

![Park-wide ensemble trajectories under both scenarios](ensemble-trajectories-parkwide.png)

The two scenarios visibly diverge from 2026 onward: `wet` tracks upward while `dry` continues the historical downward trend. This is the expected result — the model learned a positive precipitation–vegetation relationship during training, so higher future precipitation translates to higher forecast cover.

The stratum-level comparison plots tell the same story split by stratum:

![Ensemble trajectories for stratum A](ensemble-trajectories-stratum-A.png)

![Ensemble trajectories for stratum B](ensemble-trajectories-stratum-B.png)

### Stratum-level response differences

Stratum A shows a wider gap between the two scenarios than stratum B. This is also expected. The driver–response scatter plot from training captures why:

![Driver–response relationship by stratum](driver-response-relationship.png)

Stratum A has a steeper ppt–cover slope historically — it occupies higher cover values at high precipitation and drops further when precipitation is low. Because the model fits a stratum-specific precipitation effect (via the `ppt*stratum` interaction term), it learns this stronger sensitivity and carries it into the forecast. The result is that the same change in precipitation produces a larger swing in predicted cover for stratum A than for B.

The trend summary bar chart makes this concrete: the slope estimates for stratum A are more extreme in both directions (more negative under `dry`, more positive under `wet`) than those for stratum B.

![Forecast trend summary by stratum and scenario](forecast-trend-summary.png)

If you feel comfortable with this mock forecast, feel free to check out the [next tutorial]({{< ref "/docs/5-forecasting/running/tutorials/climate-futures-data" >}}).
