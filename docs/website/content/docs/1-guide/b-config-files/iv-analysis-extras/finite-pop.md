---
title: Finite population correction
weight: 2
---

# Finite population correction
When the size of the sample collected is a significant proportion of the overall population, then we must use a finite population correction (FPC), a topic we introduce briefly in [another post]({{< ref "/posts/sampling-and-populations.md" >}}).

## Syntax
```yml
finite population correction: true
finite population info:
    file: assets/uplands-data/ROMN/LIBI_SampledUnsampled_Ratio.csv
    columns with the number of sampled and unsampled sites:
       - '# sampled sites'
       - '# unsampled sites'
    covariate info includes unsampled sites: true
```

## Usage

To toggle FPC, use the `finite population correction` key.

```yml
finite population correction: true
```

When toggled, FPC requires the population size of each stratum. The population sizes can be specified
via one of two options.

**Option 1:** The `finite population info` file. This should be
a small CSV with one row per stratum giving the number of sampled and unsampled sites.

```yml
finite population info:
    file: # the path to the counts CSV
    columns with the number of sampled and unsampled sites:
        # the two column names in that CSV, in order
        - # name of sampled column
        - # name of unsampled column
```

The CSV must include a column identifying the stratum, matching the
stratification column used elsewhere in your config. Any stratum not included in the
CSV is treated as having 0 unsampled sites (i.e. fully sampled).

An example counts CSV:

| Master_Stratification | # sampled sites | # unsampled sites |
|-----------------------|-----------------|-------------------|
| Gulley1 | 6 | 23 |
| Gulley2 | 8 | 30 |
| Uplands | 3 | 21 |

**Option 2:** The covariates CSV. This involves assigning the `covariate info includes unsampled sites` key to `true`, which is detailed below.

The `covariate info includes unsampled sites` flag indicates whether covariate values are included for all unsampled sites.

```yml
covariate info includes unsampled sites: # true/false
```

If set to `true`, the covariate data CSV should contain data for all sites (rows for both the sampled + unsampled sites).

If set to `false`, the covariate data CSV should only contain data for the sampled sites.

> **When to set to true/false:** If you do not have measured covariates at your unsampled
> sites, use `false`. If unsampled-site covariates are avaiable and you want them to inform the model, use `true`.

If both Option 1 and Option 2 are provided, the population sizes are verified against each other to ensure the sizes align.