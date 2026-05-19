---
title: Disable time
weight: 8
---

# Disable time as a fixed effect
Disable time as a fixed effect (remove time slope) so models are driven by covariates only. If you are fitting a model for forecasting, you must carefully consider whether or not to include a time term.

## Syntax
```yml
time effect: disabled
```
Note that having time effect followed by anything will also disable the linear time term. 

## Usage
See the [Fitting for forecasting]({{< ref "/docs/5-forecasting/guide/fitting-a-model.md" >}}) entry for a more thorough introduction to this topic.
