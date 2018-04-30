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

[image1]: ./writeup/checker_original_v_undistorted.jpg "Checker Original vs Undistorted"
[image2]: ./writeup/road_original_v_undistorted.jpg "Road Original vs Undistorted"
[image3]: ./writeup/combined_binary_example.jpg "Combined Binary Example"
[image4]: ./writeup/mask.jpg "Masked Image"
[image5]: ./writeup/undistorted_src_points.jpg "Undistorted with Source Points"
[image6]: ./writeup/warped_dst_points.jpg "Warped with Destination Points"
[image7]: ./writeup/fit_poly.jpg "Fitted Polynomials"
[image8]: ./writeup/highlighted_lane.jpg "Highlighted Lane"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the cell # 2, 3 of the IPython notebook located in "./code.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cal_undistort()` function defined in cell # 3 which in turn uses `cv2.calibrateCamera()` and `cv2.undistort()` functions.  I applied this distortion correction to the checkerboard test image and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I then used the same obtained objpoints and imgpoints with the function `cal_undistort()` on test image No. 5, and obtained this result:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of gradient thresholds, saturation thresholds (in HLS space), value thresholds (in HSV space), and lightness thresholds (in HLS space) (to deselect shadows) to generate a binary image (thresholding steps in cell # 5, 7, 8, 9, 10, 11).  Here's an example of my combined output for this step.

![alt text][image3]

I then defined a region of interest in cell # 12 and a function to mask images in cell # 13, and used it to apply a mask to filter out all irrelevant parts of the image, which would otherwise just confuse the algorithm. Here's an example of my output for this step.

![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `bird_eye()`, which appears in cell # 16, which takes in an image, as well as source (`src`) and destination (`dst`) points from cell # 14, 15.  I fine tuned the source and destination points manually, by plotting the polygons formed by both the source and destination points, which lead to the following points.

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 120, 719      | 350, 719      | 
| 577, 455      | 350, 100      |
| 705, 455      | 950, 100      |
| 1200, 719     | 950, 719      |

I verified that my perspective transform was working as expected by testing on a straight line image, and drawing the `dst` points onto the warped counterpart to verify that the lines are parallel and perpendicular in the warped image.

![alt text][image5]
![alt text][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I used the convolution sliding window search method in cell # 18, with some modifications to enable the algorithm to just skip a layer and put None for the found window position, instead of having to always report the position of the highest convolution output which might very well be just noise, so I chosen a pixel threshold (in this case = 100, but it depends on the chosen window width and height), and in this case the windows are detected only if the highest convolution output exceeds that threshold.

Then I defined a function in cell # 19 to get the x and y coordinates of the found windows and append them to left and right x & y coordinates lists (if window position is None then don't append anything for that layer for that side (left or light)).

Lastly I fitted 2nd order polynomials for each of the left and right lines based on their x & y coordinates lists. giving the output below.

![alt text][image7]

After that I defined a function in cell # 21 to search nonzero points around the fitted polynomials, which will be used for both coloring the lines in cell # 22, and as a second search algorithm that will be used in the pipeline after the lane lines are initially detected by the convolution window search algorithm, similarly the `get_coordinates()` function can be used again but on the found pixels instead of the found windows, and then 2nd order polynomials can be fitted around the found pixels.

#### 5. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

In cell # 23 I defined a function to highlight the area between the 2 fitted polynomials in green to clearly show the detected lane, then I warped it back in cell # 24 to the undistorted road image, by swaping the dst and src points in the same `bird_eye()` function. giving the below result

![alt text][image8]

#### 6. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In cell # 26 I defined a function to calculate the radius of curvature in meters, by refitting different polynomials after multiplying by the x & y scale correction factors, and then applying the radius of curvature equation.

In cell # 27 I defined a function to calculate the offset of the vehicle by measuring the distance in pixels between the center of the lane and the center of the image and multiplying it by the scale correction factor in the x direction.

---

### Pipeline (video)

#### 1. In addition to the above, I defined the class that holds the lane line characteristics and history of detections, and a sanity check function that discards the detections that seems to be unreasonable by measuring the distance between the left and right lines on multiple equally spaced points and dismiss the detection if the distance is too big or small, considering instead the last valid detection. I then used all of these functions in a single pipeline function and fed it the video images.

#### 2. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./writeup/project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The algorithm doesn't work perfectly in the shadowy and bright parts of the road, it could benefit from some fine tuning for the threshold values used, also the value threshold can be made adaptive (to be increased for bright frames and reduced for dark frames).

The detected lines wobble a bit, especially near the bottom of the image, which can be corrected by averaging the output over the last n frames, instead of showing each output individually, which will also be very helpful for showing an accurate radius of curvature.

The sanity check function works well for discarding bad detections, but it can be further improved by comparing the radii of the curves, and discarding the detection if the radii are very dissimilar, also it could be made to check the left and right curves separately, as for example the left curve might be bad, but the right one might be perfectly good so no need to discard it, however that will require more complex criteria like comparing the detection to the previous frames, and checking for sudden changes in order to know which of the curves is the bad one.
