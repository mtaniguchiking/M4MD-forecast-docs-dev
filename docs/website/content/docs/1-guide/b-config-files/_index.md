---
title: Config files
bookCollapseSection: true
weight: 2
---

# Config files

We use YAML to create analysis configuration files. These files are intended to be human-readable, and they describe how a model relates to the data, including nonignorable aspects of the sampling design. We organize information about the sampling design, the data, and the model in this way:

1. Design metadata
2. The analysis config file, which in turn consists of:
    - The data (response, site location, and covariate information)
    - The model (likelihood, link function(s), group-level effects, predictor variables, etc.)
    - Other conditions or quantities (posterior predictive checks, etc.)

We call these subcomponents of the analysis config file "blocks". The blocks provide sensible organization to the specifications of an analysis.

The ./assets/uplands-config/templates folder contains a set of templates for common types of data within the NPS Inventory and Monitoring division. The templates include all of the options & extras and are organized to match the developers guide. The templates include:

1. covercclass-ordinal.yml – Plant cover measured using Daubenmire cover classes, modeled as ordinal data from a beta distributed latent variable
2. density-counts.yml – Counts of individuals of by species/groups where you want to account for area (e.g., get results as individuals/acre), modeled as poisson or similar
3. gaps-gamma.yml – Canopy or basal gaps (or cover) measured using line intercept, modeled as gamma 
4. lpi-binomial.yml – Plant or soil cover measured using line-point intercept, modeled as binomial or similar
5. presence-binomial.yml – Presence/absence of species/groups, modeled as binomial or similar
6. richness-counts.yml – Species richness (counts of species), modeled as poisson 
7. soil-stability-ordinal.yml – Soil aggregate stability ratings (1-6), modeled as ordinal data from a normally distributed latent variable
8. species-counts.yml – Counts of individuals of by species/groups, modeled as poisson or similar

For more background on YAML files, see [this helpful tutorial](https://www.cloudbees.com/blog/yaml-tutorial-everything-you-need-get-started).
