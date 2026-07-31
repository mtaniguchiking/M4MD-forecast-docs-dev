---
title: Covariates
weight: 5
---

# Covariates
The predictors to use for modeling the mean of the response variable. Covaraites can be continuous (e.g., climatic water deficit) or categorical (e.g., soil texture class). Standardization is applied to continuous covariates automatically before fitting, so the estimated coefficients are on the standardized scale.

## Syntax
By default, each covariate enters the model linearly. Additive terms can be specified using “comma” notation. So a `TextureClass, deficit.pregr` specification would include soil texture and deficit as linear covariates.

```yml
additional covariates:
    - TextureClass
    - TextureClass, deficit.pregr
```

Interaction terms can be specified using "star" notation. So a `TextureClass * deficit.pregr` specification would produce a defict-by-texture interaction term. Note that there is no automatic expansion for the main effects, so a more complete specification would entail something like the following:

```yml
additional covariates:
    - TextureClass, deficit.pregr, TextureClass * deficit.pregr
```

### Non-linear terms
You can also include power and transform terms to capture non-linear relationships between a predictor and the response. As with interactions, there is no automatic expansion of the main effects. This means every term should be listed explicitly.

#### Powers
Raise a covariate to the 2nd or 3rd power with `^`:

```yml
additional covariates:
    - deficit.pregr, deficit.pregr^2
```

This fits both a linear and a quadratic `deficit.pregr` effect (a parabola). Only powers `2` and `3` are supported. As with interactions, no main effect is added automatically. For example, `deficit.pregr^2` on its own fits only the quadratic term.

#### Transforms
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

#### Combining with interactions
Powers and transforms can appear on either side of an interaction:

```yml
additional covariates:
    - deficit.pregr, deficit.pregr * TextureClass, deficit.pregr^2, deficit.pregr^2 * TextureClass
    - log(deficit.pregr) * TextureClass
```

## Categorical variable handling
By default, dummy coding is applied to categorical variables. For categorical variables (e.g., `Botanist`), the optional parenthetical declaration can set the reference level or implement sum-to-zero effect coding. See [this resource](https://stats.oarc.ucla.edu/spss/faq/coding-systems-for-categorical-variables-in-regression-analysis) for additional background on dummy vs effect coding.

### Reference level
Unless otherwise specified, the reference level will be set using R's default handling for factors. That is, the reference level will be the the first level of the variable when the factors are sorted in ascending alphabetical order. The user can specify the reference level by using the optional parenthetical declaration. For example, when specifying `Botanist (JA)`, the reference level becomes the botanist with initials `JA`.  

If the covariate is a nuisance variable, you may also want to "fix" the variable at a specific value for the inference outputs. You can change the way `pred`-type means are evaluated using the `trend conditions` block, described [here]({{< ref "/docs/1-guide/b-config-files/iv-analysis-extras/conditions.md" >}}). See the [object types]({{< ref "/docs/1-guide/c-outputs/object-types/_index.md" >}}) entry for a more thorough introduction to this topic.

### Sum-to-zero effect (deflections)
The optional parenthetical declaration can also be used to implement sum-to-zero effect coding, which we call deflections. When deflections are specified, the coefficients for each level of the categorical variable sum to zero rather than setting one level of the categorical variable as the reference level. For example, if you are interested in inference for management zones (or other zones that were not used in stratification), you specify `MgmtZone (deflections)`. So, rather than comparing each level of `MgmtZone` to a reference level, the `MgmtZone (deflections)` specification compares each level of `MgmtZone` to the grand mean.

If you specify `(deflections)`, you must include the effect coding in the covariates data file. See [Covariates Data]({{< ref "/docs/1-guide/a-data/ii-x-info.md" >}}) for how to code deflections in your covariate file. 

The `(deflections)` specification will produce a [zone inference folder]({{< ref "/docs/1-guide/c-outputs/output-dir/03-inference/_index.md" >}}) in the output directory. By default the model returns only the first {{< katex >}}k-1{{< /katex >}} coefficients, but the {{< katex >}}k^{\mathrm{th}}{{< /katex >}} coefficient can be computed as a derived quantity. See [this StackExchange post](https://stats.stackexchange.com/a/163148) for more on this calculation. 
