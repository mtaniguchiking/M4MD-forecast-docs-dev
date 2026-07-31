---
title: Zones
weight: 2
---

# Zones
Estimate trend by management zones or other post hoc areas in addition to strata, useful for understanding patterns in park areas with different management or treatments (e.g., grazing, prescribed fire, etc.). 

## Syntax
```yml
Additional covariates:
-	Deficit, MgmtZone (deflections)
```
See [covariates] ({{< ref "/docs/1-guide/a-data/ii-x-info.md" >}}) for how to code deflections in your covariate file.

## Usage
The inference consists of a mean and credible interval across strata (i.e., at the park scale) for each of the management zones. There is no reference level: each management treatment has its own intercept or intercept and time slope. The model uses sum-to-zero effect coding (Kruschke 2015), which we call deflections, to estimate how much a specific zone deviates from an overall population baseline. See this write-up for a description of the math: [open document](zone/Predications_for_management_areas.pdf).

## Identifiability
If a management zone has very similar site membership to a stratum, there may be an issue of identifiability, where the model can't estimate the 'effect' of a site falling in the management zone because it's already being accounted for with the intercept term for the stratum. The two parameters (in this example, the coefficient for the zone effect and intercept term for the stratum) can trade-off and have no effect on the likelihood. This may manifest by models that do not converge. To account for it, in the config file under “group-level effects”, select “none”. 
