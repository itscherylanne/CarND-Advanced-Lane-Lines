## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

[//]: # (Image References)
[image0]: ./examples/calibration_detection.png "Undistorted"
[image1]: ./examples/calibration_undistort.png "Undistorted"
[image2]: ./examples/distort_v_undistort.png "Road Transformed"
[image3]: ./examples/color_and_edge.png "Binary Example"
[image4]: ./examples/perspective1.png "Warp Example"
[image4a]: ./examples/perspective2.png "Warp Example"
[image4b]: ./examples/perspective3.png "Warp Example"
[image4c]: ./examples/perspective4.png "Warp Example"
[image4d]: ./examples/perspective5.png "Warp Example"
[image4e]: ./examples/perspective6.png "Warp Example"
[image4f]: ./examples/perspective7.png "Warp Example"
[image4g]: ./examples/perspective8.png "Warp Example"
[image5]: ./output/sliding_window_straight_lines1.jpg "Fit Visual"
[image5a]: ./output/sliding_window_straight_lines2.jpg "Fit Visual"
[image5b]: ./output/sliding_window_test1.jpg "Fit Visual"
[image5c]: ./output/sliding_window_test2.jpg "Fit Visual"
[image5d]: ./output/sliding_window_test3.jpg "Fit Visual"
[image5e]: ./output/sliding_window_test4.jpg "Fit Visual"
[image5f]: ./output/sliding_window_test5.jpg "Fit Visual"
[image5g]: ./output/sliding_window_test6.jpg "Fit Visual"
[image6]: ./output/final_straight_lines1.jpg "Output"
[image6a]: ./output/final_straight_lines2.jpg "Output"
[image6b]: ./output/final_test1.jpg "Output"
[image6c]: ./output/final_test2.jpg "Output"
[image6d]: ./output/final_test3.jpg "Output"
[image6e]: ./output/final_test4.jpg "Output"
[image6f]: ./output/final_test5.jpg "Output"
[image6g]: ./output/final_test6.jpg "Output"
[video1]: ./output/project_video.mp4 "Video"

Overview
---
This repository contains starting files for my submission of the Advanced Lane Finding Project.

My project includes the following files:
* Project4-AdvancedLaneFinding.ipynb jupyter notebook containing lane finding algorithm
* camera_calibration.ipynb for calculating the offline camera calibration
* ./camera_cal/cal_data_pickle.p offline camera calibration results
* ./output folder for storing results
* README.md summarizing the results

---
### Camera Calibration


The code for this step is contained in the two code cells of the IPython notebook located in `camera_calibration.ipynb`.

In the first cell, I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  
![alt text][image0]

In the second cell, I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

The results were saved in a pickle file and loaded later in `Project4-AdvancedLaneFinding.ipynb`

### Pipeline (single images)

#### Distortion Correction

Once we understood the camera's distortion coefficients, we would need to apply them our imagery. This is for correcting the lens distortion so that we may correctly detect and measure our lane lines. Below is an application of `undistort` on a single dash-cam image.
![alt text][image2]

#### Lane Segmentation: Color Transforms, Binarization, Region of Interest Isolation
The functions used to isolate the lane line pixels may be found in cell 9 of `Project4-AdvancedLaneLines.ipynb`. These functions are `region_of_interest()`, `laplacian_thresholding()`, and `combined_color_thresholding`.

Below is an example showing the two binary images created by `laplacian_thresholding()` and `combined_color_thresholding`. After applying `region_of_interest()` pixel masking and applying the perspective transform (discussed below), we get the final image on the right.
![alt text][image3]


#### Perspective Transform

The code for my perspective transform includes a function called `corners_unwarp()`, which appears in cell 4 of `Project4-AdvancedLaneFinding.ipynb`.  The `corners_unwarp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([
    [512, 480],
    [766, 480],
    [1165, 670],
    [100, 670]])

dst = np.float32(
            [[offset, 0],
            [img_size[0] - offset, 0],
            [img_size[0] - offset, img_size[1]],
            [offset, img_size[1]]])
```


I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]
![alt text][image4a]
![alt text][image4b]
![alt text][image4c]
![alt text][image4d]
![alt text][image4e]
![alt text][image4f]
![alt text][image4g]

