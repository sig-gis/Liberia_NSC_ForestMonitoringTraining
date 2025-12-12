---
layout: page
title:  "Area Estimation and Uncertainty Analysis"
parent: "12. Sample-Based Area Estimation"
permalink: /AnalysisAreaEstimation
nav_order: 4
---

# Sample-Based Area Estimation and Uncertainty Analysis

The **confusion matrix** compares the mapped class with the labels assigned by interpreters at the reference points. It shows the agreement (green fields on the diagonal) and disagreement (yellow fields off the diagonal) frequency between the two.

It provides insight into the quality of the mapping method and helps to identify areas that may need further refinement or a double check in CEO.


To **estimate the area of a target land cover change class** we can apply a stratified estimator by Cochran (1977, Eq. 5.1). It incorporates the confusion matrix and each stratum area to **estimate the true unbiased area of the target land cover change class**. This method accounts for accuracy variability in the land cover map, improving area estimation precision.

*Made up example but strata weights from Arevalo et al. (2018)*



