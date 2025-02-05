---
layout: page
title:  "Preprocessing Imagery"
parent: "9. Classification with SEPAL"
nav_order: 1
---

# Preprocessing Imagery in SEPAL

## Optical mosaics
*Combine images to create single raster datasets with optical mosaics*

### Overview

A mosaic is a combination or fusion of two or more images. In SEPAL, you can create a single raster dataset from several raster datasets by mosaicking them together.
This can be achieved on both contiguous rasters (see first image below) and overlapping images (see second image below).

![Contiguous rasters](../_images/cookbook/optical_mosaic/mosaic_contiguous.gif)
![Overlapping images](../_images/cookbook/optical_mosaic/mosaic_overlay.png)

These overlay areas can be managed in various ways. For example, you can choose to:

- Keep only the raster data from the first or last dataset.
- Combine the values of the overlay cells using a weighting algorithm.
- Average the values of the overlay cells.
- Take the maximum or minimum value.

In addition, certain corrections can be made to the image to account for clouds, snow, and other factors; these operations are complex and repetitive.

SEPAL offers an interactive and intuitive way to create mosaics in any area of interest (AOI).

> **Note:**
>
> You won't be able to retrieve the images if your SEPAL and Google Earth Engine (GEE) accounts are not connected. For more information, go to [GEE setup](../setup/gee).

## Start

Once the mosaic recipe is selected, SEPAL will display the recipe process in a new tab (**1**) and the **AOI selection** window will appear in the lower right (**2**).

![Landing page](../_images/cookbook/optical_mosaic/landing.png)

The first step is to change the name of the recipe. This name will be used to identify your files and recipes in SEPAL folders. Use the best-suited convention for your needs. Simply double-click the tab and write a new name. It will default to:

```code
Optical_mosaic_<start_date>_<end_date>_<band name>
```

![Default title](../_images/cookbook/optical_mosaic/default_title.png)
![Modified title](../_images/cookbook/optical_mosaic/modified_title.png)

> **Note:**
>
> The SEPAL team recommends using the following naming convention:
>
> ```code
> <aoi name>_<dates>_<measure>
> ```

## Parameters

In the lower-right corner, five tabs allow you to customize the mosaic creation:

- `AOI`: Area of interest.
- `DAT`: Target date of interest for the mosaic/composite.
- `SRC`: Source datasets of the mosaic/composite.
- `SCN`: Scene selection parameters.
- `CMP`: Composition parameters.

![Mosaic parameters](../_images/cookbook/optical_mosaic/no_parameters.png)

### AOI selection

The data exported by the recipe will be generated from within the bounds of the AOI. There are multiple ways to select the AOI in SEPAL:

- Administrative boundaries.
- EE Tables.
- Drawn polygons.

For more details, see [AOI selection](../feature/aoi_selector).

![Select AOI](../_images/cookbook/optical_mosaic/aoi.png)

### Date

#### Yearly mosaic

In the `DAT` tab, select a year for the pixels in the mosaic. Then click `Apply`.

![Year selection](../_images/cookbook/optical_mosaic/select_year.png)

#### Seasonal mosaic

Expand the date selection tool in the `DAT` panel and select a season of interest.

- Click the **calendar icon** to open the **Date selection** pop-up.
- Use the slider to define a season around the target date.
- Adjust past/future season parameters to increase the pool of images.

![Season selection](../_images/cookbook/optical_mosaic/select_season.png)

### Sources

A mosaic uses different raster datasets obtained from multiple sources. SEPAL allows you to select data from multiple entry points:

- **L8**: [Landsat 8 Tier 1](https://developers.google.com/earth-engine/datasets/catalog/LANDSAT_LC08_C01_T1)
- **L7**: [Landsat 7 Tier 1](https://developers.google.com/earth-engine/datasets/catalog/LANDSAT_LE07_C01_T1)
- **L4-5**: [Landsat 4 & 5 Tier 1](https://developers.google.com/earth-engine/datasets/catalog/LANDSAT_LT04_C01_T1)
- **A+B**: [Sentinel-2 Multispectral](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2)

![Source selection](../_images/cookbook/optical_mosaic/select_source.png)

### Scenes

> **Note:** If Sentinel and Landsat data have been selected, all scenes must be used due to different tiling systems.

You can select scenes manually or automatically:

- `Use all scenes`: Includes all available images.
- `Select scenes`: Prioritize based on:
  - **Cloud-free**: Prioritizes images with zero or few clouds.
  - **Target date**: Prioritizes images that match the target date.
  - **Balanced**: Maximizes both cloud-free and target date criteria.

![Scene selection](../_images/cookbook/optical_mosaic/scene_method.png)

### Composite

> **Note:**
>
> Default settings:
>
> - **Correction**: `SR`, `BRDF`
> - **Pixel filters**: None
> - **Cloud detection**: `QA bands`, `Cloud score`
> - **Cloud masking**: `Moderate`
> - **Cloud buffering**: `None`
> - **Snow masking**: `On`
> - **Composing method**: `Medoid`

Define the compositing method for the final image.

![Composite options](../_images/cookbook/optical_mosaic/composite_options.png)

### Analysis

Select scenes and begin the analysis using the top-right menu:

- `Auto-select scenes` (`magic wand` icon)
- `Clear selected scenes` (`trash` icon)
- `Retrieve mosaic` (`cloud download` icon)

![Analysis menu](../_images/cookbook/optical_mosaic/analysis.png)

### Retrieve

> **Important:**
>
> Exporting requires a small computation quota. See [Resource setup](../setup/resource).

Export the image to:

- `SEPAL workspace` (downloads as `.tif` file).
- `Google Earth Engine Asset`.

> **Note:** If `Google Earth Engine Asset` is not displayed, ensure your GEE account is connected to SEPAL.

![Retrieve pane](../_images/cookbook/optical_mosaic/retrieve.png)

### Exportation status

Monitor task progress in the **Tasks** tab (bottom-left corner).

- View task progress.
- Check for errors.
- Monitor tasks using the [GEE task manager](https://code.earthengine.google.com/tasks).

![Download process](../_images/cookbook/time_series/download.png)
![Download complete](../_images/cookbook/time_series/download_complete.png)

### Access

Once downloaded, the data is stored in the `Downloads` folder:

```bash
.
└── downloads/
    └── <MO name>/
        ├── <MO name>_<gee tile id>.tif
        └── <MO name>_<gee tile id>.vrt
```

> **Tip:**
>
> The full folder structure is required to read the `.vrt` file.

Now that you've downloaded the optical mosaic, you can use it in other SEPAL workflows or transfer it to your computer using [FileZilla](../setup.filezilla.html).
