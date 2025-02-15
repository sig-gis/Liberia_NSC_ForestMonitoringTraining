---
layout: page
title:  Challenges
parent: "9. Classification with GEE"
nav_order: 5
---

# Challenges

## Rerun the Analysis for 2024

Rerun the entire analysis for 2024, from the `preprocessing` and `classification` scripts. 

The necessary intermediate assets, `projects/pc556-ncs-liberia-forest-mang/assets/refPoints_10m_2014_400PerClass` and `projects/pc556-ncs-liberia-forest-mang/assets/predImage_30m_2024_v1` for doing the classification have already been exported for you, and the scripts are set up in such a way that they will import the correct assets if you simply change the `d1` and `d2` variables to `2024` at the top of the script. 
```javascript
// dates of interest
var d1 = '2024-1-1'
var d2 = '2024-12-31'
```

<!-- ## Visualize Spectral Signatures

## Rerun the Analysis for a Simpler LULC Typology -->





