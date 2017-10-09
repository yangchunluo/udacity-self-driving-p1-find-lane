# **Finding Lane Lines on the Road** 

Yangchun Luo
Oct 8, 2017

---
The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

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

### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when ... 

Another shortcoming could be ...


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to ...

Another potential improvement could be to ...
