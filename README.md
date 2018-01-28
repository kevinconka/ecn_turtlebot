# ecn_turtlebot
[ROS package] M1 group proyect

**In collaboration with Mahmoud Ali**

The aim of this project is to perform localization using an Extended Kalman Filter (EKF) and beacons as visual markers.
Detailed information can be found in the [PDF report](Presentation_ALI_Serrano.pdf).

## Hardware
* Turtlebot 2
* Omni 60 (omnidirectional camera capruting 360° panoramic images at 60Hz)

![turtlebot](/Images/turtlebot.jpg)

## Software
* ROS kinect

## Nodes description

### Image processing nodes (using OpenCV)

`image_rectifier` rectifies images published by camera (Omni 60)
* Subscribes to
  * `camera_info`: intrensic parameters and distortion coefficients required for image rectification
  * `image_raw`: image itself
* Publishes to
  * `image_rect`: rectified image
  
![raw_image](/Images/non_rectified.jpg "Raw color image") ![rectified](/Images/rectified.jpg "Rectified color image")
___
  
`image_tiles` combines 5 different images into single panoramic image (no stitching)
* Subscribes to
  * `camera0/image_rect`: rectified image from camera0
  * `camera1/image_rect`: rectified image from camera1
  * `camera2/image_rect`: rectified image from camera2
  * `camera3/image_rect`: rectified image from camera3
  * `camera4/image_rect`: rectified image from camera4
* Publishes to
  * `image_tiles`: resulting panoramic image
  
![tiles](/Images/tiles.jpg)
___
  
`blob detector_v1` outputs distance between a beacon (blob) and the omni camera. HSV thesholding and morhological operations are used as pre-processing methods.
* Subscribes to
  * `image`: often remapped to `image_tiles` in our case
* Publishes to
  * `beacon_distance`: distance in cm
* Parameters
  * `invert`: boolean variable used to determine if image should be inverted or not (useful for solving discontinuity in HSV transformation)
  * `HMin`: minimum threshold value for Hue channel
  * `HMax`: maximum threshold value for Hue channel
  * `SMin`: minimum threshold value for Saturation channel
  * `SMax`: maximum threshold value for Saturation channel
  * `SMin`: minimum threshold value for Value channel
  * `SMax`: maximum threshold value for Value channel
  
![green_blob](/Images/blob_1.jpg "Green beacon detected") ![green_mask](/Images/blob_2.jpg "Green mask")
___

`blob dist interface` manages the distances associated with the beacons in the scenario
* Subscribes to
  * `blue\beacon_distance`: often remapped to `image_tiles` in our case
  * `red\beacon_distance`: often remapped to `image_tiles` in our case
  * `green\beacon_distance`: often remapped to `image_tiles` in our case
* Publishes to
  * `beacon_distances`: distances in cm [blue, red, green]

### Localization nodes

`odom interface` extract relevand data from the `nav_msgd/Odometry` coming from the turtlebot
* Subscribes to
  * `odom`: contains information about the 3D pose of the turtlebot
* Publishes to
  * `odom_2D`: publishes 2D pose data (x, y, theta) and their covariance
  
___
  
`ekf_node` implements the extended kalman filter (**not fully tested**)
* Subscribes to
  * `odom2D`: contains 2D pose data (x, y, theta) and corresponding covariance
  * `beacon_distances`: contains distances in cm [blue, red, green]
* Publishes to
  * `robot_pose_ekf`: 2D pose estimation
  
  
 


