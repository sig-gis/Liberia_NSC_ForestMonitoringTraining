---
layout: page
title: "9. Classification with GEE"
permalink: /09_class_GEE
has_children: True
nav_order: 10
---

# Classification with Google Earth Engine (GEE)

This portion of the workshop will demonstrate a Random Forest classification mapping workflow using a combination of data sources like optical imagery, SAR imagery, and elevation layers. We will  import and pre-process the necessary data and use it to train and run a Random Forest model.

Follow along by copying and pasting each code block in the lesson into your own blank script. At the end you will have the entire workflow saved to a script file on your own GEE account.

## Objectives
1. Understand the general process for training and applying a model in Google Earth Engine on satellite data
2. Adapt the provided workflow for a different areas of interest and time periods
3. Experiment with different ways to improve accuracy of your classification

## Setup     

1. Log into your Google Earth Engine account and open the code editor.
2. Click this link to accept the GEE script repository as a "reader" - https://code.earthengine.google.com/?accept_repo=users/ee-scripts/Liberia_Forest_SIG_workshops
3. Create 2 new script files in your own script repository - name them `Preprocessing` and `Classification`.
4. You can always check the full scripts here in the above reposity, in a folder called `09_classification_GEE` with this lesson's scripts in it - `preprocessing` and `classification`.