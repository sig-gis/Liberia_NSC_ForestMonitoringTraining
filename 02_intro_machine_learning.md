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

A model is the mathematical function or algorithm that learns patterns from data and makes predictions. One of the things a model can be used fore is **classification**, for our case it will be image classification.

A **Random Forest** is a machine learning model that makes predictions by combining multiple decision trees. It is called a "forest" because it consists of many trees, and each tree votes to make the final decision.

In machine learning, a **classifier** is an algorithm that automatically sorts data into categories or classes. The goal of a classifier is to learn from training data and make accurate predictions about new data.

A **Random Forest Classifier** is a model trained to classify images into categories like "forest" or "non-forest".

It performs well in predicting most classes, but may struggle with classes that have similar characteristics.

<img align="center" src="../images/ML/ML_randomforest_graphic.pngpng" hspace="5" vspace="5" width="700">

### General Steps of Random Forest

1. **Prepare the training data**
    - Establish the input variables (the feature, which are bands of your training image)
    - Establish the target labels (e.g., land cover classes)
    - Create a set of sample points with given target labels and extract feature values from the image
2. **Split the sample dataset into training and testing sets**
3. **Create and train several decision trees**
    - All trees are each a bit different as they are trained independently
    - Bootstrap sampling: A different random selection from the training data is used to train each tree
    - Random feature selection: At each node of the tree a random set of input variables (features) is chosen on which to base the decision of that node
<img align="center" src="../images/ML/ML_randomtrees.png" hspace="5" vspace="5" width="600">

4. **Predict class labels for an image**
    - Given an image where none or only some of the pixel labels are known, each tree makes a predition for every pixel
    - (e.g., Is this forest or non-forest?)
5.  **Aggregate the trees' predictions to reach a final decision** 
    - (e.g., majority answer)
5. **Evaluate the model using the testing dataset**

<img align="center" src="../images/ML/ML_full_process_graphic.pngpng" hspace="5" vspace="5" width="1000">


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


## Sources
- Breiman, L. (2001). Random Forests. Machine Learning, 45(1), 5–32.
- Hastie, T., Tibshirani, R., & Friedman, J. (2009). The elements of statistical learning: Data mining, inference, and prediction (2nd ed.). Springer.
- Ho, T. K. (1995). Random Decision Forests. Proceedings of the 3rd International Conference on Document Analysis and Recognition, 278–282.
- Liaw, A., & Wiener, M. (2002). Classification and Regression by randomForest. R News, 2(3), 18–22.
- GeeksforGeeks. (n.d.). Random Forest Algorithm in Machine Learning. Retrieved from https://www.geeksforgeeks.org/random-forest-algorithm-in-machine-learning/
- Analytics Vidhya. (2021, June). Understanding Random Forest Algorithm With Examples. Retrieved from https://www.analyticsvidhya.com/blog/2021/06/understanding-random-forest/
- [image] MyGeoBlog. (2019, October 18). Using artificial intelligence for satellite image classification. MyGeoBlog. https://mygeoblog.com/2019/10/18/using-artificial-intelligence-for-satellite-image-classification/
- [image] Manning Publications. (n.d.). Bagging. Manning LiveBook. Retrieved February 6, 2025, from https://livebook.manning.com/concept/machine-learning/bagging



