---
title: Mock data walkthrough
weight: 1
bookToc: true
---

# Your First Forecast (Mock Data)

## Prerequisites

A working M4MD repo. [TODO: formalize this]

## Step 0: Understand the mock data

This mock dataset represents a park unit (ELDO) with two vegetation strata (A and B), three sites per stratum, and three transects per site — surveyed annually from 2000 to 2025. Each transect records how many of 100 sample points hit a plant (`y_hits` out of `n_points = 100`). A single climate covariate, precipitation (ppt), declines gradually over the training period and drives the vegetation response.

There are three files, all pre-committed to `assets/_data/`:

| File | What it contains |
|------|-----------------|
| `pg-hits.csv` | Transect-level vegetation observations (one row per transect-year) |
| `pg-covariates.csv` | Annual precipitation per site (one row per site-year) |
| `pg-covariates-scenarios.csv` | Future precipitation under three climate scenarios × three GCM model runs |

{{< hint info >}}
You don't need to generate the data — it's already in the repo. The script that created it is at `forecasting/getting-started/generate-mock-data.R` if you want to see exactly how it was built or regenerate it.
{{< /hint >}}

### Trainingg observations (`pg-hits.csv`)

Each row is one transect in one year. The model fits to `y_hits` as a binomial outcome.

| stratum | park_unit | event_year | transect | site | n_points | y_hits |
|---------|-----------|------------|----------|------|----------|--------|
| A | ELDO | 2000 | 1 | A1 | 100 | 65 |
| A | ELDO | 2000 | 2 | A1 | 100 | 64 |
| A | ELDO | 2000 | 3 | A1 | 100 | 57 |
| A | ELDO | 2001 | 3 | A1 | 100 | 60 |
| … | … | … | … | … | … | … |

### Climate covariates (`pg-covariates.csv`)

One precipitation value (mm) per site per year — shared across all transects at that site.

| park_unit | stratum | site | event_year | ppt |
|-----------|---------|------|------------|-----|
| ELDO | A | A1 | 2000 | 476 |
| ELDO | A | A1 | 2001 | 500 |
| ELDO | A | A1 | 2002 | 518 |
| … | … | … | … | … |

### Future scenarios (`pg-covariates-scenarios.csv`)

Three climate trajectories (continued decline, flat, and increasing), each with three "GCM" model runs, covering 2026–2040.

| scenario_name | model_run_name | park_unit | stratum | site | event_year | ppt |
|---------------|----------------|-----------|---------|------|------------|-----|
| continued_decline | Mock-GCM-1 | ELDO | A | A1 | 2026 | 380 |
| continued_decline | Mock-GCM-1 | ELDO | A | A1 | 2027 | 405 |
| flat | Mock-GCM-1 | ELDO | A | A1 | 2026 | 456 |
| … | … | … | … | … | … | … |

### What the data looks like

Vegetation hit rate tracks precipitation closely, with stratum A starting at a higher baseline (~65%) than stratum B (~35%). Both strata decline as precipitation falls.

![Vegetation hit rate by stratum over time, with precipitation overlaid](/images/01-training-hits.png)

The precipitation–vegetation relationship is what the model learns and uses to generate forecasts.

![Hit rate vs. precipitation scatter plot](/images/02-hits-vs-ppt.png)

The three future scenarios bracket the range of plausible climate outcomes. Each has three GCM model runs (small offsets representing ensemble spread).

![Climate scenarios: historical and three future trajectories](/images/03-climate-scenarios.png)

## Step 1: Fit the Model

The mock training data (`assets/_data/pg-*.csv`) are accompanying configuration files are already in this repo. Let's take a look at them before fitting the pipeline.
Run the fitting pipeline to generate the model artifacts that the forecasting
pipeline will read:

```bash
Rscript analysis-pipeline.R assets/_config/M4MD/ELDO/mock-cover.yml \
  --save-forecast-inputs
```

<!-- TODO: Describe roughly how long this takes and what console output to expect.
     Note that --save-forecast-inputs is the flag that writes the 04-forecast/
     artifacts; without it the forecast pipeline has nothing to read. -->

## Step 2: Inspect the Output (Recommended)

Before forecasting, it is worth browsing the artifacts the fitting step wrote:

```
assets/_output/M4MD/ELDO/mock-cover/<run-id>/04-forecast/forecast-inputs/
```

<!-- TODO: List the key files here (forecasting-metadata.rds, mcmc-draws-full.csv,
     covariate-moments.rds) and give a one-sentence description of each so the
     user understands what the forecast pipeline will read. -->

## Step 3: Run the Forecast

```bash
Rscript forecasting/forecast/forecast-pipeline.R \
  --config forecasting/getting-started/mock-forecast-config.yaml
```

<!-- TODO: Show expected console output (step-by-step log lines). -->

## Reading Your Outputs

<!-- TODO: Point to the reading-your-outputs page once it exists, or fold that
     content here. Outputs land in assets/_output/.../04-forecast/forecasts/.
     Describe the plots and the summary CSV at a high level. -->

---

