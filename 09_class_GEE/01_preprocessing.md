---
layout: page
title:  "Preprocessing Imagery"
parent: "9. Classification with GEE"
nav_order: 1
---

*This page has been updated since the workshop*

*The updated version does not include Planet NICFI imagery, but instead you have the option to include Google Embeddings*

*This version also assume you are using CEO data, not making dummy reference data.'


# Preprocessing Imagery in GEE

Open up the script you named `1 preprocessing`. You will copy and paste each code block into the empty script. You can check your work by looking at the following script `users/ee-scripts/Liberia_Forest_SIG_workshops/09_classification_GEE/1 preprocessing`.

## Setup

### Set Important Parameters

First, we  define some variables that will be used as parameters later throughout the script. We are bringing them to the top of the script so they are easy to change without having to scroll through the script to find them.

These include values related to setting the version number, time period of interest, the number of reference points, cloud masking, and smoothing.

```javascript
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////
// Define Parameters
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////

// General
// ------------------------------------------------------------------------------------------

// version
var version = 3;

// dates of interest
var d1 = '2024-1-1'
var d2 = '2024-12-31'

// basemap
Map.setOptions('SATELLITE')

// final map resolution
var resolution = 30;

// Decide whether or not to use annual Google Embeddings with this true/false flag
// ------------------------------------------------------------------------------------------

// ---- Feature flags, choosing to use Google Embeddings or not ----
var USE_GE = true; // toggle Google Embeddings on/off 
//final predictor image will result in 31 bands if false and 95 bands if true
  // USE_GE = true + year ≥ 2017 → GE (A00…A63) is included.
  // USE_GE = true + year < 2017 → your If(...) returns an empty image, so no GE bands (by design due to data availability). 
  // USE_GE = false → GE is not added at all, regardless of year. (may chose to do this if the results prove not helpful)

// SAR 
// ------------------------------------------------------------------------------------------

// smoothing radius for SAR (m)
var smoothingRadius = 50;

// incidence angles for SAR
var angles = ee.List([20,45]);

// Optical
// ------------------------------------------------------------------------------------------

// cloud masking parameters

// threshold for percent cloud cover in a single scene
var cloudCoverThreshold = 100

// threshold for Sentinel 2 - Cloud Score+ quality score 
// values between 0.50 and 0.65 generally work well
// Higher values will remove thin clouds, haze & cirrus shadows
var csQualityThreshold = 0.80
```

### Create Area of Interest (AOI)

An area of interest can be uploaded from a local shapefile, drawn on the map, or derived from a pre-existing dataset in the Earth Engine catalogue.

For this exercise, we use a union of a `featureCollection` of a simple polygon around Liberia's borders to be our AOI. We can also import the exact Liberia borders and filter for specific provinces by uncommenting the lines below.

```javascript
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////
// Define AOI
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////

// import the simple Liberia feature collection
var Liberia = ee.FeatureCollection("projects/pc556-ncs-liberia-forest-mang/assets/Liberia_simple")
  
// define an aoi from the feature collection
var aoi = Liberia
  .union();
  
// Add the aoi object as a layer to the map
Map.addLayer(aoi, {}, 'AOI', false);

// alternatively, use a featureCollection of administrative borders or draw your own
// https://developers.google.com/earth-engine/datasets/catalog/FAO_GAUL_2015_level1

// // import the Liberia borders feature collection
// var Liberia = ee.FeatureCollection("projects/pc556-ncs-liberia-forest-mang/assets/LBR_county_updatedProj")

// // print out the county names
// print('province names:', Liberia.aggregate_array('County').distinct())
  
// // define an aoi from the feature collection
// var aoi = Liberia
//   // or select one or a few counties
//   // (uncomment this line to select a specific set of provinces)
//   // .filter(ee.Filter.inList('County', ['Bong','Gbarpolu']))
//   // get the union of all selected counties
//   .union();

// // Add the aoi object as a layer to the map
// Map.addLayer(aoi, {}, 'AOI', false);
```

<img align="center" src="../images/class-gee/aoi2.png" hspace="15" vspace="10" width="400">

If we wanted to filter for a specific province, we can check the province names by printing them to the **Console** tab from our Liberia borders `featureCollection`.

# <img align="center" src="../images/class-gee/print_provinces.png" hspace="15" vspace="10" width="200">


## Using the Original Land Use / Land Cover (LULC) Map

First, let's import the current 2014 LULC map for Liberia. We will use this map to produce reference data for our model and to use as a visual comparison while we go through the model building process.
> *Note: We do not NEED an origianl LULC map, but it is a quick way for us to quickly get relatively trustworthy samples for the purposes of this workshop.*

We need to make sure the cloud and nodata classes both receive values of 0 and then mask them out to effectively remove them from the map so we do not sample them.

We will also symbolize the LULC classes with appropriate colors and add them to the map. 

```javascript
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////
// Import and Prepare the Original LULC Map for Sample Extraction
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////

// LULC 
// ------------------------------------------------------------------------------------------

// import images
var lulc10m = ee.Image(
  'projects/pc556-ncs-liberia-forest-mang/assets/Liberia_landcover_forest_map_10m_v1_2014') //we will use this one with 10m resolution
var lulc30m = ee.Image(
  'projects/pc556-ncs-liberia-forest-mang/assets/Liberia_landcover_forest_map_30m_2014')

// // check cloud class
// var clouds = lulc.eq(25).selfMask()
// Map.addLayer(clouds,{},'clouds')
  
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

```

