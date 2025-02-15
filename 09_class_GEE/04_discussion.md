---
layout: page
title:  Discussion
parent: "9. Classification with GEE"
nav_order: 4
---

# Discussion

## SEPAL vs. GEE

| SEPAL        | GEE          |
|:-------------|:-------------|
| **<font color = green> pro - </font>** quick to learn | **<font color = red> con - </font>** takes more time to learn |
| **<font color = green> pro - </font>** no programming experience needed | **<font color = red> con - </font>** requires understanding of programming basics |
| **<font color = red> con - </font>** complicated workflow using multiple different tools | **<font color = green> pro - </font>** simple worflow that can be done in 2 scripts |
| **<font color = red> con - </font>** limited choices for input data | **<font color = green> pro - </font>** can import any data sets from personal computer or GEE Data Catalogue (PALSAR, Harmonized Landsat Sentinel) |
| **<font color = red> con - </font>** limited choices for pre- and post-processing | **<font color = green> pro - </font>** can access all pre- and post-processing GEE tools (smoothing, cloud masking, calculating indices) |
| **<font color = red> con - </font>** limited options for altering model parameters | **<font color = green> pro - </font>** many options for altering model parameters (multiprobability mode, variable importance) |
| **<font color = red> con - </font>** difficult to rerun to test out different versions | **<font color = green> pro - </font>** easy to rerun to test out different versions |


## Ways to Improve Accuracy

### 1. Preprocessing
* increase the number of training and testing points 
    * overall and/or using a stratified approach with more points in classes with low accuracies
* change or combine cloud masking strategies
* composite imagery for more than 1 year
* merge or separate classes of interest (e.g. one combined forest class instead of separated forest classes, separate wetlands class)

### 2. Model Development
* add and remove predictor variables based on variable importance and accuracy (e.g. different combinations of imagery sources or indices)
* increase the number of trees in the random forest model
* generate more training points in the areas of highest uncertainty and retrain the model with the addition of these new points

### 3. Postprocessing
* change smoothing functions
* overlay with hand-digitized data




