---
title: Troubleshooting
bookCollapseSection: false
weight: 5
---

# Troubleshooting tips
Like many things in R, __models-for-missing-data__ can give cryptic error messages. If you need to do some troubleshooting, a good place to start is by stepping through the rowwise_fit() function. The rowwise_fit() function is defined in model-api\src\fit.r. Note that there are many other functions called by the pipeline and nested within the scripts. If you are trying to figure out where a function, variable, or output is defined, it can be helpful to search the M4MD repo on GitHub. 

If you want to step through rowwise_fit() for a given model, you should first step through the analysis-pipeline.R through line 122, including selecting your model with this_slice. This should create the analysis_scenario dataframe but not actually call the rowwise_fit() function or run the model. Then, open fit.r, define the variables below, and step through the individual lines within fit.r. The inputs to rowwise_fit() are: 

- a_scenario=analysis_summary
- yaml_info = config_list$yaml_info
- yaml_branches = config_list$yaml_branches
- n_chains = 3
- save_full_jags_obj = FALSE,
- destroy_xy = FALSE
- pass_errors = pass_errors,
- max_adapt_attempts = 3


Another helpful tip is to set save_mcarray to TRUE near the top of the analysis-pipeline.R script. This will save the z_jags file to the 99-misc folder in your outputs (the file name is z-jags.rds). z_jags is an input to the inference functions and can be used to generate new figures. 

## Cross-walk of functions with inference outputs
Sometimes the pipeline may run a model but error out as it is generating the site, stratum, park, or other types of inference. In those cases, it can be helpful to step through the individual functions that generate the various types of inference outputs. These functions are all called within the rowwise_fit() function as the pipeline is running. 

Below is a cross-walk of the type of inference, the function that generates each type of inference, including all figures and csv files, where they are defined in the source code, and their resulting output folders. The definition files are all contained in model-api\src. The output folders are all within the 03-inference folder. 

| Inference type | Function | Definition file | Output folder |
|:------|:------|:------|:------|
|   park-level    |   get_park_scale_inference()   |   inference-park-scale.R   |   park   |
|   stratum-level   |  get_stratum_inference()  |   inference-stratum-scale.R   |   stratum   |
|   site-level   |   get_site_inference()   |   serializers.R   |   site   |
|   trend (annual change)   |   get_trend_inference()   |   inference-trend.R   |   trend   |
|   covariates (marginal effects)   |   get_me_inference()   |   inference-me-drivers.R   |   me   |
|   covariates (coefficients)   |   get_coef_post_dists()  |   serializers.R   |   coef   |

### Inference function inputs from model outputs
If you need to do some troubleshooting of inference functions for a model that has already run, you can step through the functions using files saved in the output folder. Below is a cross-walk of the function inputs with the files in our output folders:

-	mcarray_list = .\99-misc\z-jags.rds
-	data = .\00-input\state-variable-data.csv
-	folder = full string for model’s folder (e.g., "C:/M4MD_SWNC/output/SODN/FOBO/pi-soil-binom/exposed_bareground/zi-od-binom_inv-logit_b0_fixed-stratum/clay_100cm_wy_deficit")
-	resp_var_desc = .\00-input\jags-info.rds, response variable (e.g., jags-info$hits for binomial model)
-	likelihood = .\00-input\jags-info.rds, likelihood variable (jags-info$liklihood)
-	deterministic_model = .\00-input\jags-info.rds, deterministic_model variable (jags-info$deterministic_model)
-	x_hat_raw = timesteps = .\00-input\state-variable-data.csv, x.hat.raw variable (data$x.hat.raw)
-	timesteps = .\00-input\state-variable-data.csv, x.hat.raw variable (data$x.hat.raw)
-	jags_obj_prefix = “hat” or “pred”
-	data_list = .\00-input\jags-data.rds


