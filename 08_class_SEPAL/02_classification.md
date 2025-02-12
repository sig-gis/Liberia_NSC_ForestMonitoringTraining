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

<font color = red> CHANGE RECIPE NAME </font>


```code
Classification_<timestamp>
```

![Default title](../images/sepal/sepal_rf/sepal_rf_2.webp)
![Modified title](../images/sepal/sepal_rf/sepal_rf_3.webp)

<!-- 
<img align="center" src="../images/intro-gee/sepal_rf_2.webp" vspace="10" width="300">

<img align="center" src="../images/intro-gee/sepal_rf_3.webp" vspace="10" width="300">
 -->

> **_Note:_**
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

> **_Note:_**
>
> Increasing the number of bands to analyze will improve the model but slow down the rendering of the final image.

> **_Note:_**
>
> If multiple images are selected, all selected images should overlap. If the classifier finds masked pixels in one of the bands, it will mask them in the resulting classification.

Select `+ Add`. The following screen should be displayed:

![Image source](../images/sepal/sepal_rf/sepal_rf_5.png)


**<font size = 3> Image type </font>**


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

**<font size = 3> Select bands </font>**

For this example, we use a public asset created with the **Optical mosaic** tool from SEPAL. It's a Landsat mosaic of Liberia for 2014 <font color = red> DO WE NEED TO SET IT TO "ANYONE CAN READ"? </font>:

```code
projects/pc556-ncs-liberia-forest-mang/assets/liberia_2014_lc
```

**<font size = 3> Image bands </font>**

Once an asset is selected, SEPAL will load its bands in the interface. Click on the band name to select it. The selected bands are summarized in the expansion panel title (1) and displayed in gold in the panel content (2).

In this example, we selected:

- `blue`
- `green`
- `red`
- `nir`
- `swir1`
- `swir2`

![Native bands](../images/sepal/sepal_rf/sepal_rf_6.png)

**<font size = 3> Derived bands </font>**

SEPAL can build additional derived bands on-the-fly, so the analysis is not limited to native bands. 

Select the green `+ Derived bands` and choose the deriving method. The selected method will be applied to the selected bands and its name will be added in the expansion panel.

![Derived bands](../images/sepal/sepal_rf/sepal_rf_7.png)


> **_Note:_**
>
> Feel free to explore the different options. The pre-computed indices are available in the `Indexes` derived bands.
>
>
> If more than two bands are selected, the operation will be applied to the Cartesian product of the bands, meaning that selecting bands A, B and C, and applying the `Difference` derived bands, will add three bands to your analysis:
> 
> - A - B
> - A - C
> - B - C


Select the following indices:

- `ndvi`
- `ndmi`
- `mndwi`
- `evi`
- `mvi`

![Derived bands](../images/sepal/sepal_rf/sepal_rf_7_5.png)



Once image selection is complete, select `✓ Apply`. The images and bands will be displayed in the `IMG` panel. Selecting the `Trash bin` button removes the image and its bands from the analysis. If you want to add more images or different datasets (e.g.Landsat and Sentinel), you can click `+ Add` to select bands specific to the other image. Click `> Next` to continue to next step.

![Selected bands](../images/sepal/sepal_rf/sepal_rf_8.png)



<font color = red> EDIT HERE ONWARDS </font>

### Legend setup

In this step, specify the legend for the output classified image. SEPAL provides multiple ways to create and customize a legend - manually or by importing a `csv` table.

![Legend setup](../images/sepal/sepal_rf/sepal_rf_9.png)

> **Important:**
>
> Legends created here are fully compatible with other SEPAL functionalities.

**<font size = 3> Manual legend </font>**

Select `+ Add` to add a new class to your legend. A class consists of:

- *Color (1)*: click the color square to open the selector (must be unique)
- *Value (2)*: select an integer value (must be unique)
- *Class name (3)*: enter a description (cannot be empty)

You can select `HEX` (4) to display the hexadecimal value of the selected color. It can also be used to insert a known color palette by utilizing its values.

If unsure which colors to use for each class, apply colors using a preselected GEE color map (5). It will color all classes in your panel.

![Manual legend](../images/sepal/sepal_rf/sepal_rf_10.png)

**<font size = 3> Import legend </font>**

You can also import a `.csv` file containing pre-made legend definitions.

Example `.csv` format:

```csv
code,class,color
10,Tree cover,#006400
20,Shrubland,#ffbb22
30,Grassland,#ffff4c
40,Cropland,#f096ff
```

Example 2:

```csv
code,class,red,blue,green
10,Tree cover,0,100,0
20,Shrubland,255,187,34
30,Grassland,255,255,76
40,Cropland,240,150,255
```

Click `^` and select `Import from CSV` to upload your file. You can select the columns that are defining your `csv` file (`Single column` for hexadecimal-defined colors or `Multiple columns` for RGB-defined colors).

