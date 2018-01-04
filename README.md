# **Finding Lane Lines on the Road** 

## Project Goal

[//]: # (Image References)

[original-images]: ./examples/Pipeline_Output/original_images.png "Original images"
[grayscale-images]: ./examples/Pipeline_Output/grayscale_images.png "Results of the grayscale transformation"
[gaussian-images]: ./examples/Pipeline_Output/gaussian_images.png "Results of the Gaussian blurring"
[canny-images]: ./examples/Pipeline_Output/canny_images.png "Canny algorithm output"
[roi-images]: ./examples/Pipeline_Output/roi_images.png "Results from clipping the image to focus only on our currrent lane"
[overlaid-images]: ./examples/Pipeline_Output/overlaid_images.png "Final output of the lane detection pipeline"

---
The goal of this project was to make a pipeline that can identify road lanes using "classical" (i.e. pre-deep learning) techniques, such as Canny edge detection and Hough transforms. In this case, the Canny and Hough techniques were implemented using the Python [OpenCV](https://docs.opencv.org/2.4/index.html) package.

In particular, this pipeline was required to successfully detect the lane lines on six images, two videos, and one video that has unique characteristics that increase the difficulty of successfully identifying the lane lines in it.  You can see the original six images below. The video annotations resulting from the pipeline can be found [here](https://www.youtube.com/watch?v=h9yxkKlpAZQ), [here](https://www.youtube.com/watch?v=tYHS5H19VYg), and [here](https://www.youtube.com/watch?v=ElnK4uAFvIY). 

![alt text][original-images]

---

### Reflection

### 1. Pipeline Description

The pipeline consists of five steps:
    1. Grayscale transformation
    2. Gaussian blurring
    3. Canny edge detection
    4. Image clipping
    5. Hough Line Transform


#### Step 1. Grayscale Transformation
As some computer vision algorithms only work on black-and-white imagery (in this case the Canny algorithm), I first convert the image to grayscale. You can see the result of this transformation in the set of images below.

![alt text][grayscale-images]


#### Step 2. Gaussian Blurring
Then, I apply Gaussian blurring to the images in order to reduce noise in the images themselves.

![alt text][gaussian-images]


#### Step 3. Canny Edge Detection
After reducing the images' noisiness, the Canny algorithm is used to detect edges within the image. As you can see in the following results, we are able to see outlines for the lane lines that I am interested in tracking. However, Canny is also picking up the outlines for other objects and markings in the image.

![alt text][canny-images]


#### Step 4. Image Clipping
To get around the problem I just described, the pipeline only looks at the lane we are currently in, and only up to a certain distance in front of the vehicle. This is achieved by simply clipping the image to focus on this region of interest.

![alt text][roi-images]


#### Step 5. Hough Line Transform
The Hough Line Transform is an operation to detect straight lines. Thus, given the previous step's output, its output provides the coordinates indicating the location of the various lane line segments. Unfortunately, these are still just chunks of the lane. Rather than providing the location of the entire lane itself (as would interest a driver), it will show gaps between dashed lines and fail to recognize  those dashed lines as components of a complete lane. 

That missing logic is completed by interpolating the lines location from the bottom of the image to the top of the region of interest. To do this, I first use the coordinates from the Hough transform to calculate that line's slope in order to determine whether it is part of the left lane or right lane (or whether it is part of a lane at all). Then, once all the lines have been assigned to a lane (or discarded), I calculate the average position of all lines within a lane. Using this average, I can then interpolate the function representing a given lane, and thereby determine its beginning and ending points within the region of interest. The pipeline then successfully detects the lane lines in the images.

![alt text][overlaid-images]


### 2. Potential Shortcoming with this Pipeline
If you look at the images and watch the first two videos, you will probably notice similar image characteristics:
    * No objects (e.g. vehicles, animals, people, etc.) are in the lane in front of the vehicle, and within the region of interest.
    * The roads' color does not vary.
    * Lanes are not unmarked.
    * Similar weather conditions.
    * Large shadows are not covering the vehicle's lane.
    * Road markings are based on a California (i.e. United States) system.
There are likely more characteristics that I have not mentioned, but you get the idea at this point. Any deviations from these characteristics would cause either the Canny edge detector or the Hough Line Transform to either miss lane lines or to have false positives that there are lane lines in some locations. When you watch the [third video](https://www.youtube.com/watch?v=ElnK4uAFvIY), you can see some of these issues affecting the lane line detector.


### 3. Possible Improvements to the Pipeline
There are ways to mitigate some of these shortcomings. One way would be to utilize a different color transformation early on in the pipeline that helps mitigate the issue of lighting (such as detecting yellow or white lanes). Another approach would be to collect a larger set of images under varying conditions and modify pipeline parameters to make it more generally robust across all these images. It is also possible that additional rules could be added to the pipeline that are only activated under certain road conditions.

Unfortunately for all these potential improvements, we can clearly see that this quickly becomes a rule-based approach to lane line detection, which is difficult to maintain and ensure robustness across various driving situations. Ideally, we would want a way to detect lanes utilizing machine learning techniques...