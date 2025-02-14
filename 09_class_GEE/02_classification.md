---
layout: page
title:  Random Forest Classification
parent: "9. Classification with GEE"
nav_order: 2
---

# Classification with Random Forest in GEE

Open up a new script and name it `1 classification`. You will copy and paste each code block into the empty script. You can check your work by looking at the following script `users/ee-scripts/Liberia_Forest_SIG_workshops/09_classification_GEE/1 classification`.

## Setup

### Set Important Parameters

Again, we define some important parameters at the beginning of the script so they are easy to change later on. These are related to the version number, time period of interest, final map resolution, and smoothing.

The version number and dates should match the ones in the preprocessing script. 

```javascript
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////
// Define Parameters
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////

// version
var version = 1;

// dates of interest
var d1 = '2014-1-1'
var d2 = '2014-12-31'

// basemap
Map.setOptions('SATELLITE')

// class band name
var classBand = 'class' //'simplifiedClass' //'class'

// final map resolution
var resolution = 30;
// smoothing radius for final map (pixels)
var modeRadius = 2;
```

### Import Data

We have already preprocessed and exported all data sets we need to run the classification, so now we just import them into this script and add them all to the map. This includes the AOI, the reference points, the predictor image (also being used as the training image), and the original 2014 LULC maps.

```javascript
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////
// Import and Visualize Data
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////

// AOI
// ------------------------------------------------------------------------------------------

// import the simple Liberia feature collection
var Liberia = ee.FeatureCollection("projects/pc556-ncs-liberia-forest-mang/assets/Liberia_simple")
  
// define an aoi geometry from the feature collection
var aoi = Liberia
  .union();
  
// Add the aoi object as a layer to the map
Map.addLayer(aoi, {}, 'AOI', false);

// Reference Points
// ------------------------------------------------------------------------------------------

var refPoints = ee.FeatureCollection(
  "projects/pc556-ncs-liberia-forest-mang/assets/refPoints_10m_"+d1.slice(0,4)+"_400PerClass")

// Predictor Variable Image
// ------------------------------------------------------------------------------------------

var predImage = ee.Image(
  "projects/pc556-ncs-liberia-forest-mang/assets/predImage_30m_"+d1.slice(0,4))

// add to map
Map.addLayer(predImage, {bands:['red','green','blue'],min:0,max:0.3}, 
  'optical', false);
Map.addLayer(predImage, {bands:['red_planet','green_planet','blue_planet'],min:64,max:5454,gamma:1.8}, 
  'optical planet', false);
  
// LULC
// ------------------------------------------------------------------------------------------

var lulc10m = ee.Image(
  'projects/pc556-ncs-liberia-forest-mang/assets/Liberia_landcover_forest_map_10m_v1_2014')
var lulc30m = ee.Image(
  'projects/pc556-ncs-liberia-forest-mang/assets/Liberia_landcover_forest_map_30m_2014')

// do some preprocessing to remove classes we don't want
lulc10m = lulc10m
  // redefine clouds as 0
  .where(lulc10m.eq(25), 0) 
  // get rid of 0 values (nodata an dclouds)
  .selfMask()
  // rename class band
  .rename('class')
lulc30m = lulc30m
  // redefine clouds as 0
  .where(lulc30m.eq(25), 0) 
  // get rid of 0 values (nodata an dclouds)
  .selfMask()
  // rename class band
  .rename('class')

// define visualization paramaters
var lulcVis = {
  min: 1,
  max: 11,
  palette: [
                        // 0 nodata
            '#006d3a',  // 1 forest_80
            '#009c53',  // 2 forest_30-80
            '#00cc6c',  // 3 forest_30
            '#00bba4',  // 4 mangroves
            '#7b0000',  // 5 settlements
            'white',    // placeholder for 6
            '#015890',  // 7 water
            '#b6da03',  // 8 grassland
            '#d29f00',  // 9 shrub
            '#e3e3e3',  // 10 baresoil
            '#fff6a9'   // 11 sand
                        // 25 clouds
            ],         
                        
};
// Add to the map
Map.addLayer(lulc10m, lulcVis, 'LULC 2014 10m', false);
Map.addLayer(lulc30m, lulcVis, 'LULC 2014 30m', false);

// select which lulc to use for generating reference data
var lulc = lulc10m;
```

