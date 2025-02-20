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

## Visualize Spectral Signatures of Different LULC Types

Open up a new script and name it `4 spectral signatures`. You will copy and paste each code block into the empty script. You can check your work by looking at the following script `users/ee-scripts/Liberia_Forest_SIG_workshops/09_classification_GEE/4 spectral signatures`.

As part of the data exploration process, we can visualize the differences in how different LULC types reflect/reemit different wavelengths of light. These are called "spectral signatures," and they can give good insight into which bands or indices will be the most useful for distinguishing different LULC types in the classification, and which LULC types will be easily confused with one another. The greater the difference in spectral signatures, the easier the LULC types will be to separate.

First, we just import the data sets we have prepped for classification: the AOI, the predictor image, the reference points, and the 2014 LULC maps. 

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
Map.centerObject(aoi, 7)

// Reference Points
// ------------------------------------------------------------------------------------------

var refPoints = ee.FeatureCollection(
  "projects/pc556-ncs-liberia-forest-mang/assets/refPoints_10m_2014_400PerClass")
  
// add to map
Map.addLayer(refPoints, {}, 
  'reference points', false);
  
// Predictor Variable Image
// ------------------------------------------------------------------------------------------

var predImage = ee.Image(
  "projects/pc556-ncs-liberia-forest-mang/assets/predImage_30m_"+d1.slice(0,4)+'_v'+version)

// print band names
print("predictor image bands:", predImage.bandNames())
// add to map
Map.addLayer(predImage, {bands:['red','green','blue'],min:0,max:0.3}, 
  'optical', false);
  
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

Next, we create some `ui.Chart.image.byClass()` objects which aggregate the median of the predictor image bands at each reference point by LULC class. First we plot the optical bands as a line graph, and then we plot th eindices as a bar chart. We will not discuss each component of creating a chart in GEE here, but if you want to learn more about this topic there are plenty of resources online.