<img align="center" src="../images/class-gee/LULC2014.png" hspace="15" vspace="10" width="400">

We can view the LULC class at any point on the map by opening the **Inspector** tab in the upper right hand corner of the screen and clicking somewhere on the map. Then, we can navigate to `Pixels` > `LULC 2014` > `class` to see the LULC class at that point.

<img align="center" src="../images/class-gee/inspector_tab.png" hspace="15" vspace="10" width="200">

## Import and Preprocess Imagery

Now, we  import and preprocess all the `ImageCollections` we will use for our classification, including the LULC map we generated the reference data from and the satellite imagery on which we will run the model.

Before beginning any remote sensing workflow, image preprocessing is essential. We have to ensure we use high quality data that represents the kind of information we need for our anlaysis. Many of the Image Collections in the GEE catalogue have undergone the more complex pieces of preprocessing, such as georeferencing and terrain, radiometric, and atmospheric correction. However, we still typically need to do the following:
* filter for the area of interest
* filter for the time period of interest (with consideration for seasonality and data availability)
* filter for certain image properties (such as orbit direction or sensor angle)
* filter for the bands of interest (with consideration for what we are trying to map)
* calculate indices
* smooth noisy imagery (SAR imagery)
* mask out clouds (optical imagery)

It is generally better to do as much filtering as we can at the beginning of our analysis to reduce the size of our data sets. This reduces the computational demands of the script.

### Elevation (DEM)

Next, let's import elevation data, which might be particularly useful in our classification for LULC types that are strongly influence by elevation. We will use the Copernicus 30m resolution DEM. 

Before importing this data set from the GEE data catalogue, we can preview important information about it by searching for 'Copernicus DEM' into the search bar above the code editor and clicking on **Copernicus DEM GLO-30: Global 30m Digital Elevation Model**. We can see how it was produced and who produced it. We can click on **See example** to see an example script using the data set, and we can click on **BANDS** to see the resolution and band descriptions.

<img align="center" src="../images/class-gee/DEM_info.png" hspace="15" vspace="10" width="400">

We symbolize and add this to the map as well.

```javascript
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////
// Import and Preprocess Imagery
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////

// DEM 
// ------------------------------------------------------------------------------------------

// import Copernicus DEM image collection
var demCollection = ee.ImageCollection('COPERNICUS/DEM/GLO30')

// print the resolution
print('DEM resolution (m):',demCollection.first().select('DEM').projection().nominalScale())

// do some preprocessing
var dem = demCollection
  // select just the DEM bands
  .select('DEM')
  // mosaic all images from the image collection into a single image
  .mosaic()
  // clip to the AOI
  .clip(aoi);

// define visualization paramaters
var demVis = {
  min: 0,
  max: 600,
  palette: ['0000ff','00ffff','ffff00','ff0000','ffffff'],
};
// add to the map
Map.addLayer(dem, demVis, 'DEM', false);
```

### Synthetic Aperture Radar (SAR) Imagery (PALSAR and Sentinel-1)

Now, we start importing the raw satellite imagery we will use in our model, starting with synthetic aperture radar (SAR) imagery. We need to do some filtering and preprocessing for the SAR imagery before we use it in our model. SAR can be more complex to interpret and analyze, but it can be very useful in tropical areas where cloud cover makes optical imagery difficult to use. 

>*Details on SAR:* 
>>*Resource:* For some background on SAR data, you can go to the <a href="https://sig-gis.github.io/Liberia_NSC_ForestMonitoringTraining/11_resources/01_sar.html" target="_blank" rel="noopener noreferrer">SAR section of the Resources page</a> on this website, and more extensive SAR materials can be found at this <a href="https://learnsar.open.uaf.edu/sar-resources/" target="_blank" rel="noopener noreferrer">website</a>.

>>In general, for forest mapping in tropical West Africa, the following types of SAR images are most useful, so we will use these to guide our decisions:
>>* Descending orbit pass 
>>* Interferometric Wide swath mode
>>* Moderate Incidence Angles (20° to 45°)
>>* C-band SAR and L-Band (e.g. Sentinel-1, ALOS PALSAR)
>>* VV/VH & HH/HV dual polarization

In this part of the script, we do several things. We will use both PALSAR (Phased Array L-band Synthetic Aperture Radar) and Sentinel-1 so that we can work with both L and C-band SAR data (both good for forest related analyses). 
> L-band provides better penetration depth into vegetation, while C-band provides higher spatial resolution. 

However, both of these data sets are only available starting in 2015, so we write functions for the importing and preprocessing of each data set, and then we call these functions only if our time period of interest is after 2015. Within each importing and preprocessing function, we:

* filter for images that match our area of interest 
* filter for our dates of interest
* filter for sensor angle
* filter for sensor mode
* select polarization bands
* composite the image (combine all the images from the time period into a single image by calculating the median of all values present at each pixel)
* clip th eimage to the AOI
* scale values to decibels (if needed)
* smooth out the image (using a focal mean function).


**Note**: Parts of the following script will produce an error if your year of interest defined at the top of your script is before 2015, because Sentinel-1 data is not available until then.

