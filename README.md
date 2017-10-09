# **Finding Lane Lines on the Road** 

Yangchun Luo
Oct 8, 2017

This is the assignment for Udacity's Self-Driving Car Term 1 Project 1.

This replaces the original [README.md](https://github.com/yangchunluo/udacity-self-driving-p1-find-lane/blob/master/README-orig.md)

---
The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report
[outlier-image] 'examples/outlier-and-regression.jpg'
---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My image processing pipeline has the following parts:
1. Convert the raw image to grayscale.
2. Apply Gaussian blur with kernel size 3.
3. Use Canny algorithm to detect edges, with the low and high threshold 50 and 150.
4. Mask the image with a region of interest, given that the camera is amount in a fixed position so the lane marking should be in a stable region in a recording.
5. Find lines in above region using Hough transformation. The parameters are borrowed from previous quizes.

In order to draw a single line on the left and right lanes, I made the following modification:
1. Instead of directly drawing lines in the hough transformation function, I separate draw_lines() from hough_lines().
2. hough_lines() returns a list of detected lines, which feeds into a new function extrapolate_lines().
3. extrapolate_lines() does two main things:  clustering the lines into two clusters and for each cluster, get the fitted line parameters (slope, intercept).
4. The output of extrapolate_lines (two: left and right) is used to plot given the width of the image. It is later cropped/masked by the previously defined region of interest. In this way, we can extend the lines in the region only.

Here I will explain the two main components in extrapolate_lines().

**Clustering**: I first tried KMeans to make two clusters. For each line (x1,y1,x2,y2), I generate slope and intercept. I added 1e-6 to the denominator for numerical stability. I used slope only to do clustering. This, however, can suffer from mis-classification. I've seen cases where certain parts of the left lane is clustered to be part of the right lane, due to the previous step finding a line with a very different slope in left lane.

This mis-classification is quite costly when later fitting the line. I changed the clustering to simply use slope > 0 and slope < 0, given the unique shape in this project. This may not generate to other video setup, but simply and worked okay for this project.

**Fitting**: After clustering, I put all the vertices in x, y arrays and use linear regression to fit a line. As mentioned above, this turned out to be quite sensitive to outliers. One source of outliers came from mis-classification. Another source came from "noise" line detected by previous steps. 

I employed per sample (vertice) weight to migitate the problem. Each sample (x,y) pair's weight is determined by the following way:
- it is proportional to the length of its line. That is, longer lines has higher weights.
- it is inversely proportional to its Eclidean distance from the average of the Hough space of (slope and intercept). That is, if a pair is very different from the mass of the cluster, it will receive little weight.
- if the slope is outside an *empirical* range, the weight is set to 0. This may not generate well in other video setups.
- if the x coordinate is 1.5 stdDev away from the cluster average, the weight is set to 0. This is based on the observation that the two lines can be easily separated by x coordinate.

The last two points are to deal with outliers. In this following example, the regressed line for left lane is off due to part of the right lane being mis-classified.

[outlier-image]

### 2. Identify potential shortcomings with your current pipeline

The approached used for clustering and line fitting has a lot of assumptions about this particular setup baked in. The method does not generate well in other situations, for example, when camera is mounted in a different location. Also, the statically defined region of interest can suffer from the same issue.

While the etrapolation code works okay in most cases, it does not work well in the challenge where the lane region contains shade.


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to find the region of interest using CNN-based approach through training. But this is a topic beyond the scope of this project.

To represent a line, I used (slope, intercept) and added some numerical stability. But both may suffer from high variance: a tiny little change in slope and result in a large change in intercept, given a point. A better may be to use (rho, theta) based representation. This may improve the KMeans cluster result, which I did not have time to explore.

