---
title: The output directory
bookCollapseSection: true
weight: 2
---

# The output directory

All outputs / logs are sent to the "assets" folder. Its contents mirror -- _almost_ exactly -- the directory structure of the "config" folder. The only difference is that the results of analyses for each state variable are stored in separate directories, whereas the configuration of analyses for like-variables (related count or cover variables, for example) can be specified simultaneously in the same YAML file. In other words, there can be many more output directories than analysis config files!

At a high level, the directory structure associated with the output of a given model looks like the following:

    . # output root
    ├── 00-input/
    ├── 01-diagnostics/
    ├── 02-checking/
    ├── 03-inference/
    ├── 99-misc/
    ├── calling-script.R   
    ├── inits-needed.txt   
    ├── mod-summary.csv    
    ├── model.jags         
    ├── output-metadata.md 
    └── system-time.txt

From the output root (`.`), there are an ordered / numbered set of directories and several files. The numeric prefix assigned to each directory is intended to provide some indication of the intended sequence by which an analyst would inspect model outputs.

| __File__ | __Description__ |
|:---|:---|
| calling-script.R | Reproducible calling script |
| inits-needed.txt | List of the parameters for which initial values (`inits`) are required |
| model.jags | JAGS model file |
| mod-summary.csv | Test statistics (`p_sd`, `p_mean`), posterior predictive loss (`ppl`), deviance information criterion (`dic`), and {{< katex >}}\hat{R}{{< /katex >}} (`gelman_diag`) |
| output-metadata.md | Basic metadata for files included in the output folder |
| system-time.txt | Time required to fit the model |

We will work through the contents of each of the directories in turn.

## A note on model summary statistics
* p_mean and p_sd should be between 0.1 - 0.9, with values near 0.5 representing the best fit to data. 
* If gelman_diag is good and p_mean and p_sd are within a good range, your “best” model is chosen by a combination of DIC and ppl. 
    * DIC is similar to AIC – lowest is best model 
    * Lower ppl is better 
    * If DIC is within a couple digits, models are not all that different and can choose based on lower ppl 
* Gelman_diag is convergence – close to 1.0 is ideal. If it’s higher, increase n_adapt and n_iter and see if it improves. If it’s very high and won’t come down, there might be something wrong with your model/data/covariates.

If running multiple models and want to easily compare them, you can compile multiple mod-summary files using the following code:
`source('src/wrangle.r')`

`get_mod_summary_files(  './output/SCPN/WUPA_S/cc-fxn/')`