```javascript
// SAR
// ------------------------------------------------------------------------------------------

// general properties of SAR usually used for forest mapping in Liberia:
  // Descending orbit pass 
  // Interferometric Wide swath mode
  // Moderate Incidence Angles (20° to 45°)
  // C-band SAR and L-Band (e.g. Sentinel-1, ALOS PALSAR)
  // VV/VH & HH/HV dual polarization

// import and preprocess SAR imagery based on the dates of interest
var sar = ee.Image(
  // if the date range is 2015-present, use SAR imagery
  ee.Algorithms.If(
    ee.Date(d1).millis()
      .gte(ee.Date('2015-01-01').millis()),
    // call function to import and preprocess Sentinel 1 and PALSAR
    ee.Image(importS1(d1,d2))
      .addBands(ee.Image(importPALSAR(d1,d2))),
    // if the date range is outside of 2015-present, return an empty image
    ee.Image(0)            
  )
);

// define visualization parameters
var s1vis = {bands:['VH_C_S1'],min:-20,max:0};
var palsarvis = {bands:['HV_L_P'],min:-20,max:0};
// add to map
Map.addLayer(sar, palsarvis,'SAR HV from PALSAR',false);
Map.addLayer(sar, s1vis,'SAR VH from S1',false);

// functions are defined below:

// ===== Import Sentinel-1 (C-band) =====
function importS1(date1, date2) {
  var s1 = ee.ImageCollection('COPERNICUS/S1_GRD')
    .filterBounds(aoi)
    .filterDate(date1, date2)
    .filter(ee.Filter.eq('instrumentMode', 'IW'))
    .map(function(image) {
      var angle = image.select('angle');
      return image.updateMask(angle.gte(20).and(angle.lte(45)));
    })
    .select(['VV', 'VH'])
    .median()
    .focal_mean(50, 'circle', 'meters')
    .clip(aoi);

  // Rename bands with new convention
  return s1.rename(['VV_C_S1', 'VH_C_S1']);
}

// ===== Import PALSAR (L-band) =====
function importPALSAR(date1, date2) {
  var palsar = ee.ImageCollection('JAXA/ALOS/PALSAR-2/Level2_2/ScanSAR')
    .filterBounds(aoi)
    .filterDate(date1, date2)
    .filter(ee.Filter.gte('IncAngleNearRange', 0))
    .filter(ee.Filter.lte('IncAngleFarRange', 90))
    .select(['HH', 'HV'])
    .median()
    .pow(2).log10().multiply(10).subtract(83.0) // Convert DN to gamma0 backscatter in dB using JAXA's scaling factor
    .focal_mean(50, 'circle', 'meters')
    .clip(aoi);

  // Rename bands with new convention
  return palsar.rename(['HH_L_P', 'HV_L_P']);
}

  // Radar Vegetation Index (RVI) is a measure used to assess the density and health of vegetation

  // S1 RVI may be extraneous and is more suited to disturbance mapping than LC mapping, but keeping it for now
          // PALSAR RVI is better suited for land cover mapping than Sentinel-1 RVI because 
          // its L-band wavelength penetrates deeper into vegetation canopies, 
          // providing stronger sensitivity to forest structure and biomass
      
  //for Dual-Polarimetric Data (e.g., HH and HV or VV and VH) a simplified version of the RVI is used
  //    RVI = (4 × cross-pol) / (co-pol + cross-pol)
  // (https://documentation.dataspace.copernicus.eu/APIs/openEO/openeo-community-examples/python/RVI/RVI.html)
  // Suitable for Sentinel-1 (VV/VH) or PALSAR-2 ScanSAR (HH/HV)
  
      // Where:
      //    - cross-pol = HV (for HH/HV systems) or VH (for VV/VH systems)
      //    - co-pol    = HH (for HH/HV systems) or VV (for VV/VH systems)
      //
      // This index ranges from 0 to 1+ and increases with vegetation volume (can exceed 1 in some high-scatter vegetation)
      // and structural complexity. Higher values typically indicate dense
      // or complex vegetation canopies.
      //
      // Example (PALSAR, using HH & HV):
      //    RVI = (4 × HV) / (HH + HV)
      //
      // Example (Sentinel-1, using VV & VH):
      //    RVI = (4 × VH) / (VV + VH)
  
  // The traditional RVI formula for Full-Polarimetric Data (HH, HV, VV) is below
  // RVI: (8*HV) / (HH + VV + (2*HV))
          // We don't want to use this one because we'd have to mix sensors
          
          
// ===== RVI from Sentinel-1 (C-band) =====
function calculateRVI_S1(image) {
  var vv = ee.Image(10).pow(image.select('VV_C_S1').divide(10));
  var vh = ee.Image(10).pow(image.select('VH_C_S1').divide(10));
  var rvi = vh.multiply(4).divide(vv.add(vh)).rename('RVI_C_S1'); //simplified RVI
  return rvi;
}

// ===== RVI from PALSAR (L-band) =====
// Radar Vegetation Index
// used to quantify vegetation structure and density
function calculateRVI_PALSAR(image) {
  var hh = ee.Image(10).pow(image.select('HH_L_P').divide(10));
  var hv = ee.Image(10).pow(image.select('HV_L_P').divide(10));
  var rvi = hv.multiply(4).divide(hh.add(hv)).rename('RVI_L_P'); //simplified RVI
  return rvi;
}

// ===== Compose SAR image =====
var s1 = importS1(d1, d2);
var palsar = importPALSAR(d1, d2);

// Merge bands
var sar = s1.addBands(palsar);

// Compute RVIs
var rviC = calculateRVI_S1(s1); //optionally comment out
var rviL = calculateRVI_PALSAR(palsar);

// Add RVIs to SAR image
sar = sar.addBands(rviL).addBands(rviC); //optionally comment out rviC

```

