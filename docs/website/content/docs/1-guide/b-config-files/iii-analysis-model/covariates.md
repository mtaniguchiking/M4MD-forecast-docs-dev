---
title: Covariates
weight: 5
---

# Covariates
The predictors to use for modeling the mean of the response variable.

## Syntax

```yml
additional covariates:
    - Botanist (JA)
    - Botanist (JA), deficit.pregr
```

## Usage
For categorical variables (e.g., `Botanist`), the optional parenthetical declaration can do one of two things.
1. It sets the reference level for dummy-coded variables. In the case of `Botanist (JA)`, the reference level becomes the botanist with initials `JA`. If not specified, the reference level will be set using R's default handling for factors (using the first level of the factor sorted in ascending alphabetical o.rder as the reference level)
2. It can also be used to implement sum-to-zero effect coding, which we call deflections. For instance `MgmtZone (deflections)`. In the case of deflections, the coefficients for each level of the categorical variable sum to zero. By default the model returns only the first {{< katex >}}k-1{{< /katex >}} coefficients, but the {{< katex >}}k^{\mathrm{th}}{{< /katex >}} coefficient can be computed as a derived quantity. See [this StackExchange post](https://stats.stackexchange.com/a/163148) for more on this calculation.

See [this resource](https://stats.oarc.ucla.edu/spss/faq/coding-systems-for-categorical-variables-in-regression-analysis) for additional background on dummy vs effect coding.

Interaction terms can be specified using "star" notation. So a `Botanist (JA), deficit.pregr` specification would produce a botanist-by-deficit interaction term. Note that there is no automatic expansion for the main effects, so a more complete specification would entail something like the following:

```yml
additional covariates:
    - Botanist (JA), deficit.pregr, Botanist (JA) * deficit.pregr
```

## Non-linear terms
By default each covariate enters the model linearly. You can also include power and transform terms to capture non-linear relationships between a predictor and the response. As with interactions, there is no automatic expansion of the main effects. This means every term should be listed explicitly.

### Powers
Raise a covariate to the 2nd or 3rd power with `^`:

```yml
additional covariates:
    - deficit.pregr, deficit.pregr^2
```

This fits both a linear and a quadratic `deficit.pregr` effect (a parabola). Only powers `2` and `3` are supported.

### Transforms
Apply a monotone transform by wrapping the covariate in one of `log`, `log1p`, or `sqrt`:

```yml
additional covariates:
    - log(deficit.pregr)
    - sqrt(deficit.pregr)
```

- `log(x)` — natural log; requires all values `> 0`
- `log1p(x)` — natural log of 1 + x; requires all values `> -1` (use this when `x` can be `0`)
- `sqrt(x)` — square root; requires all values `>= 0`

If any value falls outside the transform's domain, the fit stops and reports the offending rows.

> The transform argument must be a single covariate name, meaning nested expressions like `log(deficit.pregr^2)` are not supported.

### Combining with interactions
Powers and transforms can appear on either side of an interaction:

```yml
additional covariates:
    - deficit.pregr, deficit.pregr * MgmtZone, deficit.pregr^2, deficit.pregr^2 * MgmtZone
    - log(deficit.pregr) * Botanist (JA)
```

### Notes
1. As with interactions, no main effect is added automatically. For example, `deficit.pregr^2` on its own fits only the quadratic term
2. Standardization is applied automatically before fitting, so the estimated coefficients are on the standardized scale.