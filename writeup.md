# Detecting Lane Lines
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

[image1]: ./output_images/undistorted.png "Undistorted"
[image2]: ./output_images/undistorted_road.png "Undistorted Road"
[op-1]: ./output_images/transform_S.png "Transform_S"
[op-2]: ./output_images/transform_L.png "Transform_L"
[op-3]: ./output_images/transform_sobel_x.png "Sobel_x"
[image4]: ./output_images/perspective.png "Warp Example"
[image5]: ./output_images/line_detect_early.png "Fit Visual"
[image6]: ./output_images/output.png "Output"
[gif]: ./output_images/output.gif "GIF"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.
You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second section of the Jupyter notebook located in "./main.ipynb"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `target_corner_array` is just a replicated array of coordinates, and `target_corner_list` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `actual_corner_list` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `target_corner_list` and `actual_corner_list` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![distortion][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![distortion][image2]
(Note: it's hard to recognize the difference, but it's distortion-corrected)


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of HLS color space thresholds and sobel x gradient thresholds to generate a binary image. Here's an example of my output for this step.

**S space threshold**
![alt text][op-1]
**L Space Threshold**
![alt text][op-2]
**Sobel x gradient threshold**
![alt text][op-3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper(img)`, which appears in the section 5.2.  The `warper(img)` function was generated from `warper_factory` and it takes inputs source (`src`) and destination (`dst`) points and returns a function `warper(img)` and another function `unwarper(img)`. Internally, it uses `cv2.warpPerspective(image, ...)`.  I chose the hardcode the source and destination points in the following manner:

```python
H = 720
W = 1280
src = [[590, 450], [W*0.1, H], [1150, H], [700, 450]]
dst = [[W*0.1, 0], [W*0.1, 720], [W*0.9, 720], [W*0.9, 0]]  
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 590, 450      | 128, 0        | 
| 128, 720      | 128, 720      |
| 1150, 720     | 1152, 720      |
| 700, 450      | 1152, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

My pipelines are as follows:

- Apply Gaussian Blur with a filter size of 3
    - It helps to reduce noises in the image
- Take binarized images thresholding on sobel x & y gradients
- Take binarized images thresholding on L & S of the HLS color space
- Sum these images to a single image
- Binarized the image
- Warp the perspective
- Slide window search
    - Divide the image by multiple "windows"
    - Slide the "window" and find the most activated pixel by making a histogram
- Find the 2nd order polynomial
- Compute curvature radius
- Draw

After finding the polynomial plot, it would look like this:  
![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

It can be found in the section 7 in the Jupyter notebook.
More detailed explanations can be found in [my other notebook](https://nbviewer.jupyter.org/github/kkweon/opencv-exercises/blob/master/distortion_perspective.ipynb#12.-Measuring-curvature)

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the section 8 of the Jupyter notebook, and some preview can be shown below:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's some result:
![gif][gif]

And, [link to the video](./output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Currently, my pipeline is liekly to fail when there are too much light/shadow because it is hard to differentiate lane lines. I tried to overcome this problem by making it remember the previous line curvature. For example, if it does not find any lines, it will use the previous detected line information. However, if it's not able to detect for a longer period, it's not able to work properly. I think I have to modify the transformation step in order to make it more robust.