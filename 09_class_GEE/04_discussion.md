---
layout: page
title:  Discussion
parent: "9. Classification with GEE"
nav_order: 4
---

# Discussion

## SEPAL vs. GEE

| SEPAL       | GEE         |
|-------------|-------------|
| quick to learn | takes more time to learn |
| no programming experience needed | requires understanding of programming basics |
| complicated workflow using multiple different tools | simple worflow that can be done in 2 scripts |
| limited choices for input data | can import any data sets from personal computer or GEE Data Catalogue (PALSAR, Harmonized Landsat Sentinel) |
| limited choices for pre- and post-processing | can access all pre- and post-processing GEE tools (smoothing, cloud masking, calculating indices) |
| limited options for altering model parameters | many options for altering model parameters (multiprobability mode, variable importance) |
| difficult to rerun to test out different versions | easy to rerun to test out different versions |


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




