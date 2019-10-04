## Advanced Lane Detection Project #2

### This project is to demonstrate how to detect the lines from images/video by applying the lessons learned from the course.

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

[image1]: ./camera_cal/calibration2.png "Calibration image"
[image2]: ./test_images/test4.jpg "Input Image1 - for Slide Window"
[image3]: ./test_images/test3.jpg "Input Image2 - for Search Around"
[image4]: ./output_images/camera_calibration.png "Calibration - Original & Undistorted"
[image5]: ./output_images/calibration2_undistort.jpg "Calibration2 Image Undistorted"
[image6]: ./output_images/input_image1_warped.jpg "Input Image 1 Warped"
[image11]: ./output_images/color_threshold.jpg "Color Threshold"
[image12]: ./output_images/combined_threshold.jpg "Combined Threshold"
[image13]: ./output_images/gradient_threshold.jpg "Gradient Threshold"
[image14]: ./output_images/histogram.png "Histogram"
[image15]: ./output_images/input_image1_slide_window.jpg "Input Image 1 Slide Window applied"
[image16]: ./output_images/input_image2_search_from_prior.jpg "Input Image 2 Search from the prior Fit"
[image17]: ./output_images/search_around_prior.png "Search around from the prior Fit"
[image18]: ./output_images/draw_lane.jpg "Draw Lane detection"
[image19]: ./output_images/pipeline.png "Pipeline test"
[image20]: ./output_images/pipeline_auxdata.png "Pipeline test with curvature radius and center distance"
[video1]: ./project_video.mp4 "Video"
[video2]: ./project_video_output.mp4 "Video_line_detected"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./P2.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  For the same, the test images will also be converted into gray image. The 'objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time all chessboard corners in a test image are detected.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![calibrated_image][image4]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The 'cv2.calibrateCamera()' function as described above returns the camera matrix and distortion coefficients, and then these shall be used to distort the test image for a distortion-corrected image. 

The result image is the same as above.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image for lane edge detection.  
The thresholding functions are defined in the section of "Third, Color and Gradient thresholding". 

For the color threshold, the RGB image was converted to HLS color scheme first, and only S channel was filtered after that. S channel is especially useful in detecting yellow line of the lane to complement the edges detected from the gradient threshold using Sobel operation. 
For the gradient threshold, both X-axis oriented and directional (with angles) operations are used.

Before combining both, each thresholding (color and gradient) has been tested and compared in the section, "Color/Gradient Thresholding Verification (1)".

Each thresholding result images are as follows.
![Original Image][image2]
![Color Threshold][image11]
![Gradient Threshold][image13]

The combination of gradient and color thresholding has been applied and tested in the section, "Color/Gradient Combined Thresholding Verification (2)". 

Here's an example of the combined thresholding of color and gradient thresholds. 
![Combined Threshold][image12]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The perspective transformation has been performed in the section, "Fourth, Perspective transform from binary image to "birds-eye view"".
The cv2.getPerspectiveTransform() takes the source and destination and returns the transformation matrix which will be used to transform the input lane image into a bird-eye view image where the lane detection shall be made.

For the source which is the original camera image where there is a straight lane, 4 points will be selected from the lane. These 4 points should be the rectangle representing the parallel line when viewed from the bird-eye perspective.
And, then the cv2.getPerspectiveTransform() shall return the perspective transformation matrix needed to convert from the camera view to the bird-eye view.

I chose the source points manually from the test image, and destination points as the way I want to be displayed as the straight line on the bird-eye view image.
The source and destination points selected are as follows.

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 570, 470      | 320, 1        | 
| 270, 720      | 320, 720      |
| 1150, 720     | 960, 720      |
| 730, 470      | 920, 1        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel on the warped image in the bird-eye view. 

The source and resulting bird-eye view are as follows.
![Camera View][image2]
![Birdeye View][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The method to identify lane-line pixels using the slide window approach is implemented in the section, "Finding the Lines: Slide Window". 
This slide window method is applied when there is no prior polynomial Fit information.

Here, I used 10 slide windows and for the initial window position for the line detection, histogram has been used which gives the best X positions where more line pixels are located. 
![histogram][image14]

In each slide window, pixel positions will be recorded and it will determine the next window based on the average X value of the pixels located in the current window. It will continue till the last slide window.

The line pixel detection applied using the slide window is shown in this image. 
![Slide Window applied][image15]

When there is a prior Fit information is available, it will take advantage of the Fit information instead of slide window approach to find the line pixels. 
It will check all the pixel information against the polynomial Fit with the margin (+- 100), and only the pixels are within the range will be recorded as line pixels.
The implementation is in the section, "Finding the Lines: Search from Prior".

The line pixel detection applied using the prior polynomial Fit is shown in this image.
![Prior Fit applied][image16]

With either method, the line pixels would have been recorded in (leftx, lefty) for the left line of the lane and in (rightx, righty) for the right line of the lane.
With those pixels, the polynomial Fit can be calculated using "np.polyfit()" as below.

    left_fit = np.polyfit(lefty, leftx, 2)
    right_fit = np.polyfit(righty, rightx, 2)
  

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.
The calculation for the radius of curvature of the lane and the vehicle position is implemented in teh section, "Seventh (7th), Determine the curvature of the lane and vehicle position with respect to center".

The equation used here is as follows.
ym_per_pix = 30/720   # meters per pixel in y dimension
xm_per_pix = 3.7/700  # meters per pixel in x dimension
        
// y_eval: y-value where we want radius of curvature (the bottom of the image in this case)
y_eval = np.max(self.ploty)
curverad = ((1 + (2*fit[0]*y_eval*ym_per_pix + fit[1])**2)**1.5) / np.absolute(2*fit[0])

For the vehicle distance with respect to center, the assumption is that the current vehicle location is at the center of the bottom of the image in X-axis, since it's a camera view. 
Using the polynomial Fit info, we can find X positions of both lines given the bottom Y.

L-Line X position: left_fit[0]*h**2 + left_fit[1]*h + left_fit[2],
R-Line X postion: right_fit[0]*h**2 + right_fit[1]*h + right_fit[2],
where h is the image height (bottom of the image).

The middle position of L-Line X and R-Line is a line center, and we can compare this with the image X center to find out the position of the vehicle with respect to center.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the section, "Eighth (8th), Draw the detected lanes on the original image". 
First, I drew the filled polygon area using "cv2.fillPoly()" with the polynormial Fit pixels identified.
And, then using the inverse perspective matrix, the lane area (filled polygon) has been warped back on to original image.

Here is an example of my result on a test image:
![draw lane area][image18]

More pipeline tests with the lane area identification is shown below.
![pipeline][image19]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here is a link to my video result (./project_video_output.mp4).
![pipeline with video][video2]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The most difficulty I have with is determining whether the lane (lines) detected is valid; two lines are relatively parallel, two lanes have similar curvature, two lines are evenly spaced, and such. 
The current solution is not appropriate at this moment, since I observe that the slide window approach has been used only once at the start of the video, and since then it was never invoked. 
I would need to revisit this line validity checking function. 

I would also appreciate if you could provide some valid solution/algorithm in verifying the line validity.
Thanks!