<img align="center" src="../images/class-gee/SAR.png" hspace="15" vspace="10" width="400">



### Optical Imagery (Landsat 8 and Sentinel-2)

Now we will do something very similar with optical imagery. Optical imagery is much more intuitive to interpret and less noisy than SAR, but it is frequently obscured by clouds in tropical regions.

<img align="center" src="../images/class-gee/landsat_sentinel_timeline.png" hspace="15" vspace="10" width="600">

We will use either Landsat 8 or Harmonized Landsat-Sentinel (HLS) imagery, depending on the dates of interest. HLS combines Landsat and Sentinel-2 imagery to create cloud-free imagery with a higher spatial coverage and higher temporal resolution than is otherwise possible with only one of these sensors. It is only available starting in 2016, so we write functions for the importing and preprocessing of each data set, and then we call the Landsat function if our time period of interest is before 2016 and call the HLS function if our time period of interest is after 2016. Within each importing and preprocessing function, we:

* filter for images that match our area of interest
* filter for our dates of interest
* filter out images with high cloud cover
* mask out clouds (using the `QA_PIXEL` band for Landsat and the `Fmask` band for HLS)
* select and rename bands (so the band names are consistent regardless of which imagery source we use)
* composite the image (using median)
* clip the image to the AOI
* scale values to reflectance (if needed)

<img align="center" src="../images/class-gee/HLS_info.png" hspace="15" vspace="10" width="400">

*Resource:* For some background on cloud masking, you can go to the <a href="https://sig-gis.github.io/Liberia_NSC_ForestMonitoringTraining/11_resources/02_cloudmasking.html" target="_blank" rel="noopener noreferrer">Cloud Masking section of the Resources page</a> on this website.

```javascript
// Optical
// ------------------------------------------------------------------------------------------

// import and preprocess optical imagery based on the dates of interest
var optical = ee.Image(
  // if the date range is 2013-2015, use Landsat 8
  ee.Algorithms.If(
    ee.Date(d2).millis()
      .lt(ee.Date('2016-01-01').millis())
      .and(ee.Date(d1).millis()
        .gte(ee.Date('2013-01-01').millis())),
    // call function to import and preprocess Landsat 8
    importL8(d1,d2),
    // if the date range is 2016-present, use HLS
    ee.Algorithms.If(
      ee.Date(d1).millis()
        .gte(ee.Date('2016-01-01').millis()),   
      // call function to import and preprocess HLS
      importHLS(d1,d2).addBands(importL8(d1,d2)), 
    // if the date range is outside of 2013-present, return an empty image
    ee.Image(0)            
  )
));

// define visualization parameters
var opticalVis = {bands:['red_L8','green_L8','blue_L8'],min:0,max:0.3};
// add to map
Map.addLayer(optical, opticalVis,'optical L8',false);

// functions are defined below:

// Landsat 8 (for 2013-2015)
// -----------------------------------------

// write a function to import and preprocess Landsat 8 
function importL8(date1,date2){
  
  // import the Landsat 8 image collection
  var l8Collection = ee.ImageCollection("LANDSAT/LC08/C02/T1_L2")
  
  // print the resolution
  print('Landsat-8 resolution (m):',l8Collection.first().select(1).projection().nominalScale())
  print("Example Landsat-8 images:", l8Collection.limit(3))
  
  // do some preprocessing to make an image from the Sentinel 8 image collection
  var l8 = l8Collection
    // select images that intersect with your AOI
    .filterBounds(aoi)
    // select images in your dates of interest
    .filterDate(date1, date2)
    .filter(ee.Filter.lt('CLOUD_COVER', cloudCoverThreshold))
    // mask out cloudy pixels
    .map(function(image) {
      // define the quality score band to use
      var qaBand = image.select('QA_PIXEL');
      // select the bits of interest
      // Bits 1,2,3,4 are cloud-related 
      // The << operator is the "bitwise shift" operator, shifting 1 to the left by the specified number of bits
      var dilatedCloudBitMask = 1 << 1;  // 1 shifted left by 1 bit  = 00000010 = 2  (dilated clouds)
      var cirrusBitMask = 1 << 2;        // 1 shifted left by 2 bits = 00000100 = 4  (cirrus clouds)
      var cloudBitMask = 1 << 3;         // 1 shifted left by 3 bits = 00001000 = 8  (clouds)
      var cloudShadowBitMask = 1 << 4;   // 1 shifted left by 4 bits = 00010000 = 16 (cloud shadow)
      // Transform bits and create a mask where values are 0
      var mask = qaBand.bitwiseAnd(dilatedCloudBitMask).eq(0)
                .and(qaBand.bitwiseAnd(cirrusBitMask).eq(0))
                .and(qaBand.bitwiseAnd(cloudBitMask).eq(0))
                .and(qaBand.bitwiseAnd(cloudShadowBitMask).eq(0));
      // Apply mask
      return image.updateMask(mask);
    })
    // select and rename desired bands
    .select(['SR_B2', 'SR_B3', 'SR_B4', 'SR_B5', 'SR_B6', 'SR_B7'], 
            ['blue_L8',  'green_L8', 'red_L8',   'NIR_L8',   'SWIR1_L8', 'SWIR2_L8',])
    // get the median for the time period of interest
    .median()
    // convert to reflectance values
    .multiply(0.0000275).add(-0.2)
    // clip to the AOI
    .clip(aoi);
  
  // return the composited and cleaned Sentinel 2 image
  return(l8)
}

// Harmonized Landsat Sentinel combined (for 2016-present)
// -----------------------------------------

// write a function to import and preprocess HLS 
function importHLS(date1,date2){
  
  // import the Landsat 8 HLS image collection
  var l8Collection = ee.ImageCollection("NASA/HLS/HLSL30/v002")
    // select and rename desired bands
    .select(['B2',    'B3',    'B4',    'B5',    'B6',    'B7',   'Fmask'], 
            ['blue_HLS',  'green_HLS', 'red_HLS',   'NIR_HLS',   'SWIR1_HLS', 'SWIR2_HLS','Fmask_HLS'])
  // import the Sentinel 2 HLS image collection
  var s2Collection = ee.ImageCollection("NASA/HLS/HLSS30/v002")
    // .select(['B2',  'B3',   'B4', 'B5',      'B6',      'B7',      'B8', 'B8A',     'B11',  'B12',  'Fmask'], 
    //         ['blue','green','red','redEdge1','redEdge2','redEdge3','NIR','redEdge4','SWIR1','SWIR2','Fmask'])
    .select(['B2',  'B3',   'B4', 'B8', 'B11',  'B12',  'Fmask'], 
            ['blue_HLS','green_HLS','red_HLS','NIR_HLS','SWIR1_HLS','SWIR2_HLS','Fmask_HLS'])
  
  // merge the two collections into one big HLS image collection
  var hlsCollection = l8Collection.merge(s2Collection);
  
  // print the resolution
  print('HLS resolution (m):',hlsCollection.first().select(1).projection().nominalScale())
  print("Example HLS images:", hlsCollection.limit(3))
  
  // do some preprocessing to make an image from the HLS image collection
  var hls = hlsCollection
    // select images that intersect with your AOI
    .filterBounds(aoi)
    // select images in your dates of interest
    .filterDate(date1, date2)
    .filter(ee.Filter.lt('CLOUD_COVERAGE', cloudCoverThreshold))
    // mask out cloudy pixels
    .map(function(image) {
      // define the quality score band to use
      var qaBand = image.select('Fmask_HLS');
      // select the bits of interest
      // Bits 1,2,3,4 are cloud-related 
      // The << operator is the "bitwise shift" operator, shifting 1 to the left by the specified number of bits
      var cloudBitMask = 1 << 1;  
      var cloudAdjBitMask = 1 << 2; 
      var cloudShadowBitMask = 1 << 3; 
      var mask = qaBand.bitwiseAnd(cloudBitMask).eq(0)
                .and(qaBand.bitwiseAnd(cloudShadowBitMask).eq(0));
                // (uncomment this to remove cloud adjacent pixels)
                // .and(qaBand.bitwiseAnd(cloudAdjBitMask).eq(0));
      // Apply mask
      return image.updateMask(mask);
    })
    // select the bands of interest
    .select(['blue_HLS','green_HLS','red_HLS','NIR_HLS','SWIR1_HLS','SWIR2_HLS'])
    // get the median for the time period of interest
    .median()
    // clip to the AOI
    .clip(aoi)
  
  // return the composited and cleaned HLS image
  return(hls)
}

```

