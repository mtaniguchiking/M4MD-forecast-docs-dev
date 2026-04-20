---
title: Installation
weight: 1
bookToc: true
---

# Installation

## Obtaining a copy of the repository

### New here?
To clone the repository, open a terminal[^1] and run the following command _from_ the directory into which you'd like to place the project (e.g., from `~/repos`).
```sh
git clone https://github.com/lzachmann/models-for-missing-data.git [DIRNAME]
```

`DIRNAME` is optional, and specifies the name of the directory into which the project will be cloned on your local machine. You could call it "m4md" for instance, if you wanted something a bit shorter than the default "models-for-missing-data."

### NPS I&M staff or partner?
The latest version of the scripts have migrated to the [National Park Service's Southwestern Network Collaboration GitHub account](https://github.com/NPS-SWNC/M4MD). The models for missing data scripts are in a private repository that will become public at a later date. If you are a National Park Service Inventory and Monitoring staff member of partner, please send an email to [Cheryl McIntyre](mailto:cheryl_mcintyre@nps.gov) to request access and include your GitHub username (ideally associated with a NPS email). 

To clone the updated M4MD repository from GitHub to a NPS computer, we recommend that you do so by creating a new Version Control Project within RStudio. See Day 4: Version Control of the [IMD Advanced R Training](https://katemmiller.github.io/IMD_R_Training_Advanced/) for step-by-step instructions.

o	As of April 2026, the most up-to-date scripts are in the forecasting/main branch of the [M4MD repository]((https://github.com/NPS-SWNC/M4MD)). The forecasting/main branch includes updates to refactor the code and the forecasting extension developed by the [Eric and Wendy Schmidt Center for Data Science & Environment](https://dse.berkeley.edu/). The main branch and the [original scripts](https://github.com/lzachmann/models-for-missing-data) include many functions that are deprecated (e.g., mutate_). The pipeline and models will error out if you try to use the main branch to run a model. 