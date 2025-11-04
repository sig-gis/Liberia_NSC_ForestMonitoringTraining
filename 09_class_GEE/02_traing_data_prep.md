---
layout: page
title:  Preparing Training Data for Random Forest
parent: "9. Classification with GEE"
nav_order: 2
---


# Preparing Training Data for Random Forest

We need to train the Random Forest model (coming in the next section) by giving it examples of what each land cover looks like. We need to give the Random Forest model training data. 

We will do this by gathering a sufficient number of training points (preferably 50-100 minimum) from each land cover. We could randomly look for examples across the study area, but we have a pre-exisiting land cover map that should give a decent starting point and make this data collection more efficient. The pre-existing map was for 2014, but it will be handy to distribute samples in classes according to that map. The final resulting training data may not have the same classes as the original map for all points, either because of map errors, or because the land cover has changed between 2014 and our year of interest. The pre-exisiting map is just an efficiency tool for data gathering.

## Decision Making for the Data Collection 

### How Many Samples

Again, we define some important 
