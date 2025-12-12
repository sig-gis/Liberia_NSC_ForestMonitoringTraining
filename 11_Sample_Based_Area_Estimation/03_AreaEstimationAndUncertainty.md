---
layout: page
title:  "Area Estimation and Uncertainty Analysis"
parent: "12. Sample-Based Area Estimation"
permalink: /AnalysisAreaEstimation
nav_order: 4
---

# Sample-Based Area Estimation and Uncertainty Analysis

The **confusion matrix** compares the mapped class with the labels assigned by interpreters at the reference points. It shows the agreement (green fields on the diagonal) and disagreement (grey fields off the diagonal) frequency between the two.
<img align="center" src="../images/areaestimation/confusionmatrixexample.png" hspace="15" vspace="10" width="600">


It provides insight into the quality of the mapping method and helps to identify areas that may need further refinement or a double check in CEO.


To **estimate the area of a target land cover change class** we can apply a stratified estimator by Cochran (1977, Eq. 5.1). It incorporates the confusion matrix and each stratum area to **estimate the true unbiased area of the target land cover change class**. This method accounts for accuracy variability in the land cover map, improving area estimation precision.

<img align="center" src="../images/areaestimation/Cochran51.png" hspace="15" vspace="10" width="300">

<img align="center" src="../images/areaestimation/matrixexample.png" hspace="15" vspace="10" width="600">

Use this template to do your own analysis. Make a copy and adjust the:
    -  columns to match your reference data labels
    -  rows to match your map strata
    -  pixel counts for your map strata (get this from AREA2 or QGIS)
    -  sample counts within the matrix
    
[Spreashsheet Template for Stratified Area and Uncertainty Estimation](https://docs.google.com/spreadsheets/d/1hCHuU13Rs7j2rj1Ll7IymBtGSSfZLOLg/edit?usp=sharing&ouid=113437415151435538893&rtpof=true&sd=true)
<img align="center" src="../images/areaestimation/fullareaestimationexample.png" hspace="15" vspace="10" width="900">



# [Optional] Systematic or Random Sampling Analysis
If the team ends up deciding to use systematic or random sampling instead of stratified sampling, then here are some resources to guide that analysis. The process is more straight forward. These slides and templates are pulled from a workshop for The Gambia.

- [Slides on Sample Based Area Estimation with Simple Random/Systematic Sampling](https://docs.google.com/presentation/d/10F7c5-laA-iiWy_slMzaQz7RlCatxZ3LywfKCTS9xm0/edit?usp=sharing)
- [Spreadsheet Template for Systematic/Random Sampling Area Estimation](https://docs.google.com/spreadsheets/d/1DrMBMR11tGpUeHOy6VwA39VrYJI4CNyC/edit?usp=drive_link&ouid=113437415151435538893&rtpof=true&sd=true)