``` javascript
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////
// Visualize Spectral Signatures
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////

var sarChart = ui.Chart.image
                .byClass({
                  image: predImage
                          .select(['VV','VH','HH','HV'])
                          .addBands(lulc),
                  classBand: 'class',
                  region: refPoints,
                  reducer: ee.Reducer.median(),
                  scale: 30,
                  classLabels: ['','Forest >80%','Forest 30-80%','Forest <30%','Mangrove','Settlements',
                                '','Water','Grassland','Shrub','Bare Soil','Sand'],
                })
                .setChartType('ColumnChart')
                .setOptions({
                  title: 'Median SAR Backscatter Values',
                  hAxis: {
                    title: 'Polarization',
                    titleTextStyle: {italic: false, bold: true}
                  },
                  vAxis: {
                    title: 'Backscatter',
                    titleTextStyle: {italic: false, bold: true}
                  },
                  colors: [      
                                // 0 nodata
                    '#006d3a',  // 1 forest_80
                    '#009c53',  // 2 forest_30-80
                    '#00cc6c',  // 3 forest_30
                    '#00bba4',  // 4 mangroves
                    '#7b0000',  // 5 settlements
                    // 'white',    // placeholder for 6
                    '#015890',  // 7 water
                    '#b6da03',  // 8 grassland
                    '#d29f00',  // 9 shrub
                    '#e3e3e3',  // 10 baresoil
                    '#fff6a9'   // 11 sand
                                // 25 clouds
                  ]
                });

var l8Wavelengths = ['0.490 Blue', '0.560 Green', '0.665 Red',
                    '0.842 NIR', '1.610 SWIR1', '2.190 SWIR2']
var s2Wavelengths = ['0.490 Blue', '0.560 Green', '0.665 Red', '0.705 RedEdge1', 
                    '0.740 RedEdge2', '0.783 RedEdge3', '0.842 NIR','0.865 RedEdge4',
                    '1.610 SWIR1', '2.190 SWIR2']

var opticalChart = ui.Chart.image
                .byClass({
                  image: predImage
                          .select(['blue', 'green', 'red', 'NIR', 'SWIR1', 'SWIR2'])
                          .addBands(lulc),
                  classBand: 'class',
                  region: refPoints,
                  reducer: ee.Reducer.median(),
                  scale: 30,
                  classLabels: ['','Forest >80%','Forest 30-80%','Forest <30%','Mangrove','Settlements',
                                '','Water','Grassland','Shrub','Bare Soil','Sand'],
                  xLabels: l8Wavelengths
                })
                //.setChartType('ScatterChart')
                .setChartType('ScatterChart')
                .setOptions({
                  title: 'Median Optical Reflectance Spectral Signatures',
                  hAxis: {
                    title: 'Wavelength (µm)',
                    titleTextStyle: {italic: false, bold: true},
                    ticks: l8Wavelengths
                  },
                  vAxis: {
                    title: 'Reflectance',
                    titleTextStyle: {italic: false, bold: true}
                  },
                  colors: [      
                                // 0 nodata
                    '#006d3a',  // 1 forest_80
                    '#009c53',  // 2 forest_30-80
                    '#00cc6c',  // 3 forest_30
                    '#00bba4',  // 4 mangroves
                    '#7b0000',  // 5 settlements
                    // 'white',    // placeholder for 6
                    '#015890',  // 7 water
                    '#b6da03',  // 8 grassland
                    '#d29f00',  // 9 shrub
                    '#e3e3e3',  // 10 baresoil
                    '#fff6a9'   // 11 sand
                                // 25 clouds
                  ],
                  pointSize: 0,
                  lineSize: 2,
                  curveType: 'linear'
                });
                
var indexChart = ui.Chart.image
                .byClass({
                  image: predImage
                          .select(['NDVI','LSWI','NDMI','MNDWI','EVI','MVI'])
                          .addBands(lulc),
                  classBand: 'class',
                  region: refPoints,
                  reducer: ee.Reducer.median(),
                  scale: 30,
                  classLabels: ['','Forest >80%','Forest 30-80%','Forest <30%','Mangrove','Settlements',
                                '','Water','Grassland','Shrub','Bare Soil','Sand'],
                })
                .setChartType('ColumnChart')
                .setOptions({
                  title: 'Median Optical Index Values',
                  hAxis: {
                    title: 'Index',
                    titleTextStyle: {italic: false, bold: true}
                  },
                  vAxis: {
                    title: 'Index Value',
                    titleTextStyle: {italic: false, bold: true}
                  },
                  colors: [      
                                // 0 nodata
                    '#006d3a',  // 1 forest_80
                    '#009c53',  // 2 forest_30-80
                    '#00cc6c',  // 3 forest_30
                    '#00bba4',  // 4 mangroves
                    '#7b0000',  // 5 settlements
                    // 'white',    // placeholder for 6
                    '#015890',  // 7 water
                    '#b6da03',  // 8 grassland
                    '#d29f00',  // 9 shrub
                    '#e3e3e3',  // 10 baresoil
                    '#fff6a9'   // 11 sand
                                // 25 clouds
                  ]
                });

// print(sarChart);
print(opticalChart);
print(indexChart);
```

<img align="center" src="../images/class-gee/opticalbands_chart.png" hspace="15" vspace="10" width="800">

<img align="center" src="../images/class-gee/opticalindices_chart.png" hspace="15" vspace="10" width="800">

Looking at these charts, we can already predict that water will probably be extremely easy to distinguish from other classes, as it has a unique spectral signature and unique values in many of the indices. Sand and settlements, on the other hand, may be relatively easy to distinguish from other classes but may be difficult to distinguish from each other. Similarly, all the vegetation classes will probably be extremely difficult to distinguish from each other, with the NIR band looking like the best band for distinguishing them.

Another interesting thing to note is how similar the bare soil spectral signature is to the vegetation classes and how different it is from the sand and settlement classes - it is nearly identical to grassland. This is surprising, as bare soil should be much more similar in composition to sand and settlements than to vegetation. This could be because what was defined as bare soil in the 2014 map may not have been bare soil year-round, but rather bare soil in the dry season with a thin layer of annual herbacious vegetation in the dry season. Thus, when taking the annual median to composite the predictor image, areas which were bare in less than half of the images may appear greener than we might expect from a "bare soil" class. This is an example of why manual labelling of reference points with strict definitions and extensive QAQC is crucial. Our reference points for this exercise were just blindly generated from the 2014 map, which should not be done when generating a final product for real-world use. 

<!-- ## Rerun the Analysis for a Simpler LULC Typology -->





