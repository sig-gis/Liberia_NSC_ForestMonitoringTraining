---
layout: page
title: "12. Sample-Based Area Estimation"
permalink: /12_AreaEstimation
has_children: True
nav_order: 12
---

# What is Sample-Based Area Estimation?
- A statistical approach used to infer the characteristics of a larger area or population by analyzing data from a representative subset (sample) of that area
    - Namely, *sample-based area estimation*, for estimating areas of a feature and quantifying uncertainties, avoiding the bias inherent in using maps for area estimation

## Use: Inferring characteristics of a population based on a sample (e.g. sample-based area estimation)
Historically, maps have been used to quantify the area of each class by *pixel counting*. Pixel counting approaches simply sum up the area belonging to each class. 
 - However, this method is prone to bias due to classification errors (e.g., data noise, pixel mixing, or poor class separation).
- Pixel counting can result in over- or underestimates, and you cannot quantify this uncertainty.

*Sample-based approaches* use manually collected samples and statistical formulas based on the sampling design to estimate class areas.
- Uses manually collected reference samples (typically collected in a program like CEO) and statistical methods to estimate class areas. 
- The sample data is used to make estimates for for the whole area of interest.
- Provides **unbiased estimates and quantifies errors**, making results more reliable and robust. 




# Reminder... Overview of Sampling Design

## What is Sampling Design?
Sampling design refers to the structured approach used to select a subset of data points or observations from a larger population or dataset. A well-thought-out sampling design ensures that the selected sample is representative of the broader population, reducing bias and improving the reliability of results. 

The design includes several key dedision points such as the: 
- total sample size (number of samples)
- sampling unit (size and shape of each sample)
- sampling technique (e.g. random, gridded, proportional allocation across strata) 
- method of sample distribution (tool used to assign sample locations)

<img align="center" src="./images/sampling/map_graphic_wide.png"  vspace="10" width="300" border-radius="50%"> 



## What are the goals of a good sampling design?

1. **Representativeness**
- Ensure the sample accurately reflects the population or area of interest.
- Include all key subgroups or characteristics to avoid bias.
- Achieve proportional representation where needed, such as in stratified sampling.
- Avoid over-representing specific regions or under-sampling others.
- Ensure variability within each class is captured in the sample.
2. **Efficiency and Practicality**
-  Maximize the accuracy and reliability of the estimates while minimizing resource use, such as time, cost, and effort.
- Estimate the available resources and the number of samples needed to achieve goals early on in your design.
- Some methods for improving efficiency include stratified sampling and systematic sampling.
3. **Unbiased Estimation**
- Avoid systematic errors in selecting or measuring samples.
- Use probability-based methods to ensure each unit has a known and non-zero chance of selection, which should be recorded for later analyses.
4. **Flexibility**
- Allow for adaptability to unforeseen challenges during sampling, such as planning for iterative additional sampling.
- Plan for potential integration of additional samples or modifications without compromising the design's integrity.
5. **Validity for Analysis**
- Allow for robust statistical analysis and meaningful inference about the population.
- Ensure the sample size is adequate for achieving reliable estimates and uncertainties.
    - For most statistical analyses important to our use case, at least 30 samples from the class of interest are necessary for reliable results.
6. **Reproducibility**
- Design a sampling process that can be repeated by others for consistency and verification.
- Use clear, documented procedures for sample selection and measurement.


## Sampling design considerations for quantifying the characteristics of a popultation
Probability-based methods of sampling are used to ensure each unit has a known and non-zero chance of selection, which is required for statistical analysis and uncertainty estimations. The major objective of the sample is to provide information that is representative of the full population.

In sample-based area estimation, such as estimating forest cover, a representative sampling design ensures accurate population metrics while minimizing variance and resource use. The selected type of sampling design is tailored to the study's objectives and constraints. 

A map is not necessary for selecting sample locations but it makes the representative sampling process far more efficient when you are looking at a characteristic of interest with a geographically small area within the population, reducing the total number of points you need to collect to meet your desired level of uncertainty in your estimates.


## Some Well Known Sampling Techniques

There are several different ways to sample an area in order to achieve a representative sample or the landscape and the variations within it. Here are a few ways to distribute samples.

<img align="center" src="./images/ceo/4D_systematicsampling.png"  vspace="10" width="250"> 

**Systematic Sampling**: observations are placed at equal intervals according to a strategy

<img align="center" src="./images/ceo/4E_randomsampling.png"  vspace="10" width="250"> 

**Simple Random Sampling**: observations randomly placed

<img align="center" src="./images/ceo/4F_stratifiedsampling.png"  vspace="10" width="250"> 

**Stratified Random Sampling**: Using a map to inform the design, a minimum number of observations randomly placed in each category

Stratified random sampling has two key benefits
* Updates map-based areas to increase precision (reduces uncertainty)
* Helps increase chance of having plots in rare classes