*If you are getting an error about Planet at this point, why might that be?*


## Prepare Training and Testing Points

Next, we extract all the values from the **Training Image** (remember this is the same as the file we saved as our Predictor Image, both 2014) to the reference points, and then separate the points into a training and testing set in a 80%-20% split (which yields approximately 320 training points and 80 testing points per LULC class). 

This meets the minimum requirements of the 10*p training and √(p):1 testing rules of thumb. That is assuming we are using our most complicated image with lots of variable bands (which would dictate at least 220 training and 48 testing points per class). For 2014 we only have Landsat, so we could technically have fewer points, but we will keep it at 400 since this would work for all of our images and more points only help.

We split the data by generating a new property in the reference data points which consists of random numbers with values between 0 and 1 (in binary numbers). Then, we separate out the points with a value greater than 0.8 and less than 0.8 in the random column - which statistically should give us about an 80%-20% split.

```javascript
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////
// Prepare Training and Testing Data
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////

// extract the training image band values at the reference point locations
refPoints = predImage.sampleRegions({
      collection: refPoints, 
      properties: [classBand], 
      scale: resolution,
      geometries:true
    })

// refPoints = lulc.sampleRegions({
//       collection: refPoints, 
//       properties: [classBand], 
//       scale: 10,
//       geometries:true
//     })

// Add to the map
Map.addLayer(refPoints, {}, 'reference points', false);
// print
print('reference points:', refPoints.limit(5))

// Divide reference points into training and testing points
// Create random column in reference points
// Note: There is an equal number of points per class label, 
//    so a fully random selection of the 80/20 works fine instead of getting 80/20 per class
refPoints = refPoints.randomColumn();

// set aside 80% of data for training 
var trainPoints = refPoints.filter(ee.Filter.lt('random', 0.8));
// set aside 20% of the data for testing
var testPoints = refPoints.filter(ee.Filter.gte('random', 0.8));

// print size of testing and training data sets
print('Number of training points:', trainPoints.size());
print('Number of testing points:', testPoints.size());

// add to map
Map.addLayer(trainPoints, {color: 'black'}, 'training points', false); 
Map.addLayer(testPoints, {color: 'white'}, 'testing points', false); 

// alternatively,  import the training and testing points created in SEPAL or AREA2 
// (comment out the rest of this section above)
// var trainPoints = ee.FeatureCollection()
// var testPoints = ee.FeatureCollection()
```

<img align="center" src="../images/class-gee/trainingtesting_pts.png" hspace="15" vspace="10" width="400">

*Tip:* For most models (with between 10 and 50 predictor bands), the data splitting ratio will fall somewhere between a 75%-25% split and a 90%-10% split. You should always strive to generate as much reference data as you can within your human and computational resource limits.

Your training and testing sample points are now ready! They have both class labels and extracted band information from the Training Image. They will serve as examples for Random Forest of what spectral information of those classes look like. 

## Train the Classifier

Now, we run the classification in two separate steps - training and deployment:
1. **train** the classifier on the training points
2. **deploy** the classifier on either the predictor image or the testing points.

As discussed before, the random forest model repeatedly selects a random subset of training points and generates a series of thresholds (called decision trees) in the predictor variables that would most effectively categorize the training points into the classes of interest. When all of these decision trees are constructed, the model can be applied to a predictor image. It applies all the decision trees to every single pixel in the image, and aggregates the result of all the trees to make a final class assignment to each pixel (usually either mean or mode).

