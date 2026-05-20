---
title: Data
bookCollapseSection: true
weight: 1
---

# Data

You need to format your data outside of models-for-missing-data and store it in the appropriate folder. Generally, CSV format is recommended. It is critical that you use the same names for your units (parks), sites (plots), and event date columns across your data files. Also ensure that the actual names of the units, sites, and dates are the same (e.g., don’t have SAGU in one file and SAGW in another). The scripts does a number of joins and you will get cryptic errors if your data cannot be joined. You may use the same location and covariate data file for multiple models. There are three types of data files:

1. Response data - the observations we are trying to model plus columns with identifiers for all relevant sampling design information
2. Location data - the coordinates of your sites (plots) in a projected coordinate system; centroids are fine
3. Covariate data - discrete or continuous variables that might influence the mean of the response variable we are interested in

The covariate data are optional. The models will run without a location file but you will not get the visualizations of the spatial dependence of the data in the output directory. 