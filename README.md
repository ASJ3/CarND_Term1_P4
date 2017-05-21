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

[image1]: ./output_images/9x6_found.png "Corners Recognized"
[image2]: ./output_images/dst_bf_after.png "Undistortion Results"
[image3]: ./output_images/road_undst.png "Undistorted Road Image"
[image4]: ./output_images/sobel.png "Sobel"
[image5]: ./output_images/hue.png "Hue"
[image6]: ./output_images/saturation.png "Saturation"
[image7]: ./output_images/lightness.png "Lightness"
[image8]: ./output_images/binary_pixels.png "Comparing Selections of Pixels for Lane Finding"
[image9]: ./output_images/original_bev.png "Unwarping Perspective"
[image10]: ./output_images/searches.png "Lane Lines Search Methods"
[image11]: ./output_images/road_drawings.png "Image Output"

[video1]: ./project_video_output.mp4 "Video"


### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in cells 1-5 of the IPython notebook located in "./Project_4_Final_Code.ipynb".  

I used OpenCV and calibration pictures to help me find the specific distortion from the camera used. The calibration images are all from checkerboards at different angles and distances. Lens distortion is clearly visible on most of them. 

I passed 20 gray images through findChessboardCorners(). The goal was to associate the position of the 9x6 corners of the checkerboard as found on the pictures, with the "real life" x,y,z coordinates of these points (since checkerboards are flat, the z coordinate here is always equal to 0).

Each time 9x6 corners were found on a picture, their picture xy coordinates were stored in a list called imgpoints, while their corresponding real-life coordinates were stored in a list called objpoints (this means objpoints stores multiple times the same list of real-life coordinates).

We can see the results of the function finding corners on a specific image by calling drawChessboardCorners(). Here is one of the results:

![alt text][image1]

Once the reference points are found on multiple image, the undistort() function from OpenCV can get rid of distortion on an image (see cell 6). Here below are before and after pictures:

![alt text][image2]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

To try to identify lane lines on the road, I used a combination of Sobel edge detection and filters in the HSL color model. I found out the HSL model (Hue, Saturation, Lightness) instead of the common RGB model, because it was better at differentiating lane lines from the rest of the road. 

I used numerous examples, with various lighting and road textures, to help me find the parameters that best isolate lane lines. Finally, I turned each image into a binary image to retain only the pixels that are most likely to indicate lane lines (see cells 15-18). See below the results for each type of filter:

![alt text][image4]

![alt text][image5]

![alt text][image6]

![alt text][image7]


Different filters are better than others for different road condition (e.g. brightness of pavement). For example, the Hue and Saturation filters are pretty good at picking the yellow lane line, whereas lightness is better at recognizing the white bands on the right line (Sobel is good for both lines). Combining the results from the different binary images can sometimes lead to 'noisy' results, with a lot of non-lane pixels picken up by certain filters. Therefore I created a function that only retains a pixel from the binary images if it is present in 2 or more binary images (see cell 23-25). See below an example when all the pixels are kept, or only those with 2 or more validations:

![alt text][image8]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code to change the perspective of the road is in warp() (see cells 9-10). This function will take an image, take 4 points on that image and stretch those points to 4 coordinates on the bird's eye view image. These 4 coordinates make a rectangle on the destination image. It is important to note that while perspective is dealt with on the destination image (with lane lines displayed as parallel), the distance ratios are not accurate (i.e. the image is 'squeezed' on the y axis). Here are images with lines imposed on each picture to show the perspective transformation:

![alt text][image9]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The next steps was to use the binary images to find a 2nd order polynomial that would create two lines, one for each lane identified by the pixels on the binary image. It is necessary to use a second order polynomial, as it can represent the curves seen on a sinuous road.

Using a binary image showing pixels that might belong to one of the lane, I first used sliding windows to select the pixels that are in the two vertical areas where we expect lanes to be on the image (see cells 27-28). Once each lane has been found, we can use a faster method using the 2nd order polynomial found in a previous frame to create a narrow boundary in which to search for points belonging to lanes on a new frame:

![alt text][image10]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

As explained previously, the bird's eye view image squeezes the road on the y axis. I used the distance references from class and online information about distance between lanes and lane markings in the US to do a pixel-to-meter conversion. Then I included these measurements on the final rendered image (see cells 31-32)


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.
I combined all the steps needed to process an image (i.e. undistort, change perspective, apply Sobel and HSL filters, find and draw lanes...) into a function called process_image(). The function accepts an image as input and will return a fully processed image showing the lane, as well as curvature and distance from center (see cell 36). Here is an example of a result image:

![alt text][image11]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I had to experiment a bit with the perspective reference points on the origin picture. I think the points I got are giving relatively good results, although I would be curious to see if there are points that will give an even better bird's eye view. For further development, I would consider two types of improvements: those that are visual, and those that are functional.

Visual improvements are only designed to give a more compelling rendering. The text used for curvature and distance from center could be smoother. The lane color and its transparency could be more appealing. I even saw examples where the middle of the lane was rendered as an additional line.

From a function perspective, I think there are ways to improve finding lane lines. Although I think my current method for validating pixels on binary images does a decent job, I noticed it might be better to give more 'voting power' to a certain filter based on the lane to recognize. By analyzing many images, I noticed that the lightness filter almost always recognize properly right lane markings. Therefore it would make sense to create code that gives it more weight when it comes to recognize the right lane line. The same goes for the saturation filter and the left lane line.

Finally, I think lane drawing would be improved by averaging the 2nd degree polynomials obtained from previous frames with the polynomials found with the current frame.
