# Advanced_Lane_Detection

### Goals of this project 

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

### Final Outcome

Here's a [link to my video result](./project_video_result.mp4) 


[//]: # (Image References)

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./output/binary.PNG "Binary Example"
[image4]: ./output/warped_img.PNG "Warp Example"
[image5]: ./output/poly.PNG "Fit Visual"
[image6]: ./output/final.PNG "Output"
[video1]: ./project_video_result.mp4 "Video"


### Camera Calibration

With the chessboard images provided, I calibrated the camera in the following way to correct the radial and tangential distortions from the camera:
1. convert to gray scale using `cv2.cvtColor()`
2. look for chessboard corners using `cv2.findChessboardCorners()`
3. if the previous step successfully find the corners in the image, then add them to the `imgpoints` data structure

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 


#### Undistorted Image

![alt text][image2]


### Gradient Computations

I converted the undistorted image from RGB (Red, Green, Blue) to HLS (Hue, Lightness, Saturation) colorspace. The reason for this is that the saturation channel is less sensitive to light variation than RGB. On the other hand, the lightness channel comes handy when the goal is to detect the white color.
Once the lanes were filtered, they could now be more easily detected with gradient edge detection methods such as gradient direction and canny edge detection.

I used a combination of color and gradient thresholds to generate a binary image. Here's an example of my output for this step. 

![alt text][image3]

### Perspective Transform

The next step was to shift the perspective of the road to bird's view and detect the actual lines that compose the lanes. In order to do this, I first applied a polygon mask to focus the line detection to the road.
I got the perspective transform matrix with the cv2.getPerspectiveTransform function and applied the trasformation with cv2.warpPerspective. In order to get the source points for the transform, I took a straight lines example image, so I would know how the transformed image should look like (two vertical parallel lines).

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 595, 450      | 290, 0        | 
| 686, 450      | 989, 0        |
| 1100, 720     | 989, 719      |
| 208, 720      | 290, 719      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### Poly Fit

For images , I applied a histogram of the lower half of the image to locate the lines' base locations. After this I used windows to capture the points in each line and slided it upwards to capture the next part of the line. If a max numberof points were captured I would recenter the window. With all the captured points for each line, then I fitted a second order polynomial and with it I was now able to calculate the lines' curvature.

For Video purpose , polynomial fit was used becausethere was previous information of a polynomial fit and its points, I would use the corresponding base points as a starting point instead of applying the histogram. Also I would use the best saved coordinate points of the previous polynomial as the windows' locations. In case not enough points were found, then I would apply the histogram at that point and recenter the following windows and slide the windows as done previously.


Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Radius of Curvature 

The radius of curvature was computed by the lanes curvature starting from the lane coordinates. This function performs also the transformation from pixel reference frame to cartesian reference frame using information on the standard lane length and road width.

The offset was calclated using left and right lane fit and compute the position of the vehicle with respect to the midpoint between the two lanes.


![alt text][image6]

---