<img align="center" src="../images/class-gee/randomforest.png" hspace="15" vspace="10" width="600">

*Resource:* For some background on Random Forest, you can go to the [Machine Learning page](https://sig-gis.github.io/Liberia_NSC_ForestMonitoringTraining/02_Intro_MachineLearning) in this website, and another brief intro can be found at this [website](https://www.ibm.com/think/topics/random-forest).

First, we select the predictor variables we want the classifier to use. This is where we can remove bands as we refine the model. We remove any `constant` bands which are leftover artifacts from our exported image. Then, we train the `ee.Classifier.smileRandomForest()` classifier with 200 trees, the prepped training points, the selected LULC class band, and the selected prediction bands. 

```javascript
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////
// Train Classifier
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////

// Define prediction bands
// Get all image bands from the predictor image and remove any you want to leave out
var predBands =  predImage.bandNames()
  .removeAll(['constant','constant_1','constant_2']) //adjust as needed, the constant bands are for the demo

// print prediction bands
print('prediction bands:', predBands)

// Train random forest classifier with the training points and prediction bands
var RFclassifier = ee.Classifier.smileRandomForest({numberOfTrees:200, seed:234})
  .train({
    features: trainPoints, 
    classProperty: classBand,
    inputProperties: predBands
});

// Print decision trees
print('decision trees:', RFclassifier.explain());
```

We also print out the prediction bands and the decision trees of the random forest model so we can explore what kinds of cutoffs the random forest decision trees have created in the different predictor variables.

<img align="center" src="../images/class-gee/prediction_bands.png" hspace="15" vspace="10" width="200">

<img align="center" src="../images/class-gee/trees.png" hspace="15" vspace="10" width="400">

## Deploy Classifier

Second, we actually deploy the classifier on the full predictor image to get a classification map. We also apply a focal mode function to reduce speckling and reproject to our desired resolution (the final resolution should be no lower than the lowest resolution predictor variable dataset). 

```javascript
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////
// Run Classifier
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////

// Classify the predictor image with the trained classifier
var RFclassification = predImage
  .select(predBands)
  .classify(RFclassifier)
  // smooth to make it less speckly
  .focalMode(modeRadius, 'circle', 'pixels')
  .reproject({
    crs: predImage.projection(),
    scale: resolution})
  .clip(aoi)

// Add the classified image to the map
Map.addLayer(RFclassification, lulcVis, 
            'RF classification');
```

<img align="center" src="../images/class-gee/classification1.png" hspace="15" vspace="10" width="600">

*Tip:* When you add the final classification to the map, it might give you an error for trying to load in the entire AOI at once. We are asking the script to do many computationally expensive calculations (random forest, focal mode, etc.) and then render the products of those calculations directly onto the map in high resolution. The solution to this is exporting the final image and then reimporting it to put it onto the map, which is much less computationally expensive. Zooming in to a smaller area can also help. If you just want to look at the map, you can even comment out the unneeded parts of the script to make the image load even faster.

## Assess Accuracy

Now, we take a look at the accuracies of each individual class. We first classify the testing points and then extract the error matrix, for which we just provide the property names of the "true" and "predicted" values (which are the original extracted LULC values and the LULC values predicted by the model). We print the error matrix and the lists of user's and producer's accuracies, but we do **not** print the overall accuracy. This simple way to calculate accuracy, where we divide the number of correctly classified points by the total number of points, is **not appropriate** here because our original sample was a **stratified** random sample. We need to do a **weighted** accuracy assessment, where we weight the class accuracies based on their predicted areas, and then aggregate them. We will do this separately in a spreadsheet.

*Resource:* For some background on accuracy assessment and user's and producer's accuracy, you can go to the [Accuracy Assessment page](https://sig-gis.github.io/Liberia_NSC_ForestMonitoringTraining/10_accuracy) in this website.

```javascript
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////
// Assess Accuracy
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////

// classify the testing data using the trained classifier
var testPointsClassified = testPoints.classify(RFclassifier);

// Create confusion matrix using the true and predicted values in the tetsing data
var confusionMatrix = testPointsClassified.errorMatrix({
  // the "true" values from the reference data
  actual: classBand, 
  // the "predicted" values from the RF classification
  predicted: 'classification'
});

// Print confusion matrix and accuracies
// This is overall accuracy is a simplified version - is NOT appropriate for stratified random sampling
// print('Overall Accuracy:', confusionMatrix.accuracy());
print('Confusion Matrix:', confusionMatrix);
print('Producers Accuracy:', confusionMatrix.producersAccuracy());
print('Users Accuracy:', confusionMatrix.consumersAccuracy());
```

<img align="center" src="../images/class-gee/accuracy.png" hspace="15" vspace="10" width="200">

For helpful reference, these are the labels associated wih your class numbers.

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


## Assess Variable Importance

Lastly, we'll take a look at variable importance. The `ee.Classifier.smileRandomForest()` function automatically calculates variable importance based on the Gini Impurity Index, which measures how much each predictor variable reduces impurity in classes at each split (node) in the decision trees. We scale these so all variable importances add up to 100, and then we both print them out and create a bar chart.

```javascript
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////
// Assess Variable Importance
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////

// extract the importance of each predictor variable from the random forest model
var varImportance = ee.Dictionary(RFclassifier.explain().get('importance'));

// convert raw importance values to percentages (so they all add up to 100)
// sum up all importances
var varImportanceSum = varImportance.values().reduce(ee.Reducer.sum());
// multiply each importance value by 100 and divide by the sum to relativize them
var varRelImportance = varImportance.map(function(key, val) {
  return (ee.Number(val).multiply(100)).divide(varImportanceSum);
  });

// sort importances to go from lowest to highest
var sortedProperties = varRelImportance.keys().sort(varRelImportance.values())
var sortedImportances = ee.List([sortedProperties, varRelImportance.values().sort()])
// print
print('relative importance of variables:',sortedImportances);

// create a feature collection from the relative importances
var varRelImportanceFC = ee.FeatureCollection([
  ee.Feature(null, varRelImportance)
]);

// create a chart of the relative importances
var varRelImportanceChart = ui.Chart.feature.byProperty({
  features: varRelImportanceFC
}).setOptions({
      title: 'Variable Relative Importance',
      vAxis: {title: 'Importance'},
      hAxis: {title: 'Bands'}
  });
// print
print(varRelImportanceChart);
```

<img align="center" src="../images/class-gee/variableimportance.png" hspace="15" vspace="10" width="600">

Now, we could refine the model by removing variables one by one, starting with the ones with lowest importance and seeing how many you can remove before it dramatically reduces accuracy. You can do this earlier in the script with the `.removeAll()` function, recording the variable importances and resulting accuracies at each removal. Usually, you will see a sharp decrease in accuracy after removing certain variables, indicating which variables should be kept in the model.

*Try it out. Remove a variable band or two and re-run th script. How did the accuracies change?*

## Calculate Areas

Additionally, we need to calculate class areas based on the map (*these are pixel counts not our final unbiased area estimates!*) to use later in our weighted accuracy assessment. We use the `ee.Reducer.count()` function to count the number of pixels in each class and then store these values in a `List` of `Dictionaries`. Each `Dictionary` has 2 keys: the class and the pixel count. We convert these to a `featureCollection` with 12 empty features in it (one for each class), attaching the correct class and pixel count as properties to each feature. Thus, we end up with a data type that can be exported as a .csv containing a row for each class and a column containing the pixel counts of those classes (unfortunately, converting to a `featureCollection` is the only way to export a table from GEE).

```javascript
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////
// Calculate Areas
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////

// Create an extra image with a value of 1 in every pixel covering the AOI
// (this is just an something needed for the ee.Reducer.group() function because it needs an image with 2 bands)
var constantImage = ee.Image(1).clip(aoi);

// Calculate the number of pixels in each class
var pixelCounts_dict = constantImage.addBands(RFclassification).reduceRegion({
  // Use ee.Reducer.count().group() function
  reducer: ee.Reducer.count().group({
    // Use band "classification" for grouping 
    // (which is now band 1 not 0 since we attached the extra image)
    groupField: 1, 
    groupName: 'classification',
  }),
  geometry: aoi, // region of interest
  scale: 30, // Resolution of the image in meters
  maxPixels: 1e50 // Maximum number of pixels in the region
});

// // Print the resulting dictionary of a list of dictionaries
// print("Pixel Counts as Dictionary:", pixelCounts_dict);

// Convert the this dictionary to just a list of dictionaries
// (we extract the list of dictionaries)
var pixelCounts_list = ee.List(pixelCounts_dict.get('groups'));

// // Print the resulting list of dictionaries
// print("Pixel Counts as List of Dictionaries:", pixelCounts_list);

// Create a FeatureCollection from the list of dictionaries
var pixelCounts_fc = ee.FeatureCollection(
  // apply this function to all the dictionaries in the list
  pixelCounts_list.map(function(dict) {
    var dict2 = ee.Dictionary(dict);
    // extract the className from the dictionary
    var className = ee.Number(dict2.get('classification'));
    // extract the count from the dictionary
    var count = ee.Number(dict2.get('count'));
    // put these into an empty feature class as new properties
    // (this allows us to export the info as separate columns in a CSV)
    return ee.Feature(null, {
      'classification': className,
      'count': count
    });
  })
);

// Print the resulting FeatureCollection
print("Pixel Counts as FeatureCollection", pixelCounts_fc);
```

<img align="center" src="../images/class-gee/pixelcounts.png" hspace="15" vspace="10" width="200">

## Export

The last thing we do is export the classified testing points to our asset library for proper accuracy assessment and the classification map for easy visualization. We also export the testing points and class areas to our Google Drive for use in the accuracy assessment.

```javascript
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////
// Export
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////

// Export the classification map to Asset Library
Export.image.toAsset({
  // the image you want to export
  image: RFclassification,
  // the name of the task
  description: 'RFclassification',
  // the path and name of the asset (change this to be in your own asset library)
  assetId: 'projects/pc556-ncs-liberia-forest-mang/assets/RFclassification_'+resolution+'m_'+d1.slice(0,4)+'_v'+version,
  // the geometry to clip the image to
  region: aoi,
  // the resolution of the image
  scale: resolution,
  maxPixels: 1e13
})
            
// Export the reference points Asset Library
Export.table.toAsset({
  // the feature collection you want to export
  collection: testPointsClassified,
  // the task name
  description: 'testPointsClassified',
  // the asset ID and path (change to your own asset library)
  assetId: 'projects/pc556-ncs-liberia-forest-mang/assets/testPointsClassified_'+resolution+'m_'+d1.slice(0,4)+'_v'+version,
});

// Export the class areas as a CSV to Google Drive
Export.table.toDrive({
  // the feature collection
  collection: pixelCounts_fc,
  // the task name
  description: 'classAreas',
  // folder and file name in Drive
  fileNamePrefix: 'classAreas_'+resolution+'m_'+d1.slice(0,4)+'_v'+version,
  fileFormat: 'CSV'
});

// Export reference points to Google Drive
Export.table.toDrive({
  collection: testPointsClassified,
  // the task name
  description: 'todrive_testPointsClassified',
  // folder and file name in Drive
  fileNamePrefix: 'testPointsClassified_'+resolution+'m_'+d1.slice(0,4)+'_v'+version,
  // columns to export
  selectors: 'system:index,classification,class'
});
```

Code checkpoint: check your work in `users/ee-scripts/Liberia_Forest_SIG_workshops/09_classification_GEE/2 classification`.