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

## Sources
- Congalton, R. G., & Green, K. (2009). Assessing the accuracy of remotely sensed data: Principles and practices (2nd ed.). CRC Press.
- Olofsson, P., Foody, G. M., Herold, M., Stehman, S. V., Woodcock, C. E., & Wulder, M. A. (2014). Good practices for estimating area and assessing accuracy of land change. Remote Sensing of Environment, 148, 42–57. <a href="https://doi.org/10.1016/j.rse.2014.02.015" target="_blank" rel="noopener noreferrer">https://doi.org/10.1016/j.rse.2014.02.015</a>
