# Advanced_Lane_Detection

### Goals of this project 

[//]: # (Image References)

[image1]: ./output_images/calibration.png "Calibration Steps"
[image2]: ./output_images/undistorted.png "Road Transformed"
[image3]: ./output_images/binary_grad_color.png "Binary Example"
[image4]: ./output_images/persp_transf_boxes.png "Warp Example"

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


### Camera Calibration 
1. convert to gray scale using `cv2.cvtColor()`
2. look for chessboard corners using `cv2.findChessboardCorners()`
3. if the previous step successfully find the corners in the image, then add them to the `imgpoints` data structure
Once all images have been precessed, the camera calibration and distortion coefficients are computed using `cv2.calibrateCamera()`

### Perspective Transform 
#### Perspective Transformation
This section contains the function `perspective_transform()` that computes the perspective transformation to obtain a birds-eye view of the road.
To perform this transformation are required two set of points:
* four *source* points on the original image that defines a polygon that contains the two lanes;
* four *destination* points on the final image that defines the final polygon in which the original one will be mapped.

## Pipeline of Code 

```python
# remove distortion from image
undist_img = undistorted_image(img, mtx, dist)

# compute combined gradient binary
combined_grad_img = combine_gradients(undist_img, ksize, mod_thresh, dir_thresh)

# compute color binary
color_img = color_transformation(undist_img, s_thresh)

# combine the two contributions
binary_img, stack_img = combine_gradinet_with_color(combined_grad_img, color_img)

# perspective transform
perspec_img, M, Minv, src_raw, dst_raw = perspective_transform(binary_img)

# find lane points
histogram, lane_point_img, left_fitx, right_fitx, ploty = step_find_lane_lines(perspec_img)

# compute curvature
curvature = measure_curvature_meters(left_fitx, right_fitx, ploty)

# compute ego position
ego_pos = determine_ego_position(img, left_fitx, right_fitx)

# compose the final image
fin_img = draw_result(undist_img, perspec_img, lane_point_img, Minv, left_fitx, right_fitx, ploty, curvature, ego_pos, True)
```
