# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps, the steps are introduced as following:

* Convert the image to grayscale
* Use Guassian kernel to smooth the image, kernel size [3x3]
* Extract edges by Canny method, the `low_threshod` and `high_threshold` are recommended to set with a ratio of 1:2 or 1:3. Here, in my experiment, I set them 50 and 130, respecively.
* Since the lanes are usually in a certain area of the whole image, I use quadrilateral to mask the area out. I set the coordinates of them as [(150,img.shape[0]),(435,330),(530,330),(img.shape[1],img.shape[0])]
* The last thing is to use Hough transform to detect lines given the Canny edges. According to the experience of the quiz, I did not tune the parameters to much. I fix `rho=1`, `theta=np.pi/180`, `threshold=10`, `min_line_len=20`, `max_line_gap=10`.

Based on the above pipeline, I first test it on images. Here is one example of the original image and the output of the detected lanes.

![alt text][./test_images/solidWhiteCurve.jpg]
*original image*

![alt text][./test_images_output/solidWhiteCurve_lane.png]
*Output of the detected lanes*

The final goal of the project is to draw single lines of left and right lanes. In order to do so, the basic idea utilized here is to find the parameterized description of lane lines and use coordinates of bottom and top points to draw them. 

In order to do so, I first calculate the slopes of all the detected line segments. If it is below, I regard it as part of the left line, and it is considered as part of right line, if it is above zero. Then, by averaging them, I can obtain the slopes of left and right lanes. Besides the slope, one point on the line should be necessary to final localize the line. I use the averaged coordinate of all the left and right line segments to be the point. Given the point and the slope, the lane lines can be finally determined. 

However, when I test the method on the video, the detected lines on several frames somewhow 'jump'. I think it is due to the reason that the slope cannot be robustly estimated by averaging. If there are some outlier line segments, slope calculation can be errorneous by taking those into account. So I decide to use [RANSAC](https://scikit-learn.org/stable/auto_examples/linear_model/plot_ransac.html#sphx-glr-auto-examples-linear-model-plot-ransac-py) to robustly estimate the lines. The slopes of lines seem to be more stable than the above method.  

Here is an illusration of the final result of lane line detection.

![alt text][./test_images_output/solidWhiteCurve_lane_new_draw.png]
*Output of the final detected lanes*


### 2. Identify potential shortcomings with your current pipeline


One shortcoming is that the designed pipeline needs too many manually tuned parameters, especially in the steps of **region mask determination**, **Canny edge extraction** and **Hough transform**. It can severely influence the generalization of the proposed approach on other videos and it needs a lot of work to search the optimal parameter in a several dimensional space. 



### 3. Suggest possible improvements to your pipeline

Advanced edge detection robust to the shadowing of trees along the sides of the roads. 

If the roads are curly, hough transform may not work very well. So find other curve detection methods are also possible.

To utilize time serious information for tracking the line segment position by e.g. Kalman filter. 