#### Finding the Lane Line and Fitting a Curve
The line detection code could be found at cell  11 in `Project4-AdvancedLaneLines.ipynb` by the function `find_lines()`.

Given the perspective transformed binary image, the algorithm calculates a histogram of detected pixels along the X axis (line 9 of cell 11). This histogram then identifies two groupings of pixels - one representing the left lane and the other representing the right lane (lines 14-16).

This initializes the sliding window algorithm, where the algorithm searches a local region (starting at the bottom of the image, centered around the initial left/right lane location) and collects all the non-zero points (the detected pixels). With the collected pixels in the local window, the mean location along the x-axis (columns) is calculated for the next window. The window slides upward and repeats the process with the updated x-location. The sliding window algorithm is performed in lines 38-63 of `find_lines()`.

![alt text][image5]
![alt text][image5a]
![alt text][image5b]
![alt text][image5c]
![alt text][image5d]
![alt text][image5e]
![alt text][image5f]
![alt text][image5g]

#### Radius of Curvature Calculation
The Radius of Curvature (RoC) calculation is embedded within the `find_lines()` function (located in cell 11 in `Project4-AdvancedLaneLines.ipynb`). It may be found on lines 86-98. More information on the RoC calculation may be found in this [link](https://www.intmath.com/applications-differentiation/8-radius-curvature.php)

The RoC formula can be generalized as follows:
``` python
    curverad = ((1 + (2*fit_cr[0]*y_eval*ym_per_pix + fit_cr[1])**2)**1.5) / np.absolute(2*fit_cr[0])
```
Where:
* `fit_cr` is the polynomial fitted equation based on the detected pixels
* `y_eval` is the maximum y-value used for plotting
* `ym_per_pix` is the pixel-to-meter conversion in the y-direction (rows)

The calculated RoCs are converted from pixels to meters by applying the following conversions. These conversion values were provided by Udacity.
``` IPython
ym_per_pix = 3.0/72.0 # meters per pixel in y dimension
xm_per_pix = 3.7/400 # meters per pixel in x dimension
```

#### Vehicle Location Calculation
The vehicle position is calculated relative to the center of the detected lane lines. This calculation may be found in lines 103 - 106 within `find_lines()`. The lane center is calculated as the midpoint between the detected lane lines in the pixel-space. The vehicle center is assumed to be at the center of the image. The pixel difference between the vehicle center and the calculated lane center is then converted to meters based on the pixel-to-meters multiplier `xm_per_pix`.

#### Results Projection

I implemented this step in cells 12 in my code in `Project4-AdvancedLaneFinding.ipynb` in the function `draw_detected_lines()`. The function takes the (x,y) points calculated from the lane fitting calculation and draws the polygon from the birds-eye vantage point as the lines. This polygon is then projected using `cv2.warpPerspective()` and the inverse warping matrix (inverse to the matrix when computing the perspective transform).  

![alt text][image6]
![alt text][image6a]
![alt text][image6b]
![alt text][image6c]
![alt text][image6d]
![alt text][image6e]
![alt text][image6f]
![alt text][image6g]

---

### Pipeline (video)

Here's a [link to my video result](./output/project_video.mp4)

---

### Discussion

My detection pipeline relies on region-of-interest masking in order to isolate the lane pixels of interest. While this works on the video for the project, lanes could not be detected in the challenge videos. This suggests that improvement on the algorithm that isolates the lane lines prior to lane fitting.

Another way to isolate the lane lines better is create a more robust edge detection. The shadows on the road creates unnecessary edges that had to be filtered out by very specific color thresholding. These color thresholds work for specifically the environment in the video file. I believe the algorithm would need to be tolerant to varying environmental conditions.
