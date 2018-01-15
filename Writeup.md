
# Self-Driving Car Engineer Nanodegree

## Project: Advanced Lane Finding


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

[image]: ./camera_cal/calibration1.jpg "Distorted"
[image1]: ./output_images/undist_calibration1.jpg "Undistorted"
[image2]: ./test_images/test2.jpg "Road Untransformed"
[image21]: ./output_images/undist_test2.jpg "Road Transformed"
[image3]: ./output_images/colorbinary_test2.jpg "Color Binary Example"
[image4]: ./output_images/warped_test2.jpg "Warp Example"
[image5]: ./output_images/outimg_test2.jpg "Fit Visual"
[image6]: ./output_images/result_test2.jpg "Output"
[video1]: ./test_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "P4 - Advanced Lane Finding Final.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. The original images is as follows:

![alt text][image]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

By using the calculated distrotion coefficients and camera matrix, the test image can be undistorted as can be seen below (red bounding lines are also applied to ensure the vertices used for transformation and warping engulf the lane lines):
![alt text][image21]


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image using sobelx, S channel, magnitude and direction thresholds. This "image_process" function can be found in cell no.6, with the code to produce results for all test images in the following cell no.7.   Here's an example of my color output for this step.

![alt text][image3]

The actual grayscale images are saved in "./output_images/" and can be read in with cmap = 'gray'.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 23 through 33 in the 3rd code cell of the IPython notebook.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[550,470],[750,470],[1250,750],[100,750]])
dst = np.float32([[320, 0], [980, 0], [980, 720], [320, 720]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 550, 470      | 320, 0        | 
| 100, 750      | 320, 720      |
| 1250, 750     | 980, 720      |
| 750, 470      | 980, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then in, cell no.9, I defined a function took a histogram of the warped binary image to identify the peaks, and thus lane locations, followed by a sliding window search for the rest of the image to trace the lanes' locations (x, y coordinates). After determining all the x,y coordinates for the left and right lanes, I fit my lane lines with a 2nd order polynomial, and in cell no.10 applied it on the test images resulting in images like this:

![alt text][image5]

note: the actual lines fitted can be viewed in the output of the .IPynb file.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In cell no.11, I defined the function that calculated the radius of curvature of the lanes, which utilized knowledge of the size of the image, US standard lane size, conversion factors and the polyfit lines that were generated. The function returns the radius of curvature for each lane, as well as the value of the center of the lane. In cell no.12, I generated the following values for the test images:

test1: 1105.48509511 m 521.166774283 m 0.34644628418 m

test2: 1479.30217568 m 484.112347388 m -0.13957938038 m

test3: 2885.34115 m 1921.02997725 m 0.382131521768 m

test4: 3169.89862259 m 487.888667151 m 0.281559451424 m

test5: 1281.07297816 m 611.481311465 m -0.0608358669085 m

test6: 2147.21547042 m 470.977797356 m 0.548764879469 m

test7: 10545.3982411 m 4037.39007583 m -0.0695567681181 m

test8: 82590.94076085 m 25379.3542181 m -0.0786961326045 m

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cells no. 13 and 14 with the functions add_text() and draw_lines(). Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The basis of accurate and successful lane detection is proper transformation/warping of images, accurate tracing of lines and polynomial fitting. One of the most difficult tuning problems was to identify the most suitable source points, that guaranteed the englufing of the lane lines, but reduced the noise and irrelvant features in the warped transformation. Furthermore, more rigorous image processing for lane identification would allow for more accurate results.

Another issue is the calculation of the curvature radii, where in the video stream, they may increase dramatically. This issue appears when the lanes straighten out, i.e. the curvature is 0, however, due to slight errors and rough image transformation and warping, the output value is significantly large, which in this case, is representative of a more or less straight lane. As such, a possible improvement would be implement a maximum threshold for the radii values, after which the code replaces it with 0, indicating a straight lane.

Possible failure scenarios and their solutions:
1.Double sided lane lines, which would require finer tuning of the region of interest for image processing, or a smaller margin of sliding search
2.Obstructions, such as sand or snow, to the visibility and identification of the lane lines, which would require more rigorous image processing and broader range of sanity check, as well as possible use of other guiding features in the image, such as a road divde.

##### References:
1.Udacity lessons and code for functions

2.priya-dwivedi for sanity check implementation
