## Writeup Template

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

[image1]: ./output_images/undistort.png "Undistorted"
[image2]: ./output_images/undistort2.png "Road Transformed"
[image3]: ./output_images/thresholded.png "Binary Example"
[image4]: ./output_images/warped.png "Warp Example"
[image5]: ./output_images/color_fit_lines.png "Fit Visual"
[image6]: ./output_images/detected.png "Output"
[image7]: ./output_images/formula.png "Formula"
[video1]: ./output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the fourth code cell of the IPython notebook located in "p4.ipynb"  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of S channel and gradient (Sobel x) thresholds to generate a binary image (thresholding steps in the 7th cell of the IPython notenook).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes functions called `warp()` and `unwarp()`, which appear in cell 9 of the IPython notebook. The `warp()` and `unwarp` functions take as input an image (`img`). They are generated using `define_warp()` function, which contains source and destination points. I chose the hardcode the source and destination points in the following manner:

```python
left1 = (170,img_size[0])
left2 = (591,450)
right1 = (img_size[1]-left1[0],img_size[0])
right2 = (img_size[1]-left2[0],left2[1])

offset = 300
    
warped_left1 = (offset,img_size[0])
warped_left2 = (offset,0)
warped_right1 = (img_size[1]-offset,img_size[0])
warped_right2 = (img_size[1]-offset,0)
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 170, 720      | 300, 720      | 
| 591, 450      | 300, 0        |
| 1110, 720     | 980, 720      |
| 689, 450      | 980, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used histogram and sliding window for finding line pixels on the first image. In the next frames I used pixels around previous line positions. With the pixels forming lines I found polynomial which could fit for these pixels. The code is contained in functions `find_lines()` and `update` of class Pipeline in cell 11 of the IPython notebook.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

For calculating the radius of curvature I used the following equation with polynomials corrected for real world:
![alt text][image7]

For calculating the position of the vehicle with respect to center I subtracted the position of the center between two lines from the center of the image.

I did this in `next_frame()` function of `Pipeline` class in cell 11 of the IPython notebook.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in `next_frame()` functon of `Pipeline` class in cell 11 of the IPython notebook.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My current approach for extracting line pixels with Sobel x and S channel doesn't work in some conditions. Especially it is affected by shadows on a road. To compensate it I used means from last 10 frames. It works well for the project video, but to generalize my solutions for challenge videos I should try other methods of pixel identification, for example using red or blue channel. I could also test new polynomials and add them to mean value only if they look like real lane lines.