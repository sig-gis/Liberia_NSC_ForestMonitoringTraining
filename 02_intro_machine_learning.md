---
layout: page
title:  "Intro to Machine Learning"
parent: "1. Intro to Machine Learning"
nav_order: 1
---

---
layout: page
title: "2. Introduction to Machine Learning"
permalink: /02_Intro_MachineLearning
nav_order: 3
---

<img align="center" src="../images/ML/ML_machinelearning_graphic.pngpng" hspace="5" vspace="5" width="800">


# Introduction to Machine Learning
Machine Learning (ML) is a type of artificial intelligence (AI) 
where computers learn patterns from data to make predictions or decisions without explicit programming. 

It involves training algorithms on large datasets to identify patterns and relationships between input variables, which can then be used to make predictions or decisions, improving prediction accuracy over time.

**Random Forest** is a simple type of ML.

## Random Forest

A model is the mathematical function or algorithm that learns patterns from data and makes predictions. One of the things a model can be used fore is **classification**.

A **Random Forest** is a machine learning model that makes predictions by combining multiple decision trees. It is called a "forest" because it consists of many trees, and each tree votes to make the final decision.

In machine learning, a **classifier** is an algorithm that automatically sorts data into categories or classes. The goal of a classifier is to learn from training data and make accurate predictions about new data.

A **Random Forest Classifier** is a model trained to classify images into categories like "forest" or "non-forest".

It performs well in predicting most classes, but may struggle with classes that have similar characteristics.

<img align="center" src="../images/ML/ML_randomforest_graphic.pngpng" hspace="5" vspace="5" width="800">

## Other Terms to Know

- Training Data – The dataset used to train an ML model. 
    - Sometimes called "reference data".

- Testing Data – A separate dataset used to evaluate the model’s performance.
    - This is usually proportionally much smaller than the training data. 

- Decision Tree – A model that makes predictions by splitting data into branches based on feature values.
    - Random Forest is an ensemble learning method that builds multiple decision trees and combines their outputs for better accuracy and robustness.

- Features - The input variables used in a machine learning model to make predictions. Features provide the information that models use to learn patterns and make predictions.
    - In our case, the *features* are the available *bands* of the composite image we give the ML model.

- Feature Importance – A score that indicates how valuable each feature is in making predictions.

