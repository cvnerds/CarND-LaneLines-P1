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

My pipeline consisted of 5 steps. First, I converted the images to grayscale and apply a little blur. I don't think the blur is important, so I keep the kernel size to a minimum of 3. I assume the lanes are rather white, so I filter out dark colors (colors with a grayscale value less than 180). I anticipate this could become problematic at night time or with bad weather sequences. However, the test sequences are only at daytime. Before I apply canny edge detection I blank out all pixels outside a problem-specific ROI. The ROI is defined as a type of quad which is almost similar to a triangle in front of the car. This quad removes most of the image. This was very helpful because of the following:

Due to the fact that some lanes are dashed lines I needed to reduce the minimum length of a line. On the other hand, this gives a lot of false positives from random image content, but also from other neighboring lanes. Using a very narrow ROI filters out most part of the image.

In terms of parametrisation I kept canny thresholds at 50,150. I read somewhere the Canny paper recommends setting a ratio of 1:2 or 1:3. 50,150 had also been the values used in the quiz. For the Hough transform I tweaked some values. I wanted to have more precision with respect to the line angle, so I chose pi/720, which gives me a precision of 0.5°. I allow a large max line gap of 80 in hopes that dashed lines will get merged. A minimum length of 0 allows to get responses for very small dashed lines near the horizon. Clearly the use of a very restrictive ROI and color filter enables these assumptions here, because otherwise you will find a great many of tiny little edges elsewhere in the image.

I modified the draw_lines function as follows:

First I am assuming the lines can be separated into left and right. I estimate the slope and empirically have chosen 40° as a typical slope. I allow the slope to deviate +- 10°. So if a slope is close to +40° it will be assigned to the line segments of the right line. If the slope is -40° it will be assigned to the line segments of the left line. Now the question is how to merge these.

Initially I wanted to draw the line segments as a polyline. However, the painting then becomes complicated due to a need to maintain line segment order and continuity. Also there is no averaging.

So I chose a different approach. The idea is that each line segment can be a small piece of a virtual line which extends along the whole ROI. You could compute a center point in or the start/end points within this ROI. I chose to work with the origin of the line where y becomes 0. This makes the math a little bit simpler.

So for a given set of line segments I compute the average origin and the average slope.
This is now a robust estimate of the current line. This works for both left and right after separation.

This approach is already surprisingly robust in the optional scene. However, to improve robustness further I tried to filter the lines between two consecutive frames. I added a class called tracker. The tracker keeps the previous slope and origin. When a new estimate is added, the mean values are computed and used. Visually I don't see that this improves the difficult scene too much.

However, now that we have such a data structure we could improve upon this in the future with a Kalman filter. This would require to define a prediction mechanism, for example by using IMUs and estimating the egomotion of the vehicle.


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when lines don't appear very bright or white anymore. For example in bad weather conditions or at night time. 

Another shortcoming could be when the road slope changes a lot. For example in a very strong curve or when driving up and down a hill the ROI might become too restrictive or too large. So some sort of road surface and road surface height estimation procedure would be helpful.

When combining lines the assumption is that similar slope means that the lines belong together. This will become a bit tricky if, for example, we had a situation where a lane is drawn with two lines. The new lane would then be estimated as a line in between those two. That might be still acceptable, but isn't quite right. This problem becomes even worse if other symbols are painted onto the ground. Any rectangle with sides parallel to the lanes would then also be used.




### 3. Suggest possible improvements to your pipeline

A possible improvement would be to use some sort of adaptive thresholding to be more robust to different lighting conditions. We could try to analyse the color histogram of the image to identify day / night scenario and adjust thresholds accordingly.

As mentioned before it would be helpul to try to segment the road surface to adjust the ROI. We anticipate problems in curves. By identifying the how the line slope changes we could adjust the ROI accordingly.

To be more robust towards outliers we could use median values instead of mean values. To improve upon filtering the lines between consecutive frames a Kalman filter could be used.