<img align="center" src="../images/class-gee/optical.png" hspace="15" vspace="10" width="400">

### Google Embeddings
This annual dataset is available 2017 onward. This is a powerful 10-meter pixel dataset in a 64-dimensional representation, or "embedding vector".

* filter for our dates of interest (annual images)
* clip the image to the AOI

```javascript
// Google Embeddings (for 2017-present)
// -----------------------------------------

// import and preprocess Google Embedding based on the dates of interest
var GE = ee.Image(
  // if the date range is 2015-present, use planet imagery
  ee.Algorithms.If(
    ee.Date(d1).millis()
      .gte(ee.Date('2017-01-01').millis()),
    // call function to import and preprocess Planet
    ee.Image(importGE(d1,d2)),
    // if the date range is outside of 2017-present, return an empty image
    ee.Image(0)            
  )
);


// ===== Import embeddings =====

// Google Embeddings importer
function importGE(date1, date2) {
  return ee.ImageCollection('GOOGLE/SATELLITE_EMBEDDING/V1/ANNUAL')
    .filterBounds(aoi)
    .filterDate(date1, date2)
    .select([
      'A00','A01','A02','A03','A04','A05','A06','A07','A08','A09','A10','A11','A12','A13','A14','A15',
      'A16','A17','A18','A19','A20','A21','A22','A23','A24','A25','A26','A27','A28','A29','A30','A31',
      'A32','A33','A34','A35','A36','A37','A38','A39','A40','A41','A42','A43','A44','A45','A46','A47',
      'A48','A49','A50','A51','A52','A53','A54','A55','A56','A57','A58','A59','A60','A61','A62','A63'
    ])
    .median()
    .clip(aoi);
}

```

## Prepare Predictor Image
The *Predictor Image* is the image data that is being analyzed by the model to generate a prediction. Said another way, it is the satellite imagery for the year for which we are trying to produce a LULC map.

Now that we have multiple composted images containing the different bands we would like to use as predictor variables in our random forest model. We combine all these bands into a single image, the Predictor Image. 

