## Advanced Lane Finding Project

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

The code for this step is contained in the accompanying 'advancedLanes.ipynb' notebook located in (under sections 'Read and print images' and 'camera caliberation').  

I started by preparing "object points", which will be the (9, 6, 0) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (9, 6) plane at z=0, such that the object points are the same for each calibration image.  
However, i found that 3 of the 20 images are not populating. By taking a range for y (5,6) and x (6,7,8,9), i was able to get all images.

Found chessboard corners and drew chessboard corners, by using cv2.findChessboardCorners() and cv2.drawChessboardCorners() functions.
All 20 images are listed in the "Read and Print the images - display corners" section of my ipynb notebook


Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. 


### Pipeline (single images)

#### 1. "Undistort camera image example" provides an exmaple undistorted image

I further applied cal_undistort() function on a test image. See the undistorted test image under 'Undistort Test Image' section of my notebook:


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I experimented with sobel abs, sobel mag, sobel direction gradient methods.
I also experimented with HLS, RBG and LAB color spaces.

Various thresholds were used for different gradients and channels.

The undistorted images and the respective binary transforms are shown in the respective sections of my notebook.



#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in 'Perspective Transform - warp function 'of the IPython notebook).  The `warp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
w,h are width and height of the image

src = np.float32([(575,464),
                  (750,464), 
                  (265,682), 
                  (1100,682)])

dst = np.float32([(450,0),
                  (w-450,0),
                  (450,h),
                  (w-450,h)])

```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 575, 464      | 450, 0        | 
| 750, 464      | 830, 0      |
| 265, 682     | 450, 720      |
| 1100, 682      |830, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a all test images.
I added it to the pipeline()
You can see it in the "Run Pipeline on All Test Images" section.



#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I computed a histogram of the bottom half of the image and found the bottom-most x position (or "base") of the left and right lane lines. These locations were identified from the local maxima of the left and right quarters of the histogram, just left and right of the midpoint. This helped to reject lines from adjacent lanes. I increased windows to 20, from which to identify lane pixels, each one centered on the midpoint of the pixels from the window below. This effectively "follows" the lane lines up to the top of the binary image, and speeds processing by only searching for activated pixels over a small portion of the image. Pixels belonging to each lane line are identified and the Numpy polyfit() method fits a second order polynomial to each set of pixels. The polyfit_using_prev_fit() leverages a previous fit and only searches for lane pixels within a certain range of that fit. 
Sections "sliding window poly fit' to 'Polyfit using previous frame' show both the output images and code for identifying lane-line pixels and fit their positions with a polynomial



#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

calc_curv_rad_and_center_dist() section of the notebook has this code.

curve_radius = ((1 + (2*fit[0]*y_0*y_meters_per_pixel + fit[1])**2)**1.5) / np.absolute(2*fit[0])
In this example, fit[0] is the first coefficient (the y-squared coefficient) of the second order polynomial fit, and fit[1] is the second (y) coefficient. y_0 is the y position within the image upon which the curvature calculation is based (the bottom-most y - the position of the car in the image - was chosen). y_meters_per_pixel is the factor used for converting from pixels to meters. This conversion was also used to generate a new fit with coefficients in terms of meters.

The position of the vehicle with respect to the center of the lane is calculated with the following lines of code:

lane_center_position = (r_fit_x_int + l_fit_x_int) /2
center_dist = (car_position - lane_center_position) * x_meters_per_pix
r_fit_x_int and l_fit_x_int are the x-intercepts of the right and left fits, respectively.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

"Draw the Detected Lane Back onto the Original Image" and "Draw Curvature Radius and Distance from Center Data onto the Original Image" sections of the notebook show the two images


---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)
https://github.com/buddha216g/CarND-AdvancedLaneFinding/blob/master/project_video_output.mp4


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

As i was learing the concepts from the class videos, i started writing the code in jupyter notebook and experimented with various features. Like thresholds, gradients, channels, combinations. May be i have to revisit this notebook and make it much more robust in terms of structure and content (esp lane finding and probably use a combination of gradient and color instead of just one), since i was not able to complete the challenge and harder challenge videos.

