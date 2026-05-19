---
title: Config files
weight: 3
---

# Config files

The forecast config is a YAML file that tells the pipeline where to find your data, how to source covariates, and how to format outputs.

A minimal example:

```yaml
# ==== PATHS =======================================================

paths:
  fitted_model_dir: path/to/model/output
  scenarios_file: path/to/future-scenario-data.csv

column mappings:
  scenario: scenario_column_name
  model_run: model_column_name

# ==== COVARIATES   ================================================

covariates:
  covariate1:
    source: provided
  covariate2:
    source: hold_mean

# ==== COMPUTATION =================================================

random seed: 42
max draws: 2000

# ==== OUTPUT ======================================================

summary levels:
  - site
  - stratum

summary quantiles:
  - 0.05
  - 0.500
  - 0.95

```

The sections below describe each field.

---

## `paths`

*Required*

```yaml
paths:
  fitted_model_dir: path/to/model/output
  scenarios_file: path/to/future-scenario-data.csv
```

| Field | Required | Description |
|---|---|---|
| `fitted_model_dir` | Yes | Path to the fitted model output directory. Must contain `04-forecast/forecast-inputs/`, which is written by the fitting pipeline when run with `--save-forecast-inputs`. |
| `scenarios_file` | Yes | Path to your scenarios CSV. See [Input data]({{< ref "/docs/5-forecasting/guide/input-data" >}})

---

## `column mappings`

*Optional*

```yaml
column mappings:
  scenario: scenario_column_name
  model_run: model_column_name
```

The default names for the scenario and model identifier columns are `scenario` and `model_run`, respectively. If your scenarios CSV uses different column names than these, declare the mapping here.

| Field | Description |
|---|---|
| `scenario` | Name of the column in your CSV that corresponds to `scenario` (e.g., `ssp`, `climate_scenario`) |
| `model_run` | Name of the column in your CSV that corresponds to `model_run` (e.g., `gcm_name`, `ensemble_member`) |

> Site, stratum, and year column aliases (e.g., `Plot_ID`, `Master_Stratification`, `Visit_Year`) are declared in the fitted model's metadata, not here.

---

## `covariates`

*Required unless the fitted model has no fixed effects.*

Declare **every base covariate** used in the fitted model and specify how its future values will be sourced. Interaction terms (e.g. `covariate*stratum`) are handled automatically — do not list them here.

```yaml
covariates:
  ppt:
    source: provided
  tmax:
    source: hold_mean
```

Each named covariate takes a `source` field with one of three strategies:

### `provided`

This means future covariate values come from the scenarios CSV. Use this for covariates where you have actual projections (e.g. downscaled climate model output).

The covariate name must match a column in your scenarios CSV.

### `hold_last`

This means the covariate is held at the last observed value from training data, separately for each site. Use this for covariates where no projection is available and a site-specific anchor is reasonable.

### `hold_mean`

The covariate is held at the training-data mean, separately for each site. Use this when you want a climatological baseline rather than a site-specific last observation.

> Covariates using `hold_last` or `hold_mean` do not require a column in the scenarios CSV.

---

## Computation

```yaml
random seed: 42
max draws: 2000
```

| Field | Required | Default | Description |
|---|---|---|---|
| `random seed` | No | — | Integer seed for reproducibility. Omit to use a random seed. |
| `max draws` | No | All draws | Number of posterior draws to use. More draws produce smoother uncertainty estimates at the cost of runtime. |

> If `max draws` is not set by the user, it will use all posterior draws (i.e. the value `n_iter` was assign to during model fitting).

---

## Output

```yaml
summary levels:
  - site
  - stratum

summary quantiles:
  - 0.05
  - 0.500
  - 0.95

output prefix: forecast
```

| Field | Required | Description |
|---|---|---|
| `summary levels` | Yes | Spatial aggregation levels to summarize. Valid values: `site`, `stratum`, `park`. Include one or more. |
| `summary quantiles` | Yes | Quantiles for the credible interval. Provide three values: lower bound, median, upper bound. The example above produces a 90% credible interval. |
| `output prefix` | No | Prefix for output CSV filenames (default: `"forecast"`). For example, `forecast-site-summaries.csv`. |
