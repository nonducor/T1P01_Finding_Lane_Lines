# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 7 steps:

1. Convert the image to grayscale
2. Apply a Gaussian blur of size 7
3. Apply a Canny edge detection with lower threshold of 40 and higher threshold of 130
4. Mask a region of interest corresponding roughly with the lower part of the figure, plus the perspective effect of the two lane markings.
5. Use a Hough transform with relatively lax parameters to identify as much as possible of the lane markings
6. Divides the lines found by the Hough transform basically in two clusters, with positive and negative gradients (respect. right and left lane marks). A few heristics are used to assign the lines that do not have a clear gradient or are too near the middle of the image.
7. Uses a least square approximated line for each side. This method is specially effective on the segmented lane marks.

A few comments about the edge and line detection:

1. I found that it was better to have more noise on edge detection and on the lines found by the Hough transform and to let the least squares method "remove the noise" than to try to generate a very clean input for this method. It ends up being more robust, especially on the segmented stripes.

2. Considering the mounting of the camera, the lane marks are assummed to always start on the bottom of the image (largest y) and are drawn up to the smallest y found by the Hough transform.

3. The Canny edge detection was applied before masking the image, to avoid artifacts produced by the mask.

Instead of changing the `draw_lines()` function, a new function named `find_lines()` was created and run before `draw_lines()` on the pipeline. This allows `draw_lines()` to be used to draw both the detected lines as well as the Hough transform lines. That was very useful while debugging the system (this feature is usually turned off by the parameter `debug_video`).

The `find_lines()` function basically follow the suggestion given by the instructors on the comments:

1. First, the Hough lines are separated according to gradient for the left and right side.
2. Then, some heuristic separation is done: if the lines are start before the pixel 450, they are forced to be assigned to the left lane. If they are above 500, they are forced to the right. If they are in the region in between but are too horizontal (less than approx. 6 deg), they are simply ignored.
3. Then, for each side, all lines are resampled and the points are recorded. This has the advantage that long lines will have more samples, thus will have a higher weight on the final line.
4. Finally, a simple least squares method is used to find the best line for each side of the lane.

The following images show the pipeline in action:

![Canny edge detection showing only the region of interest][test_images_output/01_CannyDetection.png]

![Hough lines (red) and least squares line (green)][test_images_output/02_HoughLinesAndInferredLaneMarks.png]

![Hough lines (green) and least squares lines (red) superimposed to the original image][test_images_output/03_HoughLinesAndInferredLaneMarksOnImage.png]

Note that a ruler was also drawn on the top of the last image.


### 2. Identify potential shortcomings with your current pipeline

A great shortcoming of this pipeline is that it makes too many assumptions about the geometry of the lines, such as: that they are mostly straight, that the car is reasonably near of the center of the lane and also that the camera perspective and the horizon position are well defined. Also, the algorithm is very dependent on the illumination of the scenes. When tested on the `challenge` video, it failed completely.

All parameters of the edge detector and Hough transform were adjusted by trial and error, thus it is very probable that, with a different test set, the algorithm will not perform as expected. Also, no tests were done in images where there is a car in front of the camera. This could "confuse" the algorithm.


### 3. Suggest possible improvements to your pipeline

A possible improvement to make the algorithm less dependent on the position of the lane markings would be to try to cluster the Hough lines according to orientation and apply the least squares only to the lines that are at a given distance from the two biggest peaks. Horizontal lines could be excluded to avoid artifacts due to horizon, overpasses etc.

Another potential improvement could be to try to recognize the pattern of the asphalt just in front of the car and use it to mask the image, so that only this area is searched for lane markings.

Finally, the presented algorithm is totally memoryless, so it cannot use the previous frames in the movie to help filter out detection problems. This could filter some "single-frame" issues that sometimes appear.

