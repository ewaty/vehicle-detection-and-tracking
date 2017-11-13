**Vehicle Detection Project**

![](giphy.gif)

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/car_notcar.png
[image2]: ./output_images/hog_features.png
[image3]: ./output_images/windows1.png
[image4]: ./output_images/windows2.png
[image5]: ./output_images/windows3.png
[image6]: ./output_images/windows4.png
[image7]: ./output_images/windows5.png
[image8]: ./output_images/windows6.png
[image9]: ./output_images/results1.png
[image10]: ./output_images/results2.png
[image11]: ./output_images/heat1.png
[image12]: ./output_images/heat2.png
[image13]: ./output_images/heat3.png
[image14]: ./output_images/heatmaps1.png
[image15]: ./output_images/heatmaps2.png
[image16]: ./output_images/heatmaps.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the 4th code cell of the IPython notebook.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters, checking the final test accuracy of the classifier for each set. Ultimately I settled on HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using previously mentioned parameters (`orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`). To create the features I used all the channel of the image converted to YCrCb color space. The classifier can be seen in code cell 8 of IPython notebook. The testing accuracy of the classifier was 0.9806.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to search for cars in the lower part of the image starting at y = 360 untill the bottom of the image. 

My implementation of sliding window search can be found in cell 7 of IPython notebook. The function calculates HOG features for the entire image and then subsamples the parts relevant to the given window. 

I decided to use windows of size 64x64 and 96x96. To get a feature vector of equal length `pix_per_cell` variable is scaled by a factor of 1.5 in case of 96x96 window. Image below shows results of applying sliding window search on test images for windows of scale 1 and 1.5, as well as combined images:

![alt text][image3]
![alt text][image4]
![alt text][image5]
![alt text][image6]
![alt text][image7]
![alt text][image8]

Windows overlap by 0.8. Through multiple trials I determined this was the lowest overlap that provided satisfactory visual accuracy.

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image9]
![alt text][image10]

To optimize the performance of the classifier I searched for cars only in the lower part of the image. I also extracted HOG features for the entire image before applying SVM classifier on each window.

---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_out.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

My algorithm memorizes the detected windows for the past 25 frames. After 25 iterations, it starts to create a heat map, where each pixel intensity represents the number of windows in which the pixel was included. Then values of pixels with values less than 20 are made 0 to reject false positives. New bounding boxes for the image under consideration are centered around heat map areas that passed the thresholding.

### Here are six frames and their corresponding heatmaps before and after thresholding:

![alt text][image14]
![alt text][image15]


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Although the pipeline detects cars pretty well, sometimes it fails to detect one and sometimes it sees two cars where there is one. For the first problem, a solution might be to use an object tracking function. It could be of help when a car is first detected, but then "disappears". For the second problem, the algorithm could see if there are neighboring windows, which have similar color histograms. That could indicate that both windows are in fact containing parts of the same car.
