---
title: The Math
weight: 3
---

# Forecast calculations

This page describes the calculations the forecast pipeline applies to a fitted model and a future-scenario covariate set to produce posterior predictive distributions.

**Prerequisites.** This page assumes familiarity with the model architecture described in **Zachmann et al. (2021)**, *"Bayesian Models for Analysis of Inventory and Monitoring Data with Non-ignorable Missingness"*, [JABES 26, 125–148](https://link.springer.com/article/10.1007/s13253-021-00473-z). In particular, you should understand the hierarchical structure (site within stratum within park), the role of the link function $g(\cdot)$, and the general form of Eq. 6 (outlined below).

**On this page:**

1. [Notation](#notation) — mathematical terms used on this page and their names
2. [The fitted model](#the-fitted-model) — the equation the forecast builds on
3. [What you have when a forecast starts](#what-you-have-when-a-forecast-starts) — the inputs the forecast pipeline reads
4. [The calculation](#the-calculation) — the step-by-step calculations performed by the forecast pipeline
5. [Putting it all together](#putting-it-all-together) — the forecast equation summarized

---

## Notation

Throughout this page, terms are referred to via two names: their **paper symbols** as they appear in Zachmann et al. (2021), and their **variable names** as they exist in the pipeline repo code (e.g. as they appear in `forecast-engine.R`, `mcmc-draws-full.csv`). These terms and their names are summarized here.

| Concept | Paper (Zachmann 2021) | Pipeline code |
|---|---|---|
| Observation index within site | $i$ | `n` in the JAGS fitting code (`y[n]`, `y.site[n]`, `y.strata[n]`) |
| Site index within stratum | $j$ | `site_in_stratum_index` (global form: `site_index`) |
| Stratum index | $k$ | `stratum_index` |
| Year | $t$ (paper: sequential index from year 1) | `cal_year` (raw calendar year); `x_time` (standardized using fit-time moments) |
| Posterior draw index | $m$ (superscript) | `iter` / `draw` |
| Total number of posterior draws | $M$ | `n_draws` |
| Site intercept | $\alpha_{0jk}$ | `B0[j,k]` (`B[j,1,k]` in JAGS) |
| Time trend coefficient | $\alpha_{1jk}$ (site-level) or $\alpha_{1k}$ (stratum-level) | `B1[j,k]` or `B1[k]` in the draws CSV |
| Covariate row for $(j,k,t)$ | $w_{jkt}$ | row of `fe_*` columns, ordered to match `Beta` |
| Fixed-effect coefficients | $\beta$ (or $\beta_k$) | `Beta[m]` or `Beta.hat[m]` |
| Dispersion parameter | $\sigma^2_{jk}$ (or $\sigma^2_k$) | `sigma[k]` |
| Inverse link function | $g(\cdot)$ | `inv_link(·)` / `get_inverse_link_function()` |
| Response distribution (moment-matched) | $h(\mu, \sigma^2)$ | per-family sampler in `sample_predictive()` |
| Linear predictor | argument of $g$ (paper has no symbol for it) | `eta` |
| Mean prediction | $\mu_{jkt}$ | `mu` |
| One predictive draw | $\hat y_{jkt}^{(m)}$ (paper Eq. 10) | `y_rep` |
| Training-data mean of covariate $p$ | $\bar w_p$ | `scaled:center` |
| Training-data SD of covariate $p$ | $s_{w_p}$ | `scaled:scale` |

---

## The fitted model

The fitted model follows the hierarchical generalized linear model of Zachmann et al. (2021), Eq. 6:

$$\mu_{jkt} \;=\; g\!\bigl(\alpha_{0jk} \;+\; \alpha_{1jk}\,t \;+\; w_{jkt}\,\beta\bigr)$$

with response distribution $y_{ijkt} \mid \mu_{jkt}, \sigma^2_{jk} \sim h(\mu_{jkt}, \sigma^2_{jk})$.

The forecast pipeline takes posterior draws from a fitted model and applies this equation forward in time using new covariates. The rest of this section explains the terms of Eq. 6.

### $\mu_{jkt}$ — the mean response

The expected value of the response at site $j$, stratum $k$, year $t$, on the response scale (e.g. proportion cover on $(0,1)$ for Beta, expected count on $(0, \infty)$ for Poisson). This is what the forecast pipeline ultimately produces: one $\mu_{jkt}^{(m)}$ per posterior draw $m$ (the $\mu_{jkt}^{(m)}$ are then summarized across draws into forecast bands).

### $g(\cdot)$ — the inverse link function

The inverse link maps the unconstrained linear-predictor scale (the argument of $g$, which we call $\eta$ in the code) back to the response scale. We need a link because the linear predictor can take any real value, but $y$ is constrained (e.g. $y \in (0,1)$ for proportions). Supported inverse links: `inverse-logit`, `exponential`, `linear`, `restricted-linear` (= $\max(\eta, 10^{-5})$), `zero-trick` (= $\exp(\eta)$). The link is paired with the likelihood family at fit time and stored as the `link` field in `forecasting-metadata.rds`.

### $\alpha_{0jk}$ — the site-specific intercept

Each (site, stratum) pair gets its own baseline, capturing the average response level at that site after accounting for time and covariates. In the code this is `B0[j,k]` (in the JAGS array notation, `B[j, 1, k]`). The posterior draws of $\alpha_{0jk}$ are saved in `mcmc-draws-full.csv` as columns named `B0[j,k]` and are read directly by the forecast pipeline.

### $\alpha_{1jk}$ — the trend slope

The temporal coefficient that captures linear change in the response over time. The user decides during model fitting whether to include $\alpha_{1jk}$ and at what level it is indexed. The forecast pipeline supports three "levels" of $\alpha_{1jk}$, encoded in the `b1_structure` field of `forecasting-metadata.rds`:

- **`b1_structure = "site"`** — every (site, stratum) pair gets its own slope $\alpha_{1jk}$. This is the full Eq. 6 form. Posterior draws are saved as `B1[j,k]` (or `B[j, 2, k]` in JAGS array notation).
- **`b1_structure = "stratum"`** — one slope $\alpha_{1k}$ shared across all sites within stratum $k$. This is the paper's "fixed slope within stratum" variant (Zachmann et al. 2021, Sect. 4.1.1). Posterior draws are saved as `B1[k]`.
- **`b1_structure = "none"`** — no trend term is included; the time dimension is captured only through covariates in $w$ (e.g. a precipitation series that varies over time). This corresponds to a "time effect: disabled" fit.

The forecast pipeline reads the appropriate `B1` columns from the draws CSV depending on which level the user chose during model fitting. See [Fitting a model for forecasting]({{< ref "/docs/5-forecasting/guide/fitting-a-model.md" >}}) and [Group-level effects]({{< ref "/docs/1-guide/b-config-files/iii-analysis-model/group-level-effects.md" >}}) for more context on this.

### $t$ — the time unit

The paper uses a sequential year index ($t = 1, 2, \ldots, T$ where $T$ is the number of monitoring years), while the pipeline stores years as calendar years in the year column of the scenarios CSV. To keep the trend coefficient $\alpha_{1jk}$ on the right scale at forecast time, the pipeline centers and scales $t$ using the same `time_info` that JAGS saw during fitting (`ref_year`, `rel_year_center`, `rel_year_scale`, which are stored in `forecasting-metadata.rds`). The standardized year is called `x_time` in the code. See [Step 2 of the calculation](#step-2--compute-the-linear-predictor-eta_jktm) for more details.

### $w_{jkt}$ — the additional covariate row

The vector of additional predictors for $(j, k, t)$. These are covariates such as climate drivers, site or stratum attributes, and interaction terms. In the paper, $w_{jkt}$ is a row vector. In the forecasting pipeline, you supply the raw covariate values in the scenarios CSV (one row per $(j, k, t)$), or specify them as constant in the forecasting config YAML. The pipeline standardizes them and constructs any interaction columns.

### $\beta$ — the fixed-effect coefficients

The regression coefficients on the covariate row $w_{jkt}$. The paper describes three levels of indexing depending on the modeling choice: park-level ($\beta$, shared across all strata), stratum-level ($\beta_k$), or site-level ($\beta_{jk}$). In code, $\beta$ is stored as a flat vector `Beta[m]` (or `Beta.hat[m]`, the centered version) in `mcmc-draws-full.csv`, with no $k$ subscript. Stratum-level variation in coefficients is captured by additional entries in the same vector.

### The response distribution: $y_{ijkt} \mid \mu_{jkt}, \sigma^2_{jk} \sim h(\mu_{jkt}, \sigma^2_{jk})$

Given the mean $\mu_{jkt}$ and a dispersion parameter, observations are drawn from a distribution $h$ chosen to match the support of $y$ (Beta on $(0,1)$, Poisson/Negative-Binomial on $\mathbb{N}_0$, Lognormal on $(0, \infty)$, etc). The pipeline uses moment matching, meaning each distribution's native parameters are chosen so its mean and dispersion match the supplied values.

The dispersion parameter is stratum-specific in this model (e.g. $\sigma^2_k$, not $\sigma^2_{jk}$). Dispersion is used during a forecast at **Step 4**, when we sample posterior predictive draws.

---

## What you have when a forecast starts

The fitting pipeline (run with `--save-forecast-inputs`) writes the following artifacts:

- **Posterior draws** — `mcmc-draws-full.csv`. One row per MCMC iteration $m$, with columns:

    | Column(s) | Paper symbol | What it represents |
    |---|---|---|
    | `iter` | $m$ | MCMC iteration index |
    | `B0[j,k]` | $\alpha_{0jk}^{(m)}$ | Site-specific intercept for site $j$ in stratum $k$ |
    | `B1[j,k]` or `B1[k]` | $\alpha_{1jk}^{(m)}$ or $\alpha_{1k}^{(m)}$ | Trend slope (site- or stratum-level; omitted when `b1_structure = "none"`) |
    | `Beta[...]` (or `Beta.hat[...]`) | $\beta^{(m)}$ | Fixed-effect coefficients, one column per element of the covariate row |
    | `sigma[k]`, `kappa[k]`, or `delta[k]` | $\sigma_k^{(m)}$, $\kappa_k^{(m)}$, $\delta_k^{(m)}$ | Stratum-specific dispersion parameter based on the likelihood |

- **Training covariate moments** — `covariate-moments.rds`. Per base covariate $p$: the training-data mean $\bar w_p$ and standard deviation $s_{w_p}$ used to standardize future values onto the same scale the model was fit on.
- **Model spec metadata** — `forecasting-metadata.rds`. Likelihood family, link function, additional-covariates formula string, column-name mappings, `b1_structure`, and `time_info` (the year centering/scaling reference).
- **Site and stratum lookups** — `site-in-stratum-info.csv` and `stratum-ids-and-indices.csv`. Map between human-readable site/stratum IDs and the integer indices $j, k$ used in the draws.
- **Future scenarios** — a user-supplied CSV. One row per (site, stratum, year, scenario, model_run).

---

## The calculation

[The fitted model](#the-fitted-model) describes Eq. 6 and [What you have when a forecast starts](#what-you-have-when-a-forecast-starts) lists the forecasting pipeline inputs. Building on these, this section describes the calculations performed by the pipeline.

For each posterior draw $m \in \{1, \ldots, M\}$ and each scenario row $(j, k, t)$, the pipeline performs five steps:

### Step 1 — Build the covariate row $w_{jkt}$

Raw values are sourced, standardized, and combined into interaction columns.

**(a) Source the raw value of each base covariate $p$.** The forecast config declares one of three strategies per covariate:

- `provided` — the raw value $w_{jkt,p}^{\text{raw}}$ is read directly from the scenarios CSV.
- `hold_last` — the raw value is the last training-period observation of covariate $p$ at site $j$.
- `hold_mean` — the raw value is the training-period mean of covariate $p$ at site $j$.

**(b) Standardize using training moments.** Each base covariate is centered and scaled with the moments saved in `covariate-moments.rds`:

$$w_{jkt,p} \;=\; \frac{w_{jkt,p}^{\text{raw}} \;-\; \bar w_p}{s_{w_p}}$$

This puts future values on the same scale that $\beta$ was estimated on.

**(c) Reconstruct interaction columns.** Two scenarios:

- **Continuous × stratum** (e.g. `ppt * Master_Stratification`): the standardized covariate times an indicator for each stratum level — one column per stratum.
- **Continuous × continuous** (e.g. `ppt * tmax`): the product of the two standardized covariates.

The resulting columns are ordered to align with $\beta$.

**In code:** `prepare_covariate_matrix()` (`forecast-engine.R:236–356`).

### Step 2 — Compute the linear predictor $\eta_{jkt}^{(m)}$

When the trend term is present (i.e. `b1_structure` is `"site"` or `"stratum"`), it needs a standardized year. The pipeline applies the same centering and scaling JAGS used during fitting (`time_info` in `forecasting-metadata.rds`), so the saved $\alpha_1$ posterior is interpretable on this scale. The result is called $x_{\text{time}}(t)$ (`x_time` in the code).

For each draw $m$, sum the site intercept, the trend term, and the covariate dot product:

$$\eta_{jkt}^{(m)} \;=\; \alpha_{0jk}^{(m)} \;+\; b_{\text{trend}}^{(m)}(j,k,t) \;+\; w_{jkt}\,\beta^{(m)}$$

The trend term depends on `b1_structure`:

$$b_{\text{trend}}^{(m)}(j,k,t) \;=\;
\begin{cases}
\alpha_{1jk}^{(m)} \cdot x_{\text{time}}(t) & \text{if } b_1\text{-structure} = \texttt{site} \\[4pt]
\alpha_{1k}^{(m)} \cdot x_{\text{time}}(t) & \text{if } b_1\text{-structure} = \texttt{stratum} \\[4pt]
0 & \text{if } b_1\text{-structure} = \texttt{none}
\end{cases}$$

Where each term comes from:

- $\alpha_{0jk}^{(m)}$ — the `B0[...]` columns of draw $m$.
- $b_{\text{trend}}^{(m)}(j,k,t)$ — the `B1[...]` columns of draw $m$ (site- or stratum-indexed depending on `b1_structure`).
- $w_{jkt}$ — the row built in Step 1 above.
- $\beta^{(m)}$ — the `Beta[...]` (or `Beta.hat[...]`) columns of draw $m$, in the same column order as $w_{jkt}$.

**In code:** `x_time` construction at `forecast-engine.R:569–571`; `eta <- fg$B0 + b1_term + drop(X_mat %*% beta_vec)` in `compute_draw()` at `forecast-engine.R:674`.

### Step 3 — Apply the inverse link to get $\mu_{jkt}^{(m)}$

Transform $\eta$ back to the response scale:

$$\mu_{jkt}^{(m)} \;=\; g\!\bigl(\eta_{jkt}^{(m)}\bigr)$$

See [$g(\cdot)$ — the inverse link function](#gcdot--the-inverse-link-function) above for the list of supported links.

**In code:** `mu <- inv_link(eta)` at `forecast-engine.R:675`; `get_inverse_link_function()` at `forecast-likelihoods.R:14–39`.

### Step 4 — Sample one posterior predictive draw $\hat y_{jkt}^{(m)}$

For each $(m, j, k, t)$ tuple, draw one realization $\hat y_{jkt}^{(m)}$ from the response distribution, parameterized by $\mu_{jkt}^{(m)}$ and the stratum-specific dispersion parameter from draw $m$. This conceptually mirrors the operation the JAGS model performs in its `y.rep[n]` block during fitting; for forecasting we apply it to scenario rows rather than training observations.

| Family | Dispersion param. | Predictive sampler |
|---|---|---|
| Beta | $\sigma_k^{(m)}$ | $\phi = \mu(1-\mu)/\sigma^2 - 1$; $\hat y \sim \text{Beta}(\mu\phi,\,(1-\mu)\phi)$ |
| Binomial | $\sigma_k^{(m)}$ (logit-scale SD) | $\eta_{\text{new}} \sim \mathcal{N}(\text{logit}(\mu),\,\sigma^2)$; $\hat y = \text{logit}^{-1}(\eta_{\text{new}})$ |
| Poisson | — | $\hat y \sim \text{Poisson}(\mu)$ |
| Negative-Binomial | $\sigma_k^{(m)}$ | $\kappa = \mu^2 / \max(\sigma^2 - \mu,\, 10^{-12})$; $\hat y \sim \text{NegBin}(\text{size} = \kappa,\,\mu)$ |
| NegBin-simple | $\kappa_k^{(m)}$ | $\hat y \sim \text{NegBin}(\text{size} = \kappa,\,\mu)$ |
| Lognormal | $\sigma_k^{(m)}$ | $\text{CV}^2 = (\sigma/\mu)^2$; $s_{\log} = \sqrt{\log(1 + \text{CV}^2)}$; $m_{\log} = \log\mu - s_{\log}^2/2$; $\hat y \sim \text{Lognormal}(m_{\log},\,s_{\log})$ |
| Generalized Poisson | $\delta_k^{(m)}$ | $\lambda = (1 - \delta)\mu$; inverse-CDF sampling from the GP PMF |

At the end of Step 4, every $(m, j, k, t)$ tuple has two associated quantities:

- $\mu_{jkt}^{(m)}$ — the deterministic mean, which captures parameter uncertainty.
- $\hat y_{jkt}^{(m)}$ — one draw from the response distribution, which captures parameter + observation-scale variability uncertainty.

**In code:** `sample_predictive()` (dispatch) at `forecast-likelihoods.R:303–331`; per-family samplers `sample_beta_predictive`, `sample_binomial_predictive`, etc., at `forecast-likelihoods.R:51–289`.

### Step 5 — Summarize across draws

Collapse the $M$ draws into summary statistics at each scale (site, stratum, park) and each year $t$:

- **Posterior summaries of the mean**: `mu_mean`, `mu_median`, `mu_lower`, `mu_upper` computed from $\{\mu_{jkt}^{(1)}, \ldots, \mu_{jkt}^{(M)}\}$.
- **Posterior predictive intervals**: `y_rep_mean`, `y_rep_lower`, `y_rep_upper` computed from $\{\hat y_{jkt}^{(1)}, \ldots, \hat y_{jkt}^{(M)}\}$.
- **Ensembles** across multiple climate `model_run`s: run-level `mu_median` is collapsed across runs into `ensemble_mean`, `ensemble_median`, `ensemble_lower`, `ensemble_upper`. `y_rep` is intentionally **not** collapsed across model runs.

The `*_lower` and `*_upper` columns use user-configurable quantiles set via the summary quantiles field in the forecast config YAML (see [Config files]({{< ref "/docs/5-forecasting/guide/config-files.md" >}})).

**In code:** `summarize_forecasts()` (`forecast-engine.R:763–822`); `summarize_ensemble()` (`forecast-engine.R:840–885`).

---

## Putting it all together

For each draw $m$ and scenario row $(j, k, t)$, the forecast pipeline computes:

$$\hat y_{jkt}^{(m)} \;\sim\; h\!\Bigl(\,g\!\bigl(\alpha_{0jk}^{(m)} \;+\; b_{\text{trend}}^{(m)}(j,k,t) \;+\; w_{jkt}\,\beta^{(m)}\bigr),\;\; \sigma_k^{(m)}\Bigr)$$

Summarizing the resulting $M$-sample distribution at each $(j, k, t)$ produces the forecast bands you see in the output plots.
