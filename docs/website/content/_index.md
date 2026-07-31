---
title: Welcome
type: docs
image: /images/3D98AD77-CBC4-0963-8EFBF00EA361C5DDOriginal.jpg
bibFile: content/posts/bib.json
---

# Welcome to Models for Missing Data

This website serves as a storehouse of information about software used by several National Park Service [Inventory & Monitoring (I&M) networks](https://www.nps.gov/im/networks.htm) to develop models of the status and trends of park resources. It also features general-interest articles introducing statistical concepts and applications of models.

## Sections

{{< columns >}}
### Getting started
An overview to help decide if the models are appropriate for your data, repository cloning instructions, a rundown the required software, a suggested folder structure, a quickstart tutorial for those looking to be hands-on immediately with demo data, and how to sync changes.
<--->
### Guide
Technical information about the software, including _inputs_ and _outputs_. We describe the data, the metadata syntax used to specify models, and the various outputs of the program, from model checking to inference.
{{< /columns >}}

{{< columns >}}
### [Best Practices]({{< ref "/docs/2-best-practices/_index.md" >}})
Modeling best practices, including exploratory data analysis and troubleshooting.
<--->
### [Forecasting]({{< ref "/docs/5-forecasting/_index.md" >}})
Description of the forecasting extension. The forecasting pipeline takes in a fitted model and produces posterior predictive forecasts under user-specified future climate scenarios.
{{< /columns >}}

{{< columns >}}
### [Posts]({{< ref "/posts/_index.md" >}})
Blog-style posts containing more general content, including pieces on the missing data problem, proof-of-concept simulations, and example analyses.
<--->
{{< /columns >}}
### [FAQ]({{< ref "/docs/3-faq/_index.md" >}})
Your frequently asked question, answered!
{{< /columns >}}

## Contributors

The original contributing team includes Luke Zachmann, Tom Hobbs, Erin Borgman, Dana Witwicki, Megan Swan, Cheryl McIntyre, and Carolyn Livensperger, with oversight and support by Dusty Perkins. The forecasting extension was developed by the [Eric and Wendy Schmidt Center for Data Science & Environment](https://dse.berkeley.edu/) in collaboration with the National Park Service's Inventory & Monitoring Division.

## Recommended citation

The work described in these pages is based on Zachmann et al. 2020:

{{< bibliography cited >}}