## Sampling design considerations
Random Sampling option:
- Goal: A random sample allows for simple analysis using a confusion matrix.
- Impact: Rare classes in the map may have too few points for meaningful accuracy analysis.
- Consideration: You should have at least 30 samples per class for robust and statistically valid results, so you may need to distrinute a lot of random samples to achieve this if you have rare classes.

Stratified Sampling option:
- Goal: Ensures sufficient points (e.g., >30) for rare classes by sampling disproportionately across strata.
- Impact: Stratified sampling introduces unequal sample sizes, so although helpful for class-specific accuracy (allows >30 points per strata), it can skew overall accuracy assessments if not correctly weighted
- Consideration: In order to adjust for this unequal distribution you must use a slightly more complicated analysis for your overall accuracy, by weighting class-specific accuracy based on the area proportions.

### *The >30 samples rule*
The Central Limit Theorem states that, for a sufficiently large sample size (commonly 30 or more), the sampling distribution of the sample mean approaches a normal distribution, regardless of the population's actual distribution. This normality assumption is crucial for many statistical tests and confidence interval calculations used in accuracy assessments. 
### Also Note...
The samples you are using for training the machine learning algorithm **ARE NOT THE SAME** as the samples that will be used for area estimation. 


<br />
<br />
<img align="center" src="./images/sampling/stats_graphic.png"  vspace="10" width="200"> 


## Futher explanation of statistical details 



### Using a Confusion Matrix (simple unweighted example)

We can quantify the accuracy of the map using a confusion matrix (error matrix). The reference data dictates the actual value (the truth) while the left shows the prediction (or map classification).
    - True positive and true negative mean that the classification correctly classified the labels (e.g., a flood pixel was correctly classified as flood). 
    - False positive and false negative mean that the classification does not match the truth (e.g., a flood pixel was classified as no-flood) 

<img align="center" src="./images/ceo/7A_confusionmatrix.png"  vspace="10" width="600"> 

Let’s fill in this confusion matrix with example values if 100 points were collected.
<img align="center" src="./images/ceo/7B_accuraciestable.png"  vspace="10" width="600"> 


**Producer’s Accuracy**

* The percentage of time a class identified on the ground is classified into the same category on the map. The producer’s accuracy is the map accuracy from the point of view of the map maker (producer) and is calculated as the number of correctly identified pixels of a given class divided by the total number of pixels in that reference class. The producer's accuracy tells us that for a given class in the reference pixels, how many pixels on the map were classified correctly.  **The percentage of time a class identified on the ground is classified into the same category on the map.**
* Producer's Accuracy (for flood) = True Positive / (True Positive +False Positive)
* Producer's Accuracy (for no-flood) = True Negative / (True Negative +False Negative)
* On a confusion matrix, for each class it is the... *on-the-diagonal value* / *column's sum*.

**Omission Error**

* Omission errors refer to the reference pixels that were left out (or omitted) from the correct class in the classified map. An error of omission will be counted as an error of commission in another class. (complementary to the producer’s accuracy)
* Omission error = 100% - Producer’s Accuracy
* (Flood) Omission Error is when ‘flood’ is classified as some ‘other’ category. 

**User’s Accuracy**

* The percentage of time a class identified on the map is classified into the same category on the ground. The user’s accuracy is the accuracy from the point of view of a map user, not the map maker, and is calculated as the number correctly identified in a given map class divided by the number claimed to be in that map class. The user’s accuracy essentially tells us how often the class on the map is actually that class on the ground.  **The percentage of time a class identified on the map is classified into the same category on the ground.**
* User's Accuracy (for flood) = True Positive / (True Positive +False Negative)
* User's Accuracy (for no-flood) = True Negative / (True Negative +False Positive)
* On a confusion matrix, for each class it is the... *on-the-diagonal value* / *row's sum*.

**Commission Error**

* Commission errors refer to the class pixels that were erroneously classified in the map. (complementary to the user’s accuracy)
* Commission error = 100% - user’s accuracy.
* (Flood) Commission Error is when ‘other’ is classified as ‘flood’.

**Overall Accuracy**

* Overall accuracy = (True Positive + True Negative) / Sample size
* The overall accuracy essentially tells us what proportion of the reference data was classified correctly
* On a confusion matrix, it is the... *sum of all on-the-diagonal values*.

<br />
<br />

## Sources
- Congalton, R. G., & Green, K. (2009). Assessing the accuracy of remotely sensed data: Principles and practices (2nd ed.). CRC Press.
- Olofsson, P., Foody, G. M., Herold, M., Stehman, S. V., Woodcock, C. E., & Wulder, M. A. (2014). Good practices for estimating area and assessing accuracy of land change. Remote Sensing of Environment, 148, 42–57. <a href="https://doi.org/10.1016/j.rse.2014.02.015" target="_blank" rel="noopener noreferrer">https://doi.org/10.1016/j.rse.2014.02.015</a>
