---
title: Climate futures tutorial
weight: 2
bookToc: true
---

# Forecasting with Real-World Data

<!-- TODO: Intro paragraph — this path is for users who have their own NPS
     monitoring data and want to run a real forecast. It has a moderate lift:
     the user provides a response variable CSV, and an external notebooks repo
     handles covariate and scenario data preparation.
-->

## What You'll Need

<!-- TODO: Checklist — same R/JAGS prereqs as mock walkthrough, plus:
     - your own response variable CSV (format TBD — link to column schema)
     - access to the external notebooks repo (link TBD)
-->

## Step 1: Prepare Your Covariate and Scenario Data

<!-- TODO: Point to the external notebooks repo (URL TBD). Explain:
     - User provides their response variable CSV
     - The notebooks produce a formatted covariates CSV and a scenarios CSV
     - The output format matches what analysis-pipeline.R and
       forecast-pipeline.R expect
-->

## Step 2: Fit the Model

Once your data files are ready, follow the same fit step as the mock walkthrough:

```bash
Rscript analysis-pipeline.R path/to/your-config.yml --save-forecast-inputs
```

<!-- TODO: Describe how to set up a config YAML for the user's own data.
     Link to the config reference (section 1.3) once it exists. -->

## Step 3: Run the Forecast

```bash
Rscript forecasting/forecast/forecast-pipeline.R \
  --config path/to/your-forecast-config.yaml
```

<!-- TODO: Note the key differences in the forecast config for real data vs mock:
     which fields must be changed, which scenarios file to point to, etc. -->

## Reading Your Outputs

<!-- TODO: Same as mock walkthrough — point to shared outputs guide. -->
