## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Advanced Lane Finding Project**

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

[ProjectCode]: ./AdvanceLaneFinding.ipynb
[ColorSpaces]: ./output_images/ColorChannels.png "Different Color Spaces"
[ColorGradient]: ./output_images/ColorGradientThreshold.png "Color and Gradient Threshold"
[LaneDetection]: ./output_images/LaneDetection.png "Lane Detection"
[PerspectiveTransform]: ./output_images/PerspectiveTransform.png "Perspective Transform"
[PolynomialFit]: ./output_images/PolynomialFit.png "Polynomial Fit Example"
[SlidingWindowFit]: ./output_images/SlidingWindowFit.png "Sliding Window Convolution Example"
[RegionOfInterest]: ./output_images/RegionOfInterest.png "Region Of Interest Example"
[UndistortChess]: ./output_images/Undistort.png "Undistort Chessboard Image Example"
[UndistortLane]: ./output_images/UndistortLane.png "Undistort Lane Image"
[ProjectVideoInput]: ./project_video.mp4 "Input Project Video"
[ProjectVideoOutput]: ./output_images/project_video.mp4 "Project Video with Lane Detection"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/rajatsharma01/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

This is the writeup file fro this project, however there are lots of details in jupyter notebook for this project. I would highly encourage to go through markdown comments and look at cell outputs in ![AdvanceLaneFinding.ipynb][ProjectCode].

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in ![AdvanceLaneFinding.ipynb][ProjectCode].  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![Undistoring Chessboard Image][UndistortChess]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![Undistoring Lane Image][UndistortLane]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in cell 5-7 of ![Jupyter Notebook][ProjectCode]). Below image shows how our lane image appear in different color spaces as it helps to use different color spaces for color thresholding.

![Various Color Sapces][ColorSpaces]

For Color Thresholding, I have leveraged ideas from [Joshua Owoyemi](https://medium.com/@tjosh.owoyemi/finding-lane-lines-with-colour-thresholds-beb542e0d839) for picking yellow and white lane images from RGB color space. I have also used Lab blue-green channel to select yellow line pixels. Here's an example of my output for this step.

![Color and Gradient Threshold Output][ColorGradient]

There is still too much of unwanted edges in the above output image. We can focus on region with lane lines only by masking off unwanted portion outside lane and between lane lines as well. We use similar masking method as provided in first Lane Finding project of Term1. Example output on above image:

![Region Of Interest][RegionOfInterest]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()`, which appears in cell 10 of [Jupyter Notebook][ProjectCode]. The `perspective_transform()` function takes as inputs an image (`img`). I chose the hardcode the source and destination points in the following manner:

```python
    src_pts = np.float32([[560, 470],
                          [xsize - 560, 470],
                          [xsize - 170, ysize],
                          [170, ysize]])
    
    offset = 400
    dst_pts = np.float32([[offset, 0],
                          [xsize - offset, 0],
                          [xsize - offset, ysize],
                          [offset, ysize]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 560, 470      | 400, 0        | 
| 720, 470      | 880, 0        |
| 1110, 720     | 880, 720      |
| 170, 720      | 400, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![Warping Lane Image][PerspectiveTransform]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used Sliding Window Convolution approach to detect lane lines in warped image and fit a second degree polynomial to pixels in narrowed down range by sliding video. Please refer to Lane Detection section ![AdvanceLaneFinding.ipynb][ProjectCode], details for sliding window search are mentioned in markdown notes as well.

Following is an example output for Sliding Window search and polynomial fitting:

![Sliding Windows Convolution][SlidingWindowFit]

Cell 11 and 12 implements sliding window convolution method as described in lessons, with a slight change though, it retuns a mask to slect pixels from the image, rather than pixels themseves, it helps me keep interface clean when fitting a polynomial to new frame just take an input mask to select pixels. This helps in processing frames of a window when we don't need to re-run sliding window search and can find pixels near vicinity of polynomial fit of previous frame. Cell 13 implements function to obtain mask from existing polynomial and following example shows how mask around this polynomial looks like:

![Mask of Polynomial Fit][PolynomialFit]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in Cell 14 of ![AdvanceLaneFinding.ipynb][ProjectCode]

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell 15 of ![AdvanceLaneFinding.ipynb][ProjectCode] in the function `get_marked_lane()`. Here is an example of my result on test images:

![Lane Detection][LaneDetection]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

To make use of previous fame knowledge, I had to introduce two new classes to make pipeline impementation stateful. Please refer to Video Processing in first markdown cell in ![AdvanceLaneFinding.ipynb][ProjectCode], which describes in details about these classes: `Line` and `LaneGenerator`. Implementation of these classes are present in cell 17 and 18 respectively.

Below is the link for output on ![ProjectVideo.mp4][ProjectVideoInput]

[![Project Video Output](https://img.youtube.com/vi/o8hAKpHU80A/0.jpg)](https://youtu.be/o8hAKpHU80A)

These are some details about video. On top right conrner, there is a perspective view of the image that was used during polynomial fitting for this frame. It shows some important information, which can help evaluate pipeline performance:
1. Green mask around line pixels represents search window, it switches between box like mask (sliding windows) to curve like mask (derived from previous fit)
2. Red pixels are line pixels selected from current frame for curve fitting.
3. Blue pixels are from previous 9 frames which we use along with new frame to create a smooth inertial fit. Note that these would appear as trail of red color pixels as these were current positions in previous frame.
4. Yellow fit line showing resultant fit that was ultimately selected for this frame.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here is summary of approaches that I took:
1. Using sliding window convolution to for lane detection.
2. Masking off unwanted regions outside as well as between lane lines
3. Defining a confidence factor of polynomial fit as 'vertical spread of pixels/horizontal variance of pixels'. A very low confidence factor is an indicator that we need to switch back to sliding window search and find new polynomial
4. Averaging (weighted) polynomial fit coefficients over last n frames. This helps in smoother fitting of polynomial to lane lines and avoids sudden changes due to fauly pixels.
5. Fitting polynomial by combining pixels that were used with previous n frames with new frames candidate pixels to fit the polynomial. This adds bias with previous frames fit and helps avoiding sudden shifts.
6. Reset search to sliding window if lane width and curvature radius of two lines differ by 20%

There are still some challenges as current pipeline isn't working well for the challenge video, as same lane has two shades, confusing our pipeline to think of it as a different lane.
