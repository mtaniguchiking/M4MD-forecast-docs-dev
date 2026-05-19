---
title: Model checking
weight: 3
bookToc: true
---

# Model checking and evaluation
We recommend checking model convergence before doing anything else. If you are evaluating a large number of models for a given state variable, it can be helpful to set override_dqs to TRUE near the top of the analysis-pipeline.R script. This will override (remove) the derived quantities and turns off most of the outputs, which can save you time and space during initial evaluation. 

## Failure to converge
If a model is failing to converge, it can be for all sorts of reasons. Sometimes there simply weren't enough MCMC iterations, or your inits were out of left field. If convergence issues affect specific parameters, it can be helpful to play around with different model parameterizations. Sometimes there's not enough trend of any sort to permit a random intercept / random slopes (`b0-b1`) model, for instance. (This can be verified by plotting the empirical means at each site over time, for example.) If you're seeing extreme "funnel" shapes in the funnel diagnostics, it may be helpful to try a non-centered parameterization. In yet other cases, it might be the prior(s). You can check if it's the priors by rerunning specific models ("model.jags" with modified priors) using the output directory and associated "calling-script.R".

## Poor Bayesian p values
If a Baysian p value obtained from a posterior predictive check is close to either 0 or 1 (e.g. outside of 0.2-0.8), then this is a sign that your model was not able to achieve a good fit to the data. Some common causes of this include unmodeled variance (either via covariates, random effects, or accommodating overdispersion), or a poor match between the chose likelihood distribution and the actual distribution of the process. For example, if your data were zero-inflated, but your chosen likelihood didn't accomodate this, then your model will likely produce data with a positive bias, resulting in a Bayesian p value for the mean of the data being closer to 0. Alternatively, if your data are overdispersed but you chose a Poisson distribution for the likelihood, then your model predictions for variance will be biased low, resulting in a Bayesian p value for the variance being closer to 1. Whenever a posterior predictive check fails, it is good practice to revisit the distributional assumptions about both the data _and_ the process models you used.

## Model summary comparison
The output directory for each model contains a mod-summary.csv file that includes test statistics, posterior predictive loss, deviance information criterion, and Gelman diagnostic. If you have run multiple models for a given state variable, comparing the model summaries can be useful for model selection. The get_mod_summary_files() file will summarize the mod-summary.csv files for a particular output directory and save a mod-summaries.csv file in the output root folder. Here is a real-world example of how to call the function for analyses of bareground cover at Fort Bowie National Historic Site (FOBO) in Arizona - a park within the Sonoran Desert Network (SODN).

```sh
get_mod_summary_files('C:/M4MD_SWNC/output/SODN/FOBO/pi-soil-binom/exposed_bareground') 
```

## Parameter estimates
When evaluating a model, sometimes it's helpful to get a quick sense of the parameter estimates. The mean, standard deviation, 95% credible interval, Gelman daignostic, effective sample size, and _f_ value for each parameter (park-level) are summarized in the coefficient-summary.csv file within ./<output root>/03-inference. 