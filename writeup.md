**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps:
1. I converted the images to grayscale. This is required to enable algorithms such as Canny to find edges by monitoring the change in color
2. The next step was to run the Canny algorithm which basically scans the image to find changes in color, and more permanent the color change (as in there is a color change and it is wide in dimensions), the larger Canny reports as the change. It is this change index that is used subsequently to detect lanes
3. The next step is to apply a Gaussian blur operation. My understanding is that the purpose of this is to eliminate stray marks. If there were 1 or 2 pixels that showed a change but neighboring pixels did not register the change, then blur would diminish the value of the change, enabling the actual changes in color to stand out
4. Next we mask out the regions that are not of interest. I did thi by assuming that the lanes are in the foreground and they go as far up as the midline of the image. So I retained only the bottom half. Further, I assumed that the lanes would intersect (due to the camera's perspective). I assumed this point to be in the middle of the frame. I allowed for some space on either side of this midpoint and that formed my region of interest. I only retained data that was within this area.
5. The last transformation applied was the Hough transformation. What this does is coalesce multiple line segments that are part of the same line (same slope, same intercept) to be represented by a single point, and by filtering the number of segments that map to the same point (the threshold parameter), I was able to achieve a good approximation for what represents the lanes.

At this point, I had the lane markers marked out. However, the challenge was to be able to extend the individual segments into a single lane. The difficulty here is to find which lines represent the lanes, since inspite of the filtering, etc. the transformations still retain stray lines and many of them have the same slope as the lane markers themselves. First I filtered out lines that had slopes that did not seem right. By inspection, I determined that the lanes show up within some angle range, so I filtered out lines that did not have slopes in the corresponding ranges. The next step really took me a long time - filtering the remaining lines to a smaller set that has a consistent slope and intercept so that I can get an equation using which I can find the endpoints and then draw the line. I tried various approaches from fitLine(), to findContour(), and other approaches of finding a fitting curve but none worked. I finally decided to filter the points by inspecting their endpoints and ensuring that they are in the right half and do not overlap. This helps to get a good set of lines that could then be averaged to obtain a slope and intercept, which I then used to compute lane endpoints and drew a single line. This time again, I assumed that the endpoints would be on the bottom line, and the midline of the image. All of the above filtering is what I inserted into the drawline function.

I still have few minor issues:
1. the left lane seems to be straying away in some images. I will need to inspect the points to be able to determine why that is the case. The right lane seems to coincide with the lane very nicely.
2. during the video, every now and then the lanes seem to "fly" away. Again, need to find-freeze such a frame and determine why this is occurring.

If you'd like to include images to show how the pipeline works, here is how to include an image: 

![Left lane straying while right looks good][./test_images_output/solidWhiteRight.jpg]


### 2. Identify potential shortcomings with your current pipeline


As stated above the pipeline still has glitches. The other issue is that it makes assumtions on the region of interest and the slope. Those may hold, however the biggest drawback is that we are drawing a line. As roads curve, so do the lanes, and then the lanes will be off by larger margins. This is evident towards the end of the 2nd "yellow" video.

The other major issue I see is the constants passed into the Canny algorithm, and the Hough transformation. These seem to hold for the images and videos supplied, all of which had pretty decent lighting. I am not sure if these will hold for different light settings, and when there is rain or snow, when the image would be filled with "short lines".

I can image the curves to be solved by breaking the image into multiple narrower bands and then running the algorithm on the narrower bands and then building a curve by stitching narrow straight lines. The issue of constants I am not sure how to resolve those - maybe a feedback mechanism of some sort??


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to - as stated above to break up the image into narrower rows and then processing them.

Another potential improvement could be to use a feedback loop to assess if the constants for Canny and Hough can be improved.

