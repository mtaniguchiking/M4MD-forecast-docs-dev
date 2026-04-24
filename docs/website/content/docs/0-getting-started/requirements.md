---
title: "Requirements"
weight: 2
bookToc: true
---

# Requirements

The requirements include R (v4.X.X) and RStudio (if you'd like to work with __models-for-missing-data__ interactively), and JAGS (v4.3.0). Git and Git Bash (for Windows users) are convenient if you'd like to stay up to date with the latest changes.

If you are working on a National Park Service computer, there are a few additional requirements and suggestions. You can install most of the required software yourself from the NPS Company Portal but not JAGS. Installing JAGS requires a Bison Support System (BSS) ticket.

From NPS Company Portal, install:  

  - R For Windows 4.5.x or higher (required) 
  - RStudio 2025.05.0-496 or higher (required) 
  - RTools 4.4 or higher (required) 
  - Git (x64)-2.51.2.0 or higher (required) 
  - NPS Fonts (optional)

You can check for and install the required packages from your R console with:
```sh
packages <- c("abind", "spsurvey", "hrbrthemes", "ggthemes", "ggridges", "cowplot",
              "HDInterval", "rjags", "coda", "R2jags", "runjags", "bayesplot", "extraDistr",
              "MCMCpack", "magick", "gifski", "gganimate", "multidplyr", "gstat")
install.packages(setdiff(packages, rownames(installed.packages())))  
```

   
The requirements (including a few non-essentials) are described in the latest Dockerfile. You're likely to need everything you see beneath `install2.r` in the instructions below. You can install these from your R console with `install.packages()`.
{{< code file="/static/Dockerfile-latest" language="go" >}}
