## Project 2 - Advanced Lane Finding
### Steven Novotny

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

[image1]: ./output_images/undistorted.png "Undistorted"
[image2]: ./output_images/undistorted_test.png "Undistorted Test Image"
[image3]: ./output_images/thresholded_binary.png "Binary Example"
[image4a]: ./output_images/transformed_road.png "Road Transformed"
[image4]: ./output_images/warped_binary.png "Warp Example"
[image5]: ./output_images/lane_pixels.png "Fit Visual"
[image6]: ./output_images/curvature.png "Curvature"
[image7]: ./output_images/projected_lane.png "Lanes"
[image8]: ./output_images/examples_out.png "Output"
[video1]: ./video_output.mp4 "Video"

## Description of Key Steps in Pipeliine

The following provides a brief explanation of each step in the processing pipeline.

### Camera Calibration

#### 1. Computation of the camera matrix and distortion coefficients. 

The first block of code in the processing (contained in the p2.ipynb file) is used to calculate the camera matrix and distortion coefficients. This used a 9x6 internal corner chessboard pattern and creating a mesh grid based on this geometry.

I ran through the series of calibration images, using the openCV findChessboardCorner method. Each time corners were successfully detected, the object points and the image points were added to a list. That list was then sent to the openCV calibrateCamera method.

The following image shows the effects of undistorting a calibration image.

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

For the test images, the same approach was applied--invoke the openCV undistort() method. 

![alt text][image2]

#### 2. Using color transforms, gradients or other methods to create a thresholded binary image. 

The binary image was created by applying both color and gradient thresholding. This required significant experimentation and parameter tuning--something that makes me feel skeptical of the approach's robustness. The associated methods are shown in block 2 of the notebook.

The color thresholding was based on using both the RGB image and the HSV color conversion. The image was then filtering on the S-channel and the R-channel.

The gradient thresholding used x-gradient, y-gradient, gradient magnitude, and direction for filtering and creating a binary. 

Thresholding of these computed gradient values were then combined with the color thresholding binary to create images of the type shown below.

![alt text][image3]

#### 3. Applying warping to produce a bird's eye view of the road

To transform the images to an overhead view, I carefully examined the images that were described as "straight line" images. From these, I could identify multiple points on the source image that allowed me to create a region of interest. From these, I then chose points representing parallel lines in a destination image.

```python
src = np.float32([[593,450], [688,450], \
    [195,720], [1120,720]]) 
dst = np.float32([[320,0],[im_size[0]-320,0], \
    [320,im_size[1]],[im_size[0]-320,im_size[1]]])
```

An example of this process is shown below. The source and destination points are shown via the reference lines.

![alt text][image4a]

Applying all aspects of the pipeline so far, produced the following images
![alt text][image4]

#### 4. Identifying lane-line pixels and fitting their positions with a polynomial

As shown above, the lane lines are globs on the final warped image. From this perspective, I started with a histogram approach to identify starting points at the bottom of the image. From there, I applied a sliding window to get the initial 2nd order polynomial fits to the globs. On consecutive frames of the video, I used a method based on looking at the previously calculated polynomial fits.

An example of using this method on the above binary image is illustrated in the figure below. The polynomial fit is shown in yellow.

![alt text][image5]

In the event that the fitting became poor, I reverted back to the sliding window, and started over. The poor fitting was identified by extremely small radii of curvature. However, I think this could have also been accomplished by comparing the radii of curvature calculated for the left and right lane lines.

#### 5. Calculate the the radius of curvature of the lane and the position of the vehicle with respect to center.

Block 5 of the jupyter notebook calculates the radii of curvature for the lane lines as well as the position of the vehicle relative to the calculated lane center. The curvature displayed on the output image is the mean of the left and right curvature calculations, corrected for the pixel scale relative to the real world. The assumed vehicle location is the center of the camera frame. The result is shown below.

![alt text][image6]

#### 6. Projecting the lane onto the road

In block 6 of the notebook, the lane is projected back onto the image frame using the inverse distortion matrix. The result for a single frame is shown below.

![alt text][image7]


---

## Processing Pipeline

All of the above steps were combined into a pipeline() method and validated against all of the provided test images. The results are shown below:

![alt text][image8]


### Pipeline (video)

#### 1. Pipeline applied to video 
The pipeline performed reasonably well on the entire project video with no suggestions of catastrophic failures. In fact, it looked much like my Tesla display.

[link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Parameter Tuning

The parameter tuning for the color and gradient thresholding was challenging. Though the results were reasonably good, I doubt my choices optimized the channel selection or the thresholding values. It seems that perhaps a dynamic analysis of the overall color histogram could provide a means for determining more robust thresholding values. In addition, perhaps previous frame information could help guide such a search process.

#### 2. Lane Tracking

The pipeline performed poorly on the challenge problems, particularly when the road had many lines or sharp curves. I attempted to address the second problem by ensuring the sliding window never left the image and thus lead to poor polynomial fits, but I didn't have great success. Furthermore, I believe I could have used a more robust technique for monitoring my goodness of fit, to determine when my lane prediction was poor. In the current code, I only use the radius of curvature to estimate the quality of the lane finding--assuming very small values are erroneous. Again, I believe a better approach exists.

#### 3. Speed

The code seems slow. The processing was only about 5 frames per second. If this were to be used in real time, I would need to achieve a factor of 6 speed increase.