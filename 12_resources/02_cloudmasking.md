---
layout: page
title:  "Cloud Masking"
parent: "Resources"
nav_order: 2
---

# Cloud Masking Example Scripts

In the <a href="https://code.earthengine.google.com/?accept_repo=users/ee-scripts/Liberia_Forest_SIG_workshops" target="_blank" rel="noopener noreferrer">GEE repository</a> for this workshop, you will find a folder called `cloudMasking_examples` containing some scripts with example cloud masking scripts called `HLS_cloudMasking`, `Planet_cloudMasking`, and `Sentinel_Landsat_cloudMasking`. You can use these scripts to compare various cloud masking techniques with the available optical imagery.

## Sentinel and Landsat

```javascript
// setup
// -------------------------------------------------------------------

// threshold for percent cloud cover in a single scene
var cloudCoverThreshold = 80

// dates of interest
var d1 = '2018-1-1'
var d2 = '2018-12-31'

// center the map on the AOI
Map.centerObject(geometry, 7);

// imagery
// -------------------------------------------------------------------

// import cloud score +
var csPlus = ee.ImageCollection('GOOGLE/CLOUD_SCORE_PLUS/V1/S2_HARMONIZED')
  .filterDate(d1, d2)
  .filterBounds(geometry)

// import sentinel 2
var S2collection = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
  // filter for dates of interest
  .filterDate(d1, d2)
  // filter for images that intersect with the AOI
  .filterBounds(geometry)
  // link the collection to the Cloud Score + collection
  .linkCollection(csPlus, ['cs_cdf'])
  // filter for images with low cloud cover
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE',cloudCoverThreshold))
  // mask clouds using the functions defined at the end of the script
  .map(maskS2cloudsCS)
  .map(maskS2clouds)
  // sort by cloud cover
  .sort('CLOUDY_PIXEL_PERCENTAGE',false)
  
// set visualization parameters
var S2vis = {
  min: 0.0,
  max: 0.3,
  bands: ['B4', 'B3', 'B2'],
};

// Map.setCenter(83.277, 17.7009, 12);

Map.addLayer(S2collection.median().clip(geometry), S2vis, 'Sentinel 2');
// Map.addLayer(TOA, visualization, 'TOA');

/////////////////////////////////////////////////////////////////////////////////////////

var L8collection = ee.ImageCollection('LANDSAT/LC08/C02/T1_L2')
    .filterDate(d1,d2)
    .filterBounds(geometry)
    .filter(ee.Filter.lt('CLOUD_COVER', cloudCoverThreshold))
    .map(maskL8clouds)
    .map(applyScaleFactors)
    .sort('CLOUD_COVER', false) 

var L8vis = {
  bands: ['SR_B4', 'SR_B3', 'SR_B2'],
  min: 0.0,
  max: 0.3,
};

Map.addLayer(L8collection.median().clip(geometry), L8vis, 'Landsat 8');

// functions
// -------------------------------------------------------------------

// mask clouds from Sentinel-2 images using QA band
function maskS2clouds(image) {
  // define the quality score band to use
  var qa = image.select('QA60');
  // select the bits of interest
  // Bits 10 and 11 are clouds and cirrus
  var cloudBitMask = 1 << 10;
  var cirrusBitMask = 1 << 11;
  // create a mask where values are 0
  var mask = qa.bitwiseAnd(cloudBitMask).eq(0)
      .and(qa.bitwiseAnd(cirrusBitMask).eq(0));
      // Apply mask
  return image.updateMask(mask).divide(10000);
}

// mask clouds from Sentinel-2 images using Cloud Score +
function maskS2cloudsCS(image) {
  // select the quality score band
  var qa = image.select('cs_cdf');
  // create a mask with pixels with high quality scores
  var mask = qa.gte(0.80)
  // Apply mask
  return image.updateMask(mask);
}

// mask clouds from Landsat-8 images using QA band
function maskL8clouds(image) {
  // define the quality score band to use
  var qaBand = image.select('QA_PIXEL');
  // select the bits of interest
  // Bits 1,2,3,4 are cloud-related 
  // The << operator is the "bitwise shift" operator, shifting 1 to the left by the specified number of bits
  var dilatedCloudBitMask = 1 << 1;  // 1 shifted left by 1 bit  = 00000010 = 2  (dilated clouds)
  var cirrusBitMask = 1 << 2;        // 1 shifted left by 2 bits = 00000100 = 4  (cirrus clouds)
  var cloudBitMask = 1 << 3;         // 1 shifted left by 3 bits = 00001000 = 8  (clouds)
  var cloudShadowBitMask = 1 << 4;   // 1 shifted left by 4 bits = 00010000 = 16 (cloud shadow)
  // create a mask where values are 0
  var mask = qaBand.bitwiseAnd(dilatedCloudBitMask).eq(0)
            .and(qaBand.bitwiseAnd(cirrusBitMask).eq(0))
            .and(qaBand.bitwiseAnd(cloudBitMask).eq(0))
            .and(qaBand.bitwiseAnd(cloudShadowBitMask).eq(0));
  // Apply mask
  return image.updateMask(mask);
}

// Apply scaling factors to Landsat-8 images
function applyScaleFactors(image) {
  // sclae optical bands
  var opticalBands = image.select('SR_B.').multiply(0.0000275).add(-0.2);
  // scale thermal bands
  var thermalBands = image.select('ST_B.*').multiply(0.00341802).add(149.0);
  // return new bands
  return image.addBands(opticalBands, null, true)
              .addBands(thermalBands, null, true);
}
```

