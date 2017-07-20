# **Finding Lane Lines on the Road** 
*Tom Chmielenski*
*July 18, 2017**

## Project 1 Goals


**Finding Lane Lines on the Road**

The goals / steps of this project were the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)
[image1]: ./debug_output/Grayscale.jpg "Grayscale"
[image2]: ./debug_output/GaussianBlur.jpg "Gaussian Blur"
[image3]: ./debug_output/edges.jpg "Edges"
[image4]: ./debug_output/masked_edges.jpg "Masked Edges"
[image5]: ./debug_output/HoughLines.jpg "Hough Lines"
[image6]: ./debug_output/LaneLines.jpg "Lane Lines"

[image7]: ./debug_output/HoughLines_after.jpg "Hough Lines After Modifying draw_lines()"
[image8]: ./debug_output/LaneLines_after.jpg "Lane Lines After Modifying draw_lines()"
[image9]: ./debug_output/soldWhiteRightSnapshot.png "Snapshot of SolidWhiteRight Video"

[image101]: ./test_myimages/BadLaneMarking.jpg "Bad lane Markings"
[image102]: ./test_myimages/DoubleYellowWhite.jpg "DoubleYellowWhite"
[image103]: ./test_myimages/IntersectionRoadMisalignment.jpg "IntersectionRoadMisalignment"
[image104]: ./test_myimages/LineBreakAtIntersection.jpg "LineBreakAtIntersection"
[image105]: ./test_myimages/NoDriveZones.jpg "NoDriveZones"
[image106]: ./test_myimages/NoLinesOnCountryRoad.jpg "NoLinesOnCountryRoad"
[image107]: ./test_myimages/OtherObstatcles.jpg "OtherObstatcles"
[image108]: ./test_myimages/PartialNewPaintedLines.jpg "PartialNewPaintedLines"
[image109]: ./test_myimages/TransitionBetweenMarking.jpg "TransitionBetweenMarking"
[image110]: ./test_myimages/WornLineMarkings.jpg "WornLineMarkings"

[image201]: ./test_myimages/RealityModel_LaneLines.jpg "RealityModel_LaneLines"


---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.


My pipeline ( find_lanes() ) takes a single image as a parameter and returns this image with two lane lines overlaid on it.  It consists of 7 steps, as follows:
 1.  **Grayscale Copy** - I create a grayscale copy of the original image.<p align="center">![alt text][image1]</p>
 2. **Convert Yellow Lines** - I create a mask out the yellow objects from the original image, add it to the white mask of the grayscale image, and merged with the greyscale image,  This helps detect yellow lines better in gray scale images. 
 3. **Gaussian Blur** - I smooth the grayscale image with a kernel_size of 5.<p align="center">![alt text][image2]</p>
 4.  **Apply Canny Transform** - I apply a Canny transform using a 1:3 ratio (1:2 or 1:3 ratio is recommended by Canny) to detect edges within the image.  My low threshold was 64 and my high threshold was 191.  <p align="center">![alt text][image3]</p>
 5.  **Masked the interest area** - I then create a 4 sized polygon mask to specify the area of image we are focusing on.  The area of focus is on the bottom half of the image. This area of interest is slightly smaller than 1/2 the image. If not, the line lanes may appear to intersect, but in real life, they mostly like will not.  The top trapezoid points are each located 40 pixels below the center point of the image. .<p align="center">![alt text][image4]</p>  
 6.  **Apply The Hough Transform** - Using rho=3, theata = 1 degree (radians = pi/180), min_line_len=50, max_line_gap = 100, I create the lane line image.<p align="center">![alt text][image5]</p>
 7.  **Overlay The Lane Lines on the Original Image** - Finally, I return an image with the weighted lane lines overlaid on the original image.<p align="center">![alt text][image6]</p>

Initially, this worked fairly well to identify the solid lane markings, but did not work with dashed lane markings.

