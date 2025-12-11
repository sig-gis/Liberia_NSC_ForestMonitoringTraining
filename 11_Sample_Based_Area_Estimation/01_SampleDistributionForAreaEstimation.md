---
layout: page
title:  "Sampling Design using AREA2, for Area Estimation"
parent: "12. Sample-Based Area Estimation"
permalink: /AREA2samplingAreaEstimation
nav_order: 2
---

# Using AREA2 for Sample Distribution
Refer to the full directions on AREA2 on the [3. Sampling Design / Sampling Design using AREA2 page](/03_Sampling_Design/02_AREA2_sampling_design.md).

## Further Resources
We reviewed how to use AREA2 in a follow-up workshop to the main workshop event. The recording of that walk-through is [here](https://drive.google.com/file/d/1oIiLt1I1oAF8WQIHUtkbHYQQQyzYhKD-/view?usp=sharing). You can stop watching at 16:00, when the topic switches to uploading the samples to CEO. 

The Google Sheets template we used in the follow-up workshop to calculate the number of samples per strata (proportional with a minimum sample size) is [here](https://docs.google.com/spreadsheets/d/1O2-5G-W4lZET1e8tv4uyK_-6D93FIprmj7ksD4J1WtA/edit?usp=sharing). You can make a copy of the template in order to make your own edits.

You will most likely be using Stratified Random Sampling (using a map to inform the design, a minimum number of observations randomly placed in each category), so that is assumed here. However, for the area estimation of land cover change **you will be using a different map asset than was used in the video.**

## Option 1: Making a Stratification Map

The plan discussed in the workshop event was to use the land cover maps created using Random Forest, comparing them to make **LC Change Maps** to be used as the stratification maps for sample distribution.  This could be done in GEE or QGIS, so we will discuss both options.

## Overview
Post-classification comparison is a straightforward approach for generating change maps from two land cover (LC) datasets (t1 and t2). Remember, these are the labels associated wih your class numbers for the LC maps.

 |Numeric code         |  Class name     |
 |:-------------:|:-------------:|
 | 0  | nodata |
 | 1  | forest_80 |
 | 2 | forest_30_80 |
 | 3 | forest_30 |
 | 4 | mangroves |
 | 5  | settlements |
 | 7  | water |
 | 8  | grassland |
 | 9  | shrub |
 | 10  | baresoil |
 | 11 | sand |


To make the change maps, you compare the class value of each pixel at time 1 against its value at time 2 and then assign a change label. This can be done with:
- **Full-detail transitions:** every possible class-to-class change (e.g., `forest_80 → settlements`). - **Simplified schemes:**
- binary (`change` / `no_change`), or
- thematic categories (e.g., `no_change`, `forest_loss`, `mangrove_loss`, `other_change`).
The LC classes:
```
1: forest_80
2: forest_30-80 3: forest_30
4: mangroves 5: settlements 7: water
8: grassland
9: shrub
10: bare_soil 11: sand
```
---
# 1. GEE Method
## General Approach
Use the two LC rasters (`lc_t1`, `lc_t2`) to calculate transition codes or classify changes directly using logical expressions.
---
## A. Detailed Transition Map (all change types)
Each transition can be uniquely encoded:
```js
var transition = lc_t1.multiply(100).add(lc_t2);
// e.g., forest_80 (1) → settlements (5) becomes 105 ```
---
## B. Simplified Change Classes
```js
var forestClasses = [1,2,3]; // forest strata
var isForest_t1 = lc_t1.remap(forestClasses, [1,1,1], 0); var isForest_t2 = lc_t2.remap(forestClasses, [1,1,1], 0);
var isMangrove_t1 = lc_t1.eq(4); var isMangrove_t2 = lc_t2.eq(4);
var noChange = lc_t1.eq(lc_t2); ```
```js
var changeClass =
noChange.multiply(1) // 1 = no_change .where(isForest_t1.eq(1).and(isForest_t2.eq(0)), 2) // forest_loss .where(isMangrove_t1.eq(1).and(isMangrove_t2.eq(0)), 3) // mangrove_loss .where(noChange.not()

.and(isForest_t1.eq(0).or(isForest_t2.eq(0))) .and(isMangrove_t1.eq(0).and(isMangrove_t2.eq(0))), 4); // other_change ```
---
## C. Binary Change / No-Change Map
```js
var changeBinary = lc_t1.neq(lc_t2).rename('change'); // changeBinary: 1 = change, 0 = no_change
```
---
## D. Basic Steps in GEE
1. Load LC maps (`lc_t1`, `lc_t2`), ensure same projection/resolution.
2. Compute either full transition image, simplified change classes, or binary map. 3. Visualize and export.
---
# 2. QGIS Method
## A. Binary Change / No-Change
```text
("LC_t1@1" != "LC_t2@1") ```
---
## B. Detailed Transition Codes
```text

("LC_t1@1" * 100) + "LC_t2@1" ```
---
## C. Simplified Change Classes
```text
if(
"LC_t1@1" = "LC_t2@1",
1,
if(
("LC_t1@1" = 1 OR "LC_t1@1" = 2 OR "LC_t1@1" = 3)
AND NOT("LC_t2@1" = 1 OR "LC_t2@1" = 2 OR "LC_t2@1" = 3), 2,
if(
("LC_t1@1" = 4) AND ("LC_t2@1" != 4),
3,
4
)
)
)
```
---
## D. Basic Steps in QGIS
1. Load rasters.
2. Use Raster Calculator to generate change classes. 3. Style and export.