![Import legend](../images/sepal/sepal_rf/sepal_rf_11.png)

Click `✓ Apply` to validate your selection. At this stage, you would still be able to modify the legend if needed. When done, `✓ Done` to validate this step. 

<font color = red> UPDATE THE LEGEND </font>

![Manual legend](../images/sepal/sepal_rf/sepal_rf_12.png)

#### Export legend

Once the legend is validated, select `^` and export it using `Export as CSV`. A file will be downloaded to you computer named `<recipe_name>_legend.csv`.


### Training data

> **_Note:_**
>
> This step is not mandatory.

Training data can be added from external sources or collected interactively. Select `TRN` to open the **Training Data** menu.

<font color = red> EDIT THE IMAGE W CLASSES AND COLORS </font>

![Training menu](../images/sepal/sepal_rf/sepal_rf_13.png)

Training data can be imported from:

- CSV files
- Google Earth Engine tables
- Sampled classifications
- Existing SEPAL recipes

We will use the first option, but see more details on each option in the [SEPAL Documentation](https://docs.sepal.io/en/latest/cookbook/classification.html).

<font color = red> INCLUDE THE DESCRIPTION OF ONE OF THE OPTIONS HERE </font>


### Auxiliary datasets

You can also select `AUX` to include more information that could be useful to the classification model but is not always included in satellite image bands, such as **elevation** data. Three sources are currently available on SEPAL:
 - **Latitude:** On-the-fly latitude dataset built from the coordinates of each pixel’s center.

 - **Terrain:** From the [NASA SRTM Digital Elevation 30 m dataset](https://developers.google.com/earth-engine/datasets/catalog/USGS_SRTMGL1_003), SEPAL will use the `Elevation`, `Slope` and `Aspect` bands. It will also add an `Eastness` and `Northness` band derived from `Aspect`.

 - **Water:** From the [JRC Global Surface Water Mapping Layers, v1.3 dataset](https://developers.google.com/earth-engine/datasets/catalog/JRC_GSW1_3_GlobalSurfaceWater), SEPAL will add the following bands: `occurrence`, `change_abs`, `change_norm`, `seasonality`, `max_extent`, `water_occurrence`, `water_change_abs`, `water_change_norm`, `water_seasonality` and `water_max_extent`.

 ![AUX](../images/sepal/sepal_rf/sepal_rf_22.png)


### Classifier configuration

Select `CLS` to configure the classifier. SEPAL supports:

- Random Forest
- Gradient Tree Boost
- CART
- Naive Bayes
- SVM
- Min Distance
- Decision Tree

 ![Classifiers](../images/sepal/sepal_rf/sepal_rf_23.png)

> **_Note:_**
>
>Customizing the classifier is a section designed for advanced users. Make sure that you thoroughly understand how the classifier you’re using works before changing its parameters.
>
> The default values if a Random Forest Classifier using 25 trees.

You may click `More` on the lower-left side of the panel to customize your classifier.

 ![Classifiers](../images/sepal/sepal_rf/sepal_rf_24.png)

<font color = red> INCLUDE MORE DETAIL ON CLASSIFICATION HERE </font>


![Classifier configuration](../images/sepal/sepal_rf/sepal_rf_25.webp)

![Classifier configuration](../images/sepal/sepal_rf/sepal_rf_26.webp)

![Classifier configuration](../images/sepal/sepal_rf/sepal_rf_27.png)

![Classifier configuration](../images/sepal/sepal_rf/sepal_rf_28.webp)


### Export

> **Important:**
>
> Exporting requires a small computation quota, which can be added through the `Terminal` (see  [**Terminal** section here](https://docs.sepal.io/en/latest/setup/presentation.html)).

Selecting the *cloud download* icon (**1**) to access the `Retrieve` pane to open the **Export** pane and choose the export parameters:
 - Select the band to export (**2**). There is no maximum number of bands; however, exporting useless bands will only increase the size and time of the output.
 
 - You can set a custom scale for exportation (**3**) by changing the value of the slider in metres (m).
 
 > **_Note:_** 
 >
 > Requesting a smaller resolution than images’ native resolution (10 m for Sentinel, 30 m for Landsat) will not improve the quality of the output.

You can export the image to:
 - SEPAL workspace: will  be in `.tif` format in the Downloads folder
  - Google Earth Engine Asset: will be exported to your GEE account asset list.

Select `Apply` to start the download process.

![Export](../images/sepal/sepal_rf/sepal_rf_29.png)

You can follow the progress of exportation in `Tasks` tab in the lower left of your screen.

![Export](../images/sepal/sepal_rf/sepal_rf_30.png)
![Export](../images/sepal/sepal_rf/sepal_rf_31.png)


