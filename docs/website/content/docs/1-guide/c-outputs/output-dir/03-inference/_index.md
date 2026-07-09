---
title: 03-inference
bookCollapseSection: false
weight: 3
bibFile: content/posts/bib.json
---

# 03-inference | model inference

## Overview

Tabular and graphical summaries of Markov chain Monte Carlo (MCMC) draws from the posterior distribution of the parameters of a Bayesian model. The asterisk corresponds to the coefficients. 
If the configuration file specifies `time effect: disabled`, only the `coef` (coefficient) folder is produced.
If the configuration file specifies `drivers: true`, an additional `me` (marginal effects) folder is produced.
If the pipeline specifies `save_forecast_inputs`, an additional `forecast` folder is produced.

| __File__ | __Description__ |
|:---|:---|
| coef/ | Model coefficients |
| me/ | Marginal effects |
| park/ | Park-level inference |
| site/ | Site-level inference |
| strata/ | Stratum-level inference |
| trend/ | Trend inference
| zone/ | Zone-level inference (optional) |
| coda-samples-quantiles.csv | Quantiles of the posterior distribution of key variables / parameters  |
| coda-samples-summary.txt | Descriptive statistics and quantiles of the posterior distribution of key variables / parameters |

## Subdirectories

### coef/ | model coefficients

| __File__ | __Description__ |
|:---|:---|
| additional-coef-estimates-Beta-untransformed.png | The posterior distribution of fixed effect (covariate) terms.  |
| beta-coef-estimates-untransformed.png | The median (points) and 95% equal-tailed credible intervals for group-level (random) effects. When the model is `b0` only the intercepts ({{< katex >}}\beta_{0j}{{< /katex >}}) will vary by site within each stratum. When the model is `b0-b1` both intercept ({{< katex >}}\beta_{0j}{{< /katex >}}) and time-slope terms ({{< katex >}}\beta_{1j}{{< /katex >}}) will vary by site (with correlation {{< katex >}}\rho{{< /katex >}}).  |
| hypers-hist-*.png | The hyper-distribution of random intercepts ("hypers-hist-B0.png") or random time-slope terms ("hypers-hist-B1.png") for each stratum  |

#### Examples
In the example below, the posterior distribution of the fixed effect (coefficient) is mostly above 0, indicating a positive relationship between the coefficient (mean.deficit) and the response (native graminoid cover). This figure will only appear in the output if you include a covariate in the model.
![addition-coef-estimates](01-additional-coef-estimates.png)

The example below shows the median (point) and 95% credible intervals (whiskers) for random effects. You can see how the estimate for the intercept ({{< katex >}}\beta_{0}{{< /katex >}}) varies by each site within each strata (hills and valley), where some are positive and some are negative. This particular model did not allow the time-slope terms ({{< katex >}}\beta_{1}{{< /katex >}}) to vary by site, so the coefficients for slope are fixed for each site in a stratum. If the model were `b0-b1` (varying intercept and slopes), you would also see the variation of the time-slope term by site.
![beta-coef-estimates]!(02-beta-coef-estimates-untransformed.png)

The mean ({{< katex >}}\mu{{< /katex >}}) and variance ({{< katex >}}\sigma{{< /katex >}}) of the hyper-distribution of random intercepts term for each strata.
![hypers-histogram](03-hypers-hist.png)

### me/ | marginal effects
Marginal effects folder includes any marginal effects specified in the config file.


| __File__ | __Description__ |
|:---|:---|
|me-fe_mean.deficit-band.jpg | A plot showing the marginal fixed effects, or the change in the response in relation to the covariate, while holding time constant. | 
|me-fe_mean.deficit-spaghetti.jpg | A variation of the plot above with different aesthetics. | 
|me-X.rds| The data required to reproduce the plot. | 

#### Examples
This example shows the relationship between the covariate (mean.deficit) and the response (native graminoid relative cover), while holding time constant. There appears to be a slightly positive slope. Refer to the coef output (additional-coef-estimates-Beta-untransformed.png) to see how the posterior distribution relates to 0 to determine whether there is a significant trend.
![me-fe](04-me-fe_mean.deficit-band.png)

The spaghetti plot shows the same information as the previous plot, just with lines for the posterior draws (darker shading indicates a higher concentration of lines) and dashed lines showing the 95% credible intervals. 
![me-fe-spaghetti](05-me-fe_mean.deficit.spaghetti.png)

### park/ | park-level inference
Park-level mean of the response over time. Subfolders include results for all possible sites ("all") and sampled sites ("sampled"). Unless you have provided covariates for all sites, inference with covariates (the `pred` files) will be limited to sampled sites.

In the file names below, the first asterisk corresponds to either `hat` or `pred`. The `hat` files hold any covariates present at their mean (essentially removing -- or controlling for -- the effect of changes in the covariates over time). The `pred` files (_if_ present) correspond to predictions of park-level means conditional on the actual time-varying values of covariates -- AKA "bumpy plots." The uncertainty intervals, sometimes called "credible intervals", are derived using the 95% [Highest (Posterior) Density Intervals](https://cran.r-project.org/web/packages/HDInterval/index.html), by default (dotted black lines). The median is shown in the solid black line. If "out of sample" (`oos`) JAGS objects, exist, results are provided both for the expected value (i.e., for the mean) of new observations as well as the value of new observations themselves {{< cite "hobbs2015bayesian" 197 >}}.


| __File__ | __Description__ |
|:---|:---|
| *-park-mean-annual-summary-hdi95.csv | Annual estimates of status (median, lower and upper credible intervals).  |
| *-park-mean-plot-hdi95.jpg | Graphical representations of the RDS file. In the figures, the semi-transparent thin lines are draws from the posterior distribution of the park-level mean; the thick, solid dark line is the median of the mean and the thick, dashed lines correspond to the 95% HDIs at each timestep. |
| *-park-mean-plot-objects-hdi95.rds| The data required to reproduce the plot. |

Depending on the analysis, several other files may appear, including: '\*-site-means-(\<stratum-id\>).png', and 'pred-site-means-df.csv'. 

#### Examples
In the “hat” plot, the covariate is held at its mean, producing a smooth trend line of mean native graminoid relative cover over time. 
![hat-park-mean](06-hat-park-mean-annual-summary-hdi95.png)

In the “pred” plot, the covariate is time-varying, showing the influence over changing levels of water deficit during the sampling period. Notice there still appears to be an overall declining trend.
![pred-park-mean](07-pred-park-mean-plot-hdi95.png)


### site/ | site-level inference
Under construction

### strata/ | stratum-level inference
Under construction

### trend/ | trend
Under construction

## References

{{< bibliography cited >}}
