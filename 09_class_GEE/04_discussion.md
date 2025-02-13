---
layout: page
title:  Discussion
parent: "9. Classification with GEE"
nav_order: 4
---

# Discussion

## SEPAL vs. GEE

| :SEPAL       | GEE         |
|:-------------|:-------------|
| <font color = green> **pro -** </font> quick to learn | **con -** takes more time to learn |
| <font color = green> **pro -** </font> no programming experience needed | **con -** requires understanding of programming basics |
| **con -** complicated workflow using multiple different tools | <font color = green> **pro -** </font> simple worflow that can be done in 2 scripts |
| **con -** limited choices for input data | <font color = green> **pro -** </font> can import any data sets from personal computer or GEE Data Catalogue (PALSAR, Harmonized Landsat Sentinel) |
| **con -** limited choices for pre- and post-processing | <font color = green> **pro -** </font> can access all pre- and post-processing GEE tools (smoothing, cloud masking, calculating indices) |
| **con -** limited options for altering model parameters | <font color = green> **pro -** </font> many options for altering model parameters (multiprobability mode, variable importance) |
| **con -** difficult to rerun to test out different versions | <font color = green> **pro -** </font> easy to rerun to test out different versions |


## Ways to Improve Accuracy

### 1. Preprocessing
* increase the number of training and testing points
* change or combine cloud masking strategies
* composite imagery for more than 1 year
* merge or separate classes of interest (e.g. one combined forest class instead of separated forest classes, separate wetlands class)

### 2. Model Development
* add and remove predictor variables based on variable importance and accuracy (e.g. different combinations of imagery sources or indices)
* increase the number of trees in the random forest model

### 3. Postprocessing
* change smoothing functions
* overlay with hand-digitized data




