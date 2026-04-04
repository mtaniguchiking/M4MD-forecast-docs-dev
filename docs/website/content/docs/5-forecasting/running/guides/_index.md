---
title: Guides
bookCollapseSection: true
weight: 1
---

# Running a Forecast

There are three sets of instructions for running a forecast, depending on your familiarity with the pipeline and the data you're starting with.

{{< hint info >}}
**Mock data (your first time forecasting)**

- If you want to perform your first forecast, you can start here. It provides simple mock data
and teaches you how to fit a model for forecasting, how to forecasting, and how to interpret the results.

→ [Mock data walkthrough](mock-data)
{{< /hint >}}

{{< hint warning >}}
**Climate futures data**

- If you have your own model fitting data but *not* your own future scenario data,
you can get some here. It details how to forecast using simple CCRP climate futures covariate data.

→ [Climate futures data tutorial](climate-futures-data)
{{< /hint >}}

{{< hint danger >}}
**Your own data**

- If you have your own model fitting data *and* your own future scenario data, you
can learn how to forecast here. This details the configuration options and outputs you
should know about.

→ [Your own data guide](custom-data)
{{< /hint >}}


<!-- TODO: Brief intro paragraph explaining the three paths:
     1.1 Mock data — lowest barrier, all data pre-committed, best for first-timers
     1.2 Real-world data via notebooks repo — moderate lift, uses external tool
     1.3 Fully custom — all data user-provided, full config reference

     Suggest starting with 1.1 if the user is new to the forecasting pipeline.
-->
