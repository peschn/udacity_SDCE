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

My pipeline consists takes an image as an input and output lane lines drawn on the image.  
First, I convert the images to grayscale, as this step makes it possible, to work with Canny Edge detection. Before edges can be detected, it is import to apply a Gaussian filter on the image, to smooth the gradients between pixels. This helps to reduce the impact of spurious gradients, hence reduces the error in Canny edge detection. In Canny edge detection, edges are filtered out of the image, which are the input for the Hough algorithm. The Hough algorithm allows us, to detect lines in an image and returns the start and end coordinates of each line segment.

In the next step, the hough_to_lanes function detects whether a line from the Hough algorithm belongs to the right or the left lane. To do this, I follow multiple steps. First I compute the gradient of each line and convert it to its absolute angle. Lane lines are most likely in the range between 20 degrees and 150 degrees (in absolute values). Using this selection helps to reduce the impact of detected lines that are not part of a lane, but rather of some shadows or other things. I choose 20 to 150 degrees also, as it makes it possible to detect small curvature in the lanes later on in my algorithm. Once the lane line points are detected, I sort them into right and left lane, by checking, in which quadrant they are. 

Once I have the points for the left and the right lane, I feed them into the draw function of the Lane class (I have written a class, to make it easier to work with a buffer). The draw function fits a polynomial of degree two to the lane points and draws it onto the image. I chose this type of polynomial to account for curvature. But this selection comes with two problems: 1. it is not stable, for the not solid lane line. 2. If there are two many lines which are detected by the Hough algorithm, that do not belong to a lane, than the fitted polynomial is biased. To reduce this problem, I added a buffer that captures the lane line points for 100 frames, which is about 4 seconds. It computes the polynomial based on the new and the buffered points. This helps to alleviate issue 1 and reduce the impact of issue 2. In the last step the lanes are drawn onto the image.   

![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline

There are many shortcomings to the pipeline:

1. Different light conditions, especially shadows make the pipeline unstable and lanes are not detected correctly.
2. If the road surface switches, or if there is tar on the road, e.g. from road repairs, edges and lines are detected, that do not belong to a lane, but but bias the estimation of the polynomial.
3. My method to assign points either to the left or the right lane does only work for lanes with no to small curvature. As soon as it becomes too large, points are assigned to the wrong lane. 

Points 1 to 3 are very obvious in the challenge video.

4. My pipeline is not able to deal with junctions, if the car switches lanes (due to the selection of lanes within a 20 to 150 degrees angle) or very sharp curvatures.
5. The vertices that define the area where we expect lane lines is fixed, so if the car hits a bump, the vertices are out of place and detection becomes not reliable anymore.
6. My pipeline is not agnostic the the aspect ratio of the video stream. If this changes, the scale factor can't adapt the polygon of the region-of-interest not properly anymore.
7. If there is a car in from of our car, there will be many lines within the region of interest detected that makes the prediction of the lanes hard (due to the additional noise).
8. I did not modify the draw_lines function as requested. The reason is, that I found it easier to implement a class to deal with my buffer, than having a nested function.

### 3. Suggest possible improvements to your pipeline

1. Use color information, to select if a line, that was detected by the Hough algorithm, is part of a lane or not. If it is in the yellow or the white spectrum (or in close distance to white or yellow pixels), it is likely, that it is part of a lane. Distance e.g. could be measure by Euclidean distance.

2. Weight points that are part of a lane and that are in the lower part of the image, higher in the polyfit regression, than points that are close to the center of the image. It seems from the video streams, that it is much more noisy in the center of the image than at the bottom.


