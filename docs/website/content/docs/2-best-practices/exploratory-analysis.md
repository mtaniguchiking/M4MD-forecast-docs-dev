---
title: Exploratory data analysis
weight: 1
bookToc: true
---

# Exploratory data analysis

## The response data
We recommend taking a look at basic histograms of your response variable. If you don't know already, this is a great time to think about why [_type_ of data]({{< ref "/docs/1-guide/a-data/iii-types-of-random-variables/_index.md" >}}) you're working with. Is it a count, a proportion, or some other quantity? Look for signs of skewness or zero-inflation.

## The covariates
It may be helpful to evaluate the correlation between covariates and your response variable, and among covariates. Scatterplots of the response variable vs. each covariate can sometimes be illuminating. If two potential covariates are highly corelated, one should probably be dropped in favor of the other to ensure parsimony, model convergence, and sensible results.