>**Note:**
>> The **Training Image** and the **Predictor Image** may be the same or different. In our case we are using the same image from which we are extracting the variables for the training samples, as we are for giving the Random Forest for the prediction of all the pixels. However, as long as the bands are the same between the two images, you can use training data from one image and apply it to a different image (e.g., use training points from a 2016 Training Image to classify a 2018 Preditor Image). You could not train a Random Forest classifier on a Training Image with fewer variables than the Predictor Image and expect it to be able to utilize all the information (e.g. training on 2014 with only Landsat and use the Random Forest classifier on a 2018 image with Landsat, Sentinel-2, and SAR).

Inclusiong of the Google Embeddings is noted as optional, based on a true/false flag defined in the parameters. You may want to test your results with and without the embeddings. The dataset is very powerful and could potentially lead to overfitting or outweigh the importance of your other imagery bands. Observe the results and the importace of the bands in your later classification script. The default is to include this data.

```javascript
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////
// Prepare Predictor Image
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////


// Build GE only if enabled
var GE = ee.Image(0);
if (USE_GE) {
  GE = ee.Image(
    ee.Algorithms.If(
      ee.Date(d1).millis().gte(ee.Date('2017-01-01').millis()),
      importGE(d1, d2),   // <-- call the pure importer
      ee.Image(0)
    )
  );
}

  
// When building predImage -- // merge all images into a predictor image with many bands -- optional flag to remove embeddings if results are overfitted
var predImage = dem
  .addBands(sar)
  .addBands(optical);
if (USE_GE) predImage = predImage.addBands(GE); // only include Google Embeddings if marked (chosen in parameters at the top)

// remove the constant bands we created during the if algorithms
predImage = predImage
  .select(predImage
    .bandNames()
    .removeAll(['constant','constant_1','constant_2','constant_3']))

```

### Calculate Indices
The next thing we do is calculate some spectral indices from the optical imagery that are frequently used to identify LULC classes of interest. Certain land cover types strongly reflect or absorb different wavelengths of light, and we can calculate normalized versions of these spectral differences to highlight certain land cover types. Most of these index values range from -1 to +1.

* **NDVI:** Normalized Difference Vegetation Index - highlights vegetation health and density; calculated using the NIR and red bands

* **LSWI:** Land Surface Water Index - highlights vegetation and soil water content; calculated using the NIR and SWIR1 bands

* **NDMI:** Normalized Difference Moisture Index - highlights vegetation water content; calculated with the red and SWIR2 bands

* **MNDWI:** Modified Normalized Difference Water Index - highlights open water; calculated using the geen and SWIR bands

* **EVI:** Enhanced Vegetation Index - highlights vegetation health and density; calculated using the NIR, red, and blue bands

* **MVI:** Mangrove Vegetation Index - highlights mangroves; calculated using the NIR, green, and SWIR1 bands

* **RVI:** Radar Vegetation Index - highlights vegetation density and structure; calculated using the HV, HH, and VV polarizations

As we did before, we write separate functions for the optical and SAR indices, and only call the SAR function for the time period for which SAR is available.

*Resource:* For some background on spectral indices, you can go to the <a href="https://sig-gis.github.io/Liberia_NSC_ForestMonitoringTraining/11_resources/03_indices.html" target="_blank" rel="noopener noreferrer">Spectral Indices section of the Resources page</a> on this website.