## Harmonized Landsat Sentinel

```javascript
// setup
// -------------------------------------------------------------------

// threshold for percent cloud cover in a single scene
var cloudCoverThreshold = 100

// dates of interest
var d1 = '2016-1-1'
var d2 = '2016-12-31'

// center the map on the AOI
Map.centerObject(geometry, 7);

// imagery
// -------------------------------------------------------------------

// import Landsat HLS
var landsatHLS = ee.ImageCollection("NASA/HLS/HLSL30/v002")
  // filter for images that intersect with the AOI
  .filterBounds(geometry)
  // filter for dates of interest
  .filter(ee.Filter.date(d1, d2))
  // filter for images with low cloud cover
  .filter(ee.Filter.lt('CLOUD_COVERAGE',cloudCoverThreshold))
  // mask clouds using the function defined at the end of the script
  .map(maskHSLclouds)
  // sort by cloud cover
  .sort('CLOUD_COVERAGE', false)
  // select and rename bands
  .select(['B2',    'B3',    'B4',    'B5',    'B6',    'B7',   'Fmask'], 
          ['blue',  'green', 'red',   'NIR',   'SWIR1', 'SWIR2','Fmask']);

// add the median composite to the map
Map.addLayer(landsatHLS.median().clip(geometry), {
  bands: ['red', 'green', 'blue'],
  min:0.01,
  max:0.18,
}, 'Landsat HLS');

// import Sentinel HLS
var sentinelHLS = ee.ImageCollection("NASA/HLS/HLSS30/v002")
    .filterBounds(geometry)
    .filter(ee.Filter.date(d1, d2))
    .filter(ee.Filter.lt('CLOUD_COVERAGE',cloudCoverThreshold))
    .map(maskHSLclouds)
    .sort('CLOUD_COVERAGE', false)
    .select(['B2',  'B3',   'B4', 'B8', 'B11',  'B12',  'Fmask'], 
            ['blue','green','red','NIR','SWIR1','SWIR2','Fmask'])

// add the median composite to the map
Map.addLayer(sentinelHLS.median().clip(geometry), {
  bands: ['red', 'green', 'blue'],
  min:0.01,
  max:0.18,
}, 'Sentinel HLS');

// merge the two HLS image collections
var combinedHLS = landsatHLS.merge(sentinelHLS)

// add the median composite to the map
Map.addLayer(combinedHLS.median().clip(geometry), {
  bands: ['red', 'green', 'blue'],
  min:0.01,
  max:0.18,
}, 'combined HLS');

// functions
// -------------------------------------------------------------------

// write a function for masking clouds from HLS images
function maskHSLclouds(image) {
  // define the QA band
  var qa = image.select('Fmask'); 
  // extract the bits related to cloud sin the QA band
  var cloudBitMask = 1 << 1;  
  var cloudAdjBitMask = 1 << 2; 
  var cloudShadowBitMask = 1 << 3; 
  // slect only pixels that are not clouds
  var mask = qa.bitwiseAnd(cloudBitMask).eq(0)
            .and(qa.bitwiseAnd(cloudShadowBitMask).eq(0))
            // .and(qa.bitwiseAnd(cloudAdjBitMask).eq(0));
  // return the cloud masked image
  return image.updateMask(mask);
}
```

## Planet

```javascript
// setup
// -------------------------------------------------------------------

// threshold for percent cloud cover in a single scene
var cloudCoverThreshold = 100

// dates of interest
var d1 = '2016-1-1'
var d2 = '2016-12-31'

// center the map on the AOI
Map.centerObject(geometry, 7);

// imagery
// -------------------------------------------------------------------

// import Planet NICFI
// This collection is not publicly accessible. To sign up for access,
// please see https://developers.planet.com/docs/integrations/gee/nicfi
var nicfiCollection = ee.ImageCollection('projects/planet-nicfi/assets/basemaps/africa');

// preprocess the image collection
var nicfi = nicfiCollection
  // filter for images that intersect with the AOI
  .filterBounds(geometry)
  // filter for dates of interest
  .filter(ee.Filter.date(d1, d2))
  // mask out clouds 
  .map(function(image) {
    // select and rename the bands we want to use
    var blue = image.select('B'); 
    var green = image.select('G');  
    var red = image.select('R'); 
    var NIR = image.select('N');
    // calculate a brightness value by averaging all the bands
    var brightness = image.expression(
    '(b1 + b2 + b3 + b4) / 4', {
      'b1': blue, 'b2': green, 'b3': red, 'b4': NIR
    })
    // Create a cloud mask by setting thresholds for each band (high reflectance in all bands suggests clouds)
    var cloudMask = blue
    .lt(1100)  
    .and(green.lt(1600))  
    .and(red.lt(2200)) 
    // .and(NIR.lt(3500)) 
    .and(brightness.lt(2200));  
    // Apply the cloud mask to the image
    return image.updateMask(cloudMask).addBands(brightness);
  })
  // get the median composite
  .median()
  // clip to AOI
  .clip(geometry)

// define visualization parameters
var vis = {'bands':['R','G','B'],'min':64,'max':5454,'gamma':1.8};
var NDVIvis = {min:-0.55,max:0.8,palette: ['8bc4f9', 'c9995c', 'c7d270','8add60','097210']};

// add to map
Map.addLayer(
  nicfi, 
  vis, 
  '2021-03 mosaic');
  
// calculate NDVI and add to map
Map.addLayer(
  nicfi.normalizedDifference(['N','R']).rename('NDVI'),
  NDVIvis, 
  'NDVI');
```
