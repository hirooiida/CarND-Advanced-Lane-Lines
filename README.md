# **Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

* You're reading it!

### Camera Calibration

#### Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the section 1 of the IPython notebook located in `Submission.ipynb`.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function. The function code called `undistort()` can be found in the section 2 in the `Submission.ipynb`. The comparison example between original (left) and undistorted (right) can be seen below: 

<img src=./output_images/undist_chessboard/comparison.png>

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The corresponding code can be found in the section 2 in the `Submission.ipynb`. The function name is `undistort()` and as described above, I use `cv2.calibrateCamera()` and `cv2.undistort()` there. The distortion-corrected images look like this:

<img src=./output_images/undist_images/undist_images.png>


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The corresponding code can be found in the section 3 in the `Submission.ipynb`. The function name is `create_binary_image()`. I used a combination of color and gradient thresholds to generate a binary image.  Here's an example of my output for this step.

<img src=./output_images/binary_images/binary_images.png>

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The corresponding code can be found in the section 4 in the `Submission.ipynb`. The function name is `warp()`.  The `warp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
s1 = [xsize // 2 - 66, 450]
s2 = [xsize // 2 + 66, 450]
s3 = [xsize, ysize]
s4 = [0, ysize]
src = np.float32([s1, s2, s3, s4])

d1 = [100, 0]
d2 = [xsize - 100, 0]
d3 = [xsize - 100, ysize]
d4 = [100, ysize]
dst = np.float32([d1, d2, d3, d4])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 574, 450      | 100, 0        | 
| 706, 450      | 1180, 0      |
| 1280, 720     | 1180, 720      |
| 0, 720        | 100, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

<img src=./output_images/warped_images/warped_image.png>

Then, I applied this to binary images created in the previous section. Here's the result.

<img src=./output_images/binary_warped_images/binary_warped_images.png>


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The corresponding code can be found in the section 5 in the `Submission.ipynb`. The function name is `find_lane_pixels()`.  Fist thing I did is to identify the region of interest by the sliding windows created by histgram of the binary image.   I define the two regions that have peak as each left and right lane. Then I applied second order polynomial fit to derive the shape of the lane and curvature parameters. The fitting result looks like this:

<img src=./output_images/fit_images/fit_images.png>


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The corresponding code can be found in the section 6 and 8 in the `Submission.ipynb`. `calculate_curvature()` in section 6 is a function for curvature calculation which is based on the [reference methodology](https://www.intmath.com/applications-differentiation/8-radius-curvature.php). `calculate_offset()` in section 8 is a function for vehicle offset from the center which is calculated based on the difference between pixel center of each lines and image center that is supposed to be a center of the vehicle.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The corresponding code can be found in the section 5 in the `Submission.ipynb`. The function name is `draw_lane()`.  Here is an example of my result on a test image:

<img src=./output_images/filled_images/filled_images_with_parameters.png>

---

### Pipeline (video)

#### Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_videos/processed_project_video.mp4)

---

### Discussion

#### Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

1. The lane curvature is jittery. Sanity check and smoother functionality will be needed in the process pipeline if the post-process need smoother output.
1. I see a lot of miss-detection when I apply my pipeline for `challenge_video.mp4` ([result](./output_videos/processed_project_video.mp4)). This is because of the high contrast by road patch and shadow. It could be improved if I can ignore the area with the gradient happining between light and deep gray.
1. System monitor, something more than sanity check, will be needed to monitor the lane detection performance. If the performance is low, it will need to stop the automated driving. This system monitor will need to monitor the values before the smoother. 
1. Curvature calculation is dependent on the `warp()` performance. If it needs to be improved, the warp `src` and `dst` need to be defined carefully by the control room measeurement for example.
