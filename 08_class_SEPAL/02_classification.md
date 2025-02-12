---
layout: page
title:  "Random Forest Classification"
parent: "8. Classification with SEPAL"
nav_order: 2
---

# Classification with Random Forest in SEPAL

> *A video-tutorial is available in [this YouTube video](https://www.youtube.com/watch?v=HBlYrwmq5ak)*.


# Classification

With **Classification** recipe, we can build supervised classifications of any mosaic image. It is built on top of the most advanced tools available on Google Earth Engine (GEE) – including the RandomForest classifier – allowing us access to a user-friendly interface to:

- select an image to classify
- define the class table
- add training data from external sources and on-the-fly selection.

In combination with other tools of SEPAL, the **Classification** recipe can help develop accurate land cover maps without writing a single line of code.

## Start

Once the **Classification** recipe is selected, SEPAL will show the recipe process in a new tab (**1**); the **Image selection** window will appear in the lower right (**2**).

![Landing page](../images/sepal/sepal_rf/sepal_rf_1.png)

The first step is to change the name of the recipe. This name will be used to identify your files and recipes in SEPAL folders. Use the best-suited naming convention - double-click the tab and enter a new name. It will default to:

```code
Classification_<timestamp>
```

![Default title](../images/sepal/sepal_rf/sepal_rf_2.webp)
![Modified title](../images/sepal/sepal_rf/sepal_rf_3.webp)

<!-- 
<img align="center" src="../images/intro-gee/sepal_rf_2.webp" vspace="10" width="300">

<img align="center" src="../images/intro-gee/sepal_rf_3.webp" vspace="10" width="300">
 -->

> **Note:**
>
>  It is recommended to use the following naming convention:
>
> ```code
> <image_name>_<classification>_<measures>
> ```

## Parameters

In the lower-right corner, the following tabs are available, allowing us to customize the classification:

- `IMG`: Image to classify.
- `LEG`: Legend of the classification system.
- `TRN`: Training data of the model.
- `AUX`: Auxiliary global dataset to use in the model.
- `CLS`: Classifier configuration.

![Classification parameters](../images/sepal/sepal_rf/sepal_rf_4.png)

### Image selection

The first step consists of selecting the image bands on which to apply the classifier. The number of selected bands is not limited.

> **Note:**
>
> Increasing the number of bands to analyze will improve the model but slow down the rendering of the final image.

> **Note:**
>
> If multiple images are selected, all selected images should overlap. If the classifier finds masked pixels in one of the bands, it will mask them in the resulting classification.

Select `Add`. The following screen should be displayed:

![Image source](../images/sepal/sepal_rf/sepal_rf_5.png)

<font size = 5> Image type </font>

**Image type**


We can select images from an **Existing recipe** or an exported **GEE asset**. Both should be an `ee.Image`, rather than a `Time series` or `ee.ImageCollection`.

##### Existing recipe:

- Advantages:
  - All computed bands from SEPAL can be used.
  - Any modification to the existing recipe will propagate in the final classification.
- Disadvantages:
  - The initial recipe will be computed at each rendering step, slowing down the classification process and potentially breaking on-the-fly rendering due to GEE timeout errors.

##### GEE asset:

- Advantages:
  - Can be shared with other users.
  - The computation will be faster, as the image has already been exported.
- Disadvantages:
  - Only the exported bands will be available.
  - The `Image` needs to be re-exported to propagate changes.

Both methods behave the same way in the interface.

**Select bands**

> *Tip:* for this example, we use a public asset created with the **Optical mosaic** tool from SEPAL. It's a Sentinel-2 mosaic of Eastern Province in Zambia during the dry season from 2012 to 2020. Use the following asset name if you want to reproduce our workflow:
>
> ```code
> projects/sepal-cookbook/assets/classification/zmb-eastern_2012_2021
> ```

**Image bands**

Once an asset is selected, SEPAL will load its bands in the interface. Simply click on the band name to select them. Selected bands are displayed in gold.

In this example, we selected:

- `red`
- `nir`
- `swir`
- `green`

![Native bands](../images/sepal/sepal_rf/sepal_rf_6.webp)

**Derived bands**

The analysis is not limited to native bands. SEPAL can also build additional derived bands on-the-fly.

Select `Derived bands` at the bottom of the pop-up window and choose the deriving method. The selected method will be applied to the selected bands.

> **Note:**
>
> If more than two bands are selected, the operation will be applied to the Cartesian product of the bands.

![Derived bands](../images/sepal/sepal_rf/sepal_rf_7.webp)

Once image selection is complete, select `Apply`. The images and bands will be displayed in the `IMG` panel. Selecting the `Trash` button removes the image and its bands from the analysis.

![Selected bands](../images/sepal/sepal_rf/sepal_rf_8.webp)

### Legend setup

In this step, specify the legend for the output classified image. SEPAL provides multiple ways to create and customize a legend.

![Legend setup](../images/sepal/sepal_rf/sepal_rf_9.webp)

> **Important:**
>
> Legends created here are fully compatible with other SEPAL functionalities.

##### Manual legend

Select `Add` to add a new class to your legend. A class consists of:

- **Color**: Click the color square to open the selector.
- **Value**: Select an integer value (must be unique).
- **Class name**: Enter a description (cannot be empty).

![Manual legend](../images/sepal/sepal_rf/sepal_rf_10.webp)

##### Import legend

You can import a `.csv` file containing legend definitions.

Example `.csv` format:

```csv
code,class,color
10,Tree cover,#006400
20,Shrubland,#ffbb22
30,Grassland,#ffff4c
40,Cropland,#f096ff
```

Select `Import from CSV` and upload your file.

![Import legend](../images/sepal/sepal_rf/sepal_rf_11.webp)

Once the legend is validated, export it using `Export as CSV`.

![Manual legend](../images/sepal/sepal_rf/sepal_rf_12.webp)


### Training data

> **Note:**
>
> This step is not mandatory.

Training data can be added from external sources or collected interactively.

Select `TRN` to open the **Training Data** menu.

![Training menu](../images/sepal/sepal_rf/sepal_rf_13.webp)

Training data can be imported from:

- CSV files
- Google Earth Engine tables
- Sampled classifications
- Existing SEPAL recipes

### Classifier configuration

Select `CLS` to configure the classifier. SEPAL supports:

- Random Forest
- Gradient Tree Boost
- CART
- Naive Bayes
- SVM
- Min Distance
- Decision Tree

![Classifier configuration](../images/sepal/sepal_rf/sepal_rf_25.webp)

### Export

> **Important:**
>
> Exporting requires a small computation quota.

Select `Retrieve` to open the **Export** pane and choose the export parameters.

![Export](../images/sepal/sepal_rf/sepal_rf_29.webp)

The exported image will be stored in the `Downloads` folder.

