---
layout: page
title: Resources
parent: "6. Drone Flight & Image Processing"
nav_order: 11
---


# Extra Resources

## Free Drone Imagery

Here are some places you can find free, open-source drone imagery to play around with (both raw and processed):

* <a href="https://dronemapper.com/sample_data/" target="_blank" rel="noopener noreferrer">DroneMapper</a>
* <a href="https://wingtra.com/mapping-drone-wingtraone/aerial-map-types/data-sets-and-maps/" target="_blank" rel="noopener noreferrer">Wingtra</a>
* <a href="https://support.pix4d.com/hc/en-us/articles/360000235126-Example-projects-real-photogrammetry-data#quarry" target="_blank" rel="noopener noreferrer">Pix4D</a>
* <a href="https://openaerialmap.org/" target="_blank" rel="noopener noreferrer">OpenAerialMap</a>
* <a href="https://www.opendronemap.org/odm/datasets/" target="_blank" rel="noopener noreferrer">OpenDroneMap</a>
* <a href="https://earthexplorer.usgs.gov/" target="_blank" rel="noopener noreferrer">USGS EarthExplorer</a>


## Improving Drone Images in Forests

Here are some tips from Pix4D on how to improve drone image products in areas with dense vegetation (e.g. forests):

<a href="https://support.pix4d.com/hc/en-us/articles/202560159-How-to-improve-the-outputs-of-dense-vegetation-areas-using-PIX4Dmapper" target="_blank" rel="noopener noreferrer">How to improve the outputs of dense vegetation areas using PIX4Dmapper?</a>

<a href="https://support.pix4d.com/hc/pix4dmapper" target="_blank" rel="noopener noreferrer">Pix4Dmapper documentation and Knowledgebase</a>

# Vocabulary

## Photogrammetry:

the use of photography in surveying and mapping to measure distances between objects

## Global (Mechanical) Shutter:

A type of drone camera that exposes all the pixels in an image at once (the sensor opens, takes the picture, and closes).  This type of shutter allows you to fly faster and still get high quality images.  Thus, it is best for photogrammetry.  DJI Phantoms have global shutters.  You can fly faster with a global shutter.

<img align="center" src="../images/drone/globalshutter.png" hspace="15" vspace="10" width="200">

<a href="https://www.pix4d.com/blog/rolling-shutter-correction/" target="_blank" rel="noopener noreferrer">https://www.pix4d.com/blog/rolling-shutter-correction/</a>


## Rolling (Electronic) Shutter:

A type of drone camera that exposes pixels in an image one row at a time.  This type of shutter forces you to fly slower and is more prone to image distortion (because the camera is at different locations when it is capturing the first row and the last row).  Thus, it is best for capturing still images.  DJI Mavics have rolling shutters.  You need to fly slower with a rolling shutter, and the drone should ideally stop fully when taking each picture.

<img align="center" src="../images/drone/rollingshutter.png" hspace="15" vspace="10" width="250">

<a href="https://www.pix4d.com/blog/rolling-shutter-correction/" target="_blank" rel="noopener noreferrer">https://www.pix4d.com/blog/rolling-shutter-correction/</a>


## Keypoints:

Keypoints are points in the images that stand out in the image. They are important because no matter how the image changes, you should be able to find the same points in each image.  They are used to correctly align overlapping images with each other.  Pix4d extracts many keypoints from your images, scores them based on certain criteria, and chooses the best ones to use.

A common technique for extracting keypoints is the Scale Invariant Feature Transformation (SIFT) algorithm (Lowe, 1999). SIFT identifies features in an image scale space that are invariant to image scaling and rotation, and are partially invariant to illumination changes and 3D projection.

*Read more about SIFT in this article:
Lowe, D.G., 1999. Object recognition from local scale-invariant features, Proceedings of the International Conference on Computer Vision, Corfu, September 1999.*

<img align="center" src="../images/drone/keypoints.png" hspace="15" vspace="10" width="400">

Tyagi, D. (2020, April 7). Introduction to SIFT( Scale Invariant Feature Transform). Data Breach. <a href="https://medium.com/data-breach/introduction-to-sift-scale-invariant-feature-transform-65d7f3a72d40" target="_blank" rel="noopener noreferrer">https://medium.com/data-breach/introduction-to-sift-scale-invariant-feature-transform-65d7f3a72d40</a>


## Tie Points:

Keypoint matching (creation of tie points) is the process of finding the corresponding keypoint of one image in another image. The more keypoint matches that are computed, the higher the degree of confidence that a point is placed correctly; fewer keypoint matches will yield less confidence.

Keypoint matches are used to create tie points (in this case Automatic Tie Points, or ATPs). ATPs are 3D points and their corresponding 2D keypoints (which were automatically detected in the images and used to compute 3D position of points). This is what allows the software to go from 2D to 3D, and it is also used in the calibration of the images. ATPs cannot provide the absolute position of the images (known ground coordinates), but they do provide the relative positions of your images to each other.

<img align="center" src="../images/drone/tiepoints.png" hspace="15" vspace="10" width="600">

Goodbody, T. R. H., Coops, N. C., & White, J. C. (2019). Digital Aerial Photogrammetry for Updating Area-Based Forest Inventories: A Review of Opportunities, Challenges, and Future Directions. Current Forestry Reports, 5(2), 55–75. <a href="https://doi.org/10.1007/s40725-019-00087-2" target="_blank" rel="noopener noreferrer">https://doi.org/10.1007/s40725-019-00087-2</a>


<img align="center" src="../images/drone/tiepoints_3.png" hspace="15" vspace="10" width="500">

<a href="https://catalyst.earth/catalyst-system-files/help/COMMON/concepts/TiePoint_explain.html" target="_blank" rel="noopener noreferrer">https://catalyst.earth/catalyst-system-files/help/COMMON/concepts/TiePoint_explain.html</a>


## Bundle Block Adjustment:

Bundle block adjustment (BAA) is a step in the analysis process that refines the 3D positions of features captured in your imagery. BAA uses the internal relationship between overlap images, keypoints or ground control points, and the internal and external camera parameters, and applies the adjustment to images within a specific block. Simply put, it uses ATPs to create a mathematical model, which it then uses to generate the sparse point cloud.

<img align="center" src="../images/drone/tiepoints_2.png" hspace="15" vspace="10" width="500">

Aber, J. S., Marzolff, I., & Ries, J. B. (2010). Chapter 3—Photogrammetry. In J. S. Aber, I. Marzolff, & J. B. Ries (Eds.), Small-Format Aerial Photography (pp. 23–39). Elsevier. <a href="https://doi.org/10.1016/B978-0-444-53260-2.10003-1" target="_blank" rel="noopener noreferrer">https://doi.org/10.1016/B978-0-444-53260-2.10003-1</a>



