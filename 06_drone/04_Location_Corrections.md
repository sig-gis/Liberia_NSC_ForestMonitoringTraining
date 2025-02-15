---
layout: page
title: "Location Corrections"
parent: "6. Drone Flight & Image Processing"
nav_order: 3
---

# Location Corrections

Most drones utilize GPS for location, however without correction, the drones internal GPS units are not very accurate.  There are a few methods to correct for this issue.

### Real Time Kinematics (RTK)

This method of corrections makes location correction in real time when the drone is flying.  This method requires a GPS base station in a fixed location near the flight location and an additional unit on the drone.  While flying the base station communicates with the flight control device and overhead satellites to properly triangluate the drones location.

### Post Processing Kinematics (PPK)

This method of correction is done after flying and corrections are made as an additional step using data downloaded from a virtual base station.  This steps can be used when a base station is not available or proper satellite connection is unattainable.

### Ground Control Points (GCP)

This method makes corrections during the photogrammetry processing step.  To use this method, a surveyor places targets in known locations that are evenly distributed across a project area, a minimum of three targets are required.  

<img align="center" src="../images/drone/GPS_Corrections.jpg" hspace="15" vspace="10" width="1000">