```javascript
// functions defined below:

// write function to calculate indices
function calculateHLSIndices(image){
  
  // make sure the image is recognized as an image
  ee.Image(image)
  
  // calculate the indices and rename them
  
  // When we can, we use the GEE function normalizedDifference, expressed as: (b1-b2)/(b1+b2)
  // NDVI: (NIR-Red)/(NIR+Red)
  var ndvi = image.normalizedDifference(['NIR_HLS', 'red_HLS']).rename('NDVI_HLS');
  // LSWI: (NIR-SWIR1)/(NIR+SWIR1)
  var lswi = image.normalizedDifference(['NIR_HLS', 'SWIR1_HLS']).rename('LSWI_HLS');
  // NDMI: (SWIR2-Red)/(SWIR2+Red)
  var ndmi = image.normalizedDifference(['SWIR2_HLS', 'red_HLS']).rename('NDMI_HLS');
  // NDWI: (Green-NIR)/(Green+NIR)
  var ndwi = image.normalizedDifference(['green_HLS', 'NIR_HLS']).rename('NDWI_HLS');
  // MNDWI: (Green-SWIR2)/(Green+SWIR2)
  var mndwi = image.normalizedDifference(['green_HLS', 'SWIR2_HLS']).rename('MNDWI_HLS');
  
  // for the more complicated indices, we define the input bands separately first
  var nir = image.select('NIR_HLS');
  var red = image.select('red_HLS');
  var blue = image.select('blue_HLS');
  var green = image.select('green_HLS');
  var swir1 = image.select('SWIR1_HLS');
  // EVI: 2.5 * (NIR-red) / (NIR + (6*red) - (7.5*blue) + 1)
  var evi = nir.subtract(red).multiply(2.5)
    .divide(nir.add(red.multiply(6)).subtract(blue.multiply(7.5)).add(1))
    .rename('EVI_HLS');
  // MVI: (NIR-green)/(SWIR1-green)
  var mvi = nir.subtract(green).divide(swir1.subtract(green)).rename('MVI_HLS');

  // merge indices into a single image
  var indices = ndvi
    .addBands(lswi)
    .addBands(ndmi)
    // .addBands(ndwi)
    .addBands(mndwi)
    .addBands(evi)
    .addBands(mvi);
                    
  // return the index image
  return indices
}

// write function to calculate indices
function calculateL8Indices(image){
  
  // make sure the image is recognized as an image
  ee.Image(image)
  
  // calculate the indices and rename them
  
  // When we can, we use the GEE function normalizedDifference, expressed as: (b1-b2)/(b1+b2)
  // NDVI: (NIR-Red)/(NIR+Red)
  var ndvi = image.normalizedDifference(['NIR_L8', 'red_L8']).rename('NDVI_L8');
  // LSWI: (NIR-SWIR1)/(NIR+SWIR1)
  var lswi = image.normalizedDifference(['NIR_L8', 'SWIR1_L8']).rename('LSWI_L8');
  // NDMI: (SWIR2-Red)/(SWIR2+Red)
  var ndmi = image.normalizedDifference(['SWIR2_L8', 'red_L8']).rename('NDMI_L8');
  // NDWI: (Green-NIR)/(Green+NIR)
  var ndwi = image.normalizedDifference(['green_L8', 'NIR_L8']).rename('NDWI_L8');
  // MNDWI: (Green-SWIR2)/(Green+SWIR2)
  var mndwi = image.normalizedDifference(['green_L8', 'SWIR2_L8']).rename('MNDWI_L8');
  
  // for the more complicated indices, we define the input bands separately first
  var nir = image.select('NIR_L8');
  var red = image.select('red_L8');
  var blue = image.select('blue_L8');
  var green = image.select('green_L8');
  var swir1 = image.select('SWIR1_L8');
  // EVI: 2.5 * (NIR-red) / (NIR + (6*red) - (7.5*blue) + 1)
  var evi = nir.subtract(red).multiply(2.5)
    .divide(nir.add(red.multiply(6)).subtract(blue.multiply(7.5)).add(1))
    .rename('EVI_L8');
  // MVI: (NIR-green)/(SWIR1-green)
  var mvi = nir.subtract(green).divide(swir1.subtract(green)).rename('MVI_L8');

  // merge indices into a single image
  var indices = ndvi
    .addBands(lswi)
    .addBands(ndmi)
    // .addBands(ndwi)
    .addBands(mndwi)
    .addBands(evi)
    .addBands(mvi);
                    
  // return the index image
  return indices
}


// // write function to calculate indices
// function calculateSARIndices(image){
  
//   // make sure the image is recognized as an image
//   ee.Image(image) 
  
//   // calculate the indices and rename them
//   // for the more complicated indices, we define the input bands separately first
//   // for the SAR bands, we need to convert from decibels to a linear scale for the calculations
//   var vv = ee.Image(10).pow(image.select('VV').divide(10));
//   var vh = ee.Image(10).pow(image.select('VH').divide(10));
//   var hh = ee.Image(10).pow(image.select('HH').divide(10));
//   var hv = ee.Image(10).pow(image.select('HV').divide(10));
//   // RVI: (8*HV) / (HH + VV + (2*HV)) //Radar Vegetation Index (RVI) is a measure used to assess the density and health of vegetation
//   var rvi = hv.multiply(8).divide(hh.add(vv).add(hv.multiply(2))).rename('RVI');

//   // merge indices into a single image
//   var indices = rvi
                    
//   // return the index image
//   return indices
// }

```

*Resource:* Here is a great resource published by the University of Bonn for finding indeces for many different purposes: <a href="https://www.indexdatabase.de/" target="_blank" rel="noopener noreferrer">https://www.indexdatabase.de/</a>.


### Fix Projection Issues

If we look back at the resolutions we printed out, we see that the original imagery sources had different pixel sizes. It is best to reduce the resolution of all predictor variables to that of the lowest resolution imagery source. Thus we reduce the resolution to 30m, and we need to define a projection and reproject in order to do this.

```javascript
// Fix projection issues
// ------------------------------------------------------------------------------------------

// set the projection of the image
predImage = predImage.setDefaultProjection('EPSG:4326', null, resolution);
  
// reduce the resolution of all bands to the desired resolution
predImage = predImage
  .reduceResolution({
    reducer: ee.Reducer.mean(),
    bestEffort: true})  // Allows approximation for complex regions
  .reproject({
    crs: predImage.projection(),
    scale: resolution});
    
// print
print('predictor image:', predImage)


```

## Checking Our Work

Let's print out some of our results to double check we did everything correctly.

Add a few of the indeces we made to use for input variables of the predictor image.

```javascript
// add some of the features (a.k.a. variables or bands) from the preditor image to the map
// add to map
var ndviVis = {
  bands:['NDVI_L8'],
  min:0,
  max:0.8,
  palette: ['8bc4f9', 'c9995c', 'c7d270','8add60','097210']};
Map.addLayer(predImage, ndviVis, 'NDVI L8', false);

var rviCVis = {bands:['RVI_C_S1'], min:0, max:2, palette: ['black', 'white']};
Map.addLayer(predImage, rviCVis, 'RVI C-band (S1)', false);

var rviLVis = {bands:['RVI_L_P'], min:0, max:2, palette: ['black', 'white']};
Map.addLayer(predImage, rviLVis, 'RVI L-band (PALSAR)', false);

```
<img align="center" src="../images/class-gee/NDVI.png" hspace="15" vspace="10" width="400">

Now, let's examine what we have printed to the **Console** tab. We printed out the resolution and the first few images in each `imageCollection`.

<img align="center" src="../images/class-gee/print_images.png" hspace="15" vspace="10" width="200">

We can expand the different levels of properties to get a better sense of what the data look like.

<img align="center" src="../images/class-gee/print_images2.png" hspace="15" vspace="10" width="200">