However, to conver the dashed white lane markings, to a single line, I modified the draw_lines() function by determine the slope of each line adding in a slight tolerance to help avoid jumping lane lines between frames.  If the line slope was greater than zero, it belongs to the right lane, otherwise, the left.  The slope is reversed because of the image's upper left coordinate is (0,0) and y axis is positive going down.  Once the slopes and intercepts of all the lines in this image are calculated, I compute the average for both the right and left line slopes and intercepts.  I then add these average values to global array variables.  When processing video, I create a moving average of the last 3 right and left slopes and intercepts.  
Using Linear Regression on the divided right and left lanes points, we can determine the average slope for this frame.  Using intercepts, we can draw a right and left solid lane line.

Since a video is made of multiple frames, I create a rounding average of the slopes and intercepts.  For the initial 3 frames, I use the slope for the individual frame.  After 3 frames have been processed, I switch and use the rolling average and slope.  Using this rolling average, the lane lines drawn do not vary much, thus making it more appealing to the viewer.

Here are the solid lanes lines after the Hough Transform is applied.<p align="center">![alt text][image7]</p>

And here is the updated image.<p align="center">![alt text][image8]</p>

A Snapshot from the solidWhiteLines video. <p align="center">![alt text][image9]</p>

### 2. Identify potential shortcomings with your current pipeline

There are a number of potential shortcomings with this initial approach
1.  It will not detect the right hand shoulder of the road, if there is no painted lane.
2.  The lane line shown will become distorted, if the road curves sharply.
3.  It will not work in fog or snow covered roadways when the road marking is obscured.
5.  It will probably fail at intersections, where there are no lane marking in the middle of the intersection.
6.  Double yellow lines, or yellow no drive areas would probably reak havoc on this pipeline.

### 3. My Own Exploration

As I was driving around, I began to think of different scenarios that my pipeline would have to address, some of these include:

1.  Bad Lane Marking - sometimes a vehicle will override freshly painted markings.<p align="center">![alt text][image101]</p>
2.  Double Yellow Lines - sometimes there are double yellow lines versus a single yellow line.<p align="center">![alt text][image102]</p>
3.  Misalignment of roadways at intersection - sometimes the intersecting roads are slightly offset.<p align="center">![alt text][image103]</p>
4.  Occasional breaks in line markings<p align="center">![alt text][image104]</p> 
5.  Yellow No Drive zones<p align="center">![alt text][image105]</p>
6.  No Lane Markings on Back Country Roads<p align="center">![alt text][image106]</p>
7.  Vehicles and Other Obstructions - When other cars are also on the roadway, sometimes it is hard to see the lane markings.<p align="center">![alt text][image107]</p>
8.  Lane Markings have been Partially repainted - <p align="center">![alt text][image108]</p>
9.  Transition Between Line Markings - Easy to find conditions where lane markings change.<p align="center">![alt text][image109]</p>
10. Line Markings that need to be repainted.<p align="center">![alt text][image109]</p>

Note: attempt was made in my pipeline to address any of these possible conditions

For internal work purposes, I also began to look at detecting a lane line in a photogrammary model![alt text][image201]</p>

### 4. Suggest possible improvements to your pipeline

Some improvements that I could make to my pipeline, include:
1.  Create a better weighted algorhithm to reduce the smooth out the flashing of the lane lines.
2.  Avoid looking at the very bottom pixels to avoid any vehicle parts, as camera mounts that may appear in the pictures.


### 5. My concluding observations

If I had more time...
1. I would look into Jupyter Widgets to create interactive sliders to determine the optimal parameters, as I found that finding the proper settings for the gaussian_blur kernel_size and the Hough Transform parameters to be time consuming. 
2. For the challenge assignment, I would try:
	1.  I would try to order the right and left lines from the bottom to the top of the image, then try to approximate the lane line using a curve fit, rather than a linear regression.
    2.  For concrete bridge decks, I would continue to display lane lines with the same slope as last observed.

