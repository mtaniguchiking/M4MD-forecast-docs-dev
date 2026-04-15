---
title: Pipeline Walkthrough
weight: 2
---

**\*WORK IN PROGRESS (this is empty for now)***

# Pipeline Walkthrough: `forecast-pipeline.R`

<!-- TODO: Intro paragraph — this page walks through all steps of
     forecast-pipeline.R, annotating what each step does and which
     functions it calls. Cross-reference with forecast-engine.R and
     related source files.

     Source file: forecasting/forecast/forecast-pipeline.R
-->

## Step 1: Parse and Validate Config

<!-- TODO: The pipeline reads the YAML config via read-forecast-config.R.
     Describe what is validated at this stage and what errors to expect
     if required fields are missing or malformed.
     Source: forecasting/forecast/read-forecast-config.R
-->

## Step 2: Load Model Metadata

<!-- TODO: The pipeline reads forecasting-metadata.rds from the 04-forecast/
     output directory. Describe what is in this file and why the forecast
     pipeline needs it (model spec, covariate info, site/stratum structure).
     Source: forecast-engine.R:load_forecasting_metadata()
-->

## Step 3: Load MCMC Draws

<!-- TODO: Reads mcmc-draws-full.csv. These are the posterior samples that
     will be used for the predictive draws. Describe structure (rows = draws,
     cols = parameters).
-->

## Step 4: Load Covariate Moments

<!-- TODO: Reads covariate-moments.rds — the training-data mean and SD for
     each covariate. Used in Step 6 to standardize the forecast covariates
     using the same transformation applied to the training data.
     Source: forecast-engine.R:load_covariate_moments()
-->

## Step 5: Load Scenario Data

<!-- TODO: Reads the user-supplied scenarios CSV. Describe the expected
     column schema and how scenario_id / model_run_id are used to group runs.
-->

## Step 6: Prepare the Covariate Matrix

<!-- TODO: For each scenario × model_run combination, builds the design matrix X
     that corresponds to the future conditions. Key operations:
     - Applies covariate strategies (provided / hold_last / hold_mean)
     - Standardizes covariates using training moments from Step 4
     - Reconstructs interaction terms (e.g., ppt × stratum columns)
     Source: forecast-engine.R:prepare_covariate_matrix()
-->

## Step 7: Reconstruct the Design Matrix

<!-- TODO: Describe how the fixed-effects design matrix columns are matched
     back to the parameter names in the MCMC draws. The metadata drives this
     so that the forecast pipeline doesn't need the original model config.
     Source: forecast-engine.R:build_fe_lookup()
-->

## Step 8: Sample Posterior Predictive Draws

<!-- TODO: For each MCMC draw s and each future time point:
     1. Compute linear predictor η_s using posterior parameter draw
     2. Apply inverse link function (logit → probability, log → count, etc.)
     3. Sample y_new from the appropriate likelihood
     Result: a matrix of draws representing the posterior predictive distribution.
     Source: forecast-engine.R:compute_forecast_draws(),
             forecast-likelihoods.R
-->

## Step 9: Summarize Across Draws

<!-- TODO: Aggregate the draw matrix to point estimates + credible intervals:
     - mu_median, mu_lower, mu_upper (mean response)
     - y_rep_median, y_rep_lower, y_rep_upper (observation-level)
     Source: forecast-engine.R:summarize_ensemble()
-->

## Step 10: Summarize Across Model Runs (Ensemble)

<!-- TODO: Pool across GCM model runs within a scenario to form an ensemble.
     Describe how model runs are combined (e.g., equal weighting).
     Compare across scenarios for decision support.
-->

## Step 11: Write Outputs and Plots

<!-- TODO: Describe the output files written:
     - forecast-site-summaries.csv
     - Trajectory plots (one per site or stratum?)
     - Spaghetti plots
     - Cross-scenario comparison plots
     Source: forecasting/forecast/forecast-plot-engine.R
-->