We can also check the band values of every image at specific points by opening on the **Inspector** tab and clicking somewhere on the map.

<img align="center" src="../images/class-gee/inspector_tab2.png" hspace="15" vspace="10" width="200">

We also printed the merged predictor image to make sure all the other bands were added and the indices were calculated correctly.

<img align="center" src="../images/class-gee/predictor_image.png" hspace="15" vspace="10" width="200">


## Prepare Reference Points (SKIP THIS SECTION, NOW THAT WE USE CEO DATA)

Next, we generate reference data. We can either import the points we generated in SEPAL or AREA2, or we can generate points directly in this script. 

We will create a stratified random sample based on the 2014 LULC map, allocating 400 points to each class (excluding clouds and no data). The general rules of thumb for the number of reference data points are: 

* **training points:** Generate enough training points per class to have at least 10 * p, where p the number of predictor variables (e.g. if your predictor image has 16 bands, generate at least 160 training points per LULC class)

* **testing points:** Generate enough testing points per class so the ratio between training and testing points is √(p):1, where p is the number of predictor variables (e.g. if your predictor image has 16 bands, generate at least 40 tetsing points per class - the ratio between training and testing points should be 4:1, which is an 80%-20% split)

In more recent years when SAR and Planet NICFI are available, we have 22 possible predictor bands to use in our model. Based on these rules of thumb, we should have at least 220 training points per class, and at least 48 testing points per class. Thus, 400 total reference points per class go well beyond these minimums if we split them into training and testing data later 

*Tip:* Generally, more training and testing data is better, so try to generate as much as you can within your budget and time constraints.

```javascript
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////
// Prepare Training and Testing Points
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////

// create a stratified random sample based on the LULC map
  var refPoints = lulc10m.stratifiedSample({
    // number of points per class, set as a variable at the beginning of the script
    numPoints: classPointsNum,
    // the band with the LULC classes in it
    classBand: 'class',
    // resolution of the LULC map
    scale: 10,
    // set the seed so you regenerate the same exact points every time
    seed: 111,
    // (uncomment if you want a custom number of points per class)
    // classValues:classValuesList,
    // classPoints:classPointsList, 
    dropNulls:true, 
    tileScale:2, 
    geometries: true
  });
  
// Add to the map
Map.addLayer(refPoints, {}, 'reference points', false);
// print
print('reference points:', refPoints.limit(5))

```

<img align="center" src="../images/class-gee/reference_pts.png" hspace="15" vspace="10" width="400">

## Export

The last thing we do is export the **Reference Points** and **Predictor Image** (*for this demo it is being used as our **Training Image** too*) to run the classification in a separate script. These Reference Points have yet to be split up into training and testing data, and they are simple class labels without the extracted variable information from the Training Image yet.

 While we could do all the preprocessing and analysis steps in a single script, we would get an error that our user memory limit was exceeded, meaning that the script was too computationally expensive to do all in one go. This is because we are trying to run a computationally expensive preprocessing and machine learning functions on a large image with high saptial resolution and many prediction bands. 

When exporting, make sure to change the `assetId` to a path that is in your own asset library.

```javascript
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////
// Export
// //////////////////////////////////////////////////////////////////////////////////////////
// //////////////////////////////////////////////////////////////////////////////////////////

// Export the reference points -- SKIP
//Export.table.toAsset({
//  // the feature collection you want to export
//  collection: refPoints,
//  // the task name
//  description: 'refPoints',
//  // the asset ID and path (change to your own asset library)
//  assetId: 'projects/pc556-ncs-liberia-forest-mang/assets/refPoints_10m_'+d1.slice(0,4)+'_'+classPointsNum+'PerClass_v'+version
//});

predImage = predImage.set('system:time_start', ee.Date(d1).millis())
predImage = predImage
  .set('year', d1.slice(0,4)) //preserve year in metadata
  
// Export the image
Export.image.toAsset({
  // the image you want to export
  image: predImage,
  // the name of the task
  description: 'predImage',
  // the path and name of the asset (change this to be in your own asset library)
  assetId: 'projects/pc556-ncs-liberia-forest-mang/assets/predImage_'+resolution+'m_'+d1.slice(0,4)+'_v'+version,
  // the geometry to clip the image to
  region: aoi,
  // the resolution of the image
  scale: 30,
  maxPixels: 1e13,
})

```

Now, when we run our script, a new export task will show up in the **Tasks** tab. Click **RUN** to begin the export, and track its status just below in **SUBMITTED TASKS**.

<img align="center" src="../images/class-gee/export.png" hspace="15" vspace="10" width="200">

Code checkpoint: check your work in `users/ee-scripts/Liberia_Forest_SIG_workshops/09_classification_GEE/preprocessing`

If your export is taking a long time, you can use the following pre-prepared results files from the workshop asset repository for the later steps.

<a href="https://code.earthengine.google.com/?asset=projects/pc556-ncs-liberia-forest-mang/assets/refPoints_10m_2014_400PerClass" target="_blank" rel="noopener noreferrer">https://code.earthengine.google.com/?asset=projects/pc556-ncs-liberia-forest-mang/assets/refPoints_10m_2014_400PerClass</a>

<a href="https://code.earthengine.google.com/?asset=projects/pc556-ncs-liberia-forest-mang/assets/predImage_30m_2014" target="_blank" rel="noopener noreferrer">https://code.earthengine.google.com/?asset=projects/pc556-ncs-liberia-forest-mang/assets/predImage_30m_2014</a>
