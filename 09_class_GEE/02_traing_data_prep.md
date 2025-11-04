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

### (Reminder from the Sampling Design Section) Sampling design considerations for ML algorithm training
For training machine learning algorithms, sampling design is critical to ensure that the data used to train the model covers the variability in the dataset, preventing overfitting or underfitting. Sometimes the full data set of samples is split into training and testing subsets to evaluate model performance, but separate sampling design for the validation data can also be done. Anything you want the ML algorithm to learn, should be provided as an example in the training samples.

General rules of thumb to use for sampling design to gather training data for Random Forest:
- A commonly cited rule is to have at least 10 times as many samples as there are features (variables) in the dataset to provide enough data for the model to learn meaningful patterns.
    -   Example: For 20 features, aim for at least 200 samples per class
- Decide on a minimum number of samples per class:
    -   For simple problems, aim for 50–100 samples per class.
    -   For complex problems or high-dimensional datasets, aim for 500–1,000 samples per class or more.
- Classes with fewer samples (minority classes) should have a minimum of 30–50 samples to ensure meaningful learning.
- Highly variabile classs requires larger sample sizes to capture the diversity.

### How Many Samples

Based on the logic above we will allocate 100 samples to each of the 10 classes in the original LC map. 
