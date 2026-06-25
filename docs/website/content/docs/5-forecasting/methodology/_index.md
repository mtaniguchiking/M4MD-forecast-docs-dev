---
title: Methodology
bookCollapseSection: true
weight: 3
---

# Methodology

This section explains how the forecasting pipeline works under the hood. If the [Guide]({{< ref "/docs/5-forecasting/guide" >}}) covers how to run a forecast, this section covers what happens when you do.

The three pages below move from a high-level view down to the underlying calculations.

{{< hint info >}}
**Pipeline Architecture** — the big picture

- How the forecasting scripts fit together and how data flows between them
- An architecture diagram and a reference table for each forecasting script

→ [Pipeline Architecture]({{< ref "/docs/5-forecasting/methodology/architecture" >}})
{{< /hint >}}

{{< hint warning >}}
**Pipeline Walkthrough** — what each step does

- A walkthrough of the 11 steps of `forecast-pipeline.R`
- An explanation of what each step reads, computes, and produces

→ [Pipeline Walkthrough]({{< ref "/docs/5-forecasting/methodology/pipeline-walkthrough" >}})
{{< /hint >}}

{{< hint danger >}}
**Pipeline Math** — the calculations

- The posterior predictive equations the pipeline applies to a fitted model and future scenarios
- Notation linking the Zachmann et al. (2021) paper symbols to the pipeline code

→ [Pipeline Math]({{< ref "/docs/5-forecasting/methodology/math" >}})
{{< /hint >}}