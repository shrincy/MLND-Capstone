# MLND-Capstone
My capstone project for Udacity's Machine Learning Nanodegree

#### This project is in progress. I will be updating this repository with steps I have taken so far as well as current status and future updates needed. The project is now in the final stages and I hope to have it completed shortly.

Please see my original capstone proposal [here](https://github.com/mvirgo/MLND-Capstone-Proposal).

See an early version of the model detecting lane lines with perspective transformed images [here.](https://youtu.be/ZZAgcSqAU0I)
An early version of my model trained *without* perspective transformed images, i.e. regular road images, can be seen [here!](https://www.youtube.com/watch?v=Vq0vlKdyXnI)

Lastly, with the finalized fully convolutional model, there are a couple additional videos I made. The first, which is the same video from the above two, has between 10-20% of the frames fed into the mode, as can be seen [here.](https://youtu.be/bTMwF1UoZ68) Additionally, a video made from the Challenge Video from Udacity's Advanced Lane Lines project in the SDCND, where the neural network had **never** seen the video before, can be seen [here.](https://youtu.be/_qwET69bYa8) The model performs fairly robustly on the never-before-seen video, with the only hitch due to the large light difference as it goes under the overpass.

An additional video can be seen at [this Dropbox link.](https://www.dropbox.com/s/18jia2x9pg42s4n/proj_reg_vid.mp4?dl=0)

You can download the full training set of images I used [here](https://www.dropbox.com/s/rrh8lrdclzlnxzv/full_CNN_train.p?dl=0) (**NOTE:** this is 468 MB!) and the full set of 'labels' (which are just the 'G' channel from an RGB image of a re-drawn lane with an extra dimension added to make use in Keras easier) [here](https://www.dropbox.com/s/ak850zqqfy6ily0/full_CNN_labels.p?dl=0) (157 MB). 

## Completed Steps
* Obtaining driving video
* Extracting images from video frames (see `load_videos.py`)
* Manual processing of images to remove unclear / blurry images
* Obtaining camera calibration for the camera used to obtain driving video (see `cam_calib.py`)
* Load in all road images (accounting for time series), and save to a pickle file (see `load_road_images.py`)
* Undistort images (using the camera calibration) and perspective transform to top-down images, save the transformed images (see `undistort_and_transform.py`)
* Created a file (based on my previous Advanced Lane Lines model, see `make_labels.py`) to calculate the lines, and to save the line data (which will be the labels for each image) to a pickle file. This file needs the lines re-drawn over the perspective transformed images in red to work appropriately.
* Built a neural network that can take perspect transformed images and labels, then train to predict labels on new perspective transformed images (see `perspect_NN.py` for current version).
* Created a file (see `check_labels.py`) in which I save down each image after labelling, in order to check whether the labels appear correct given the re-drawn lane lines from a computer vision-based model. This will help make sure I'm feeding good labels to my neural network for training. This file was later updated to output the actual lane "drawing" for those I found to be acceptable labels, so that I could use that actual image for training.
* Manually re-drew lane lines for detection in the `make_labels.py` file. Doing so involved re-drawing in red over the lane lines to aid the computer vision-based model to calculate the line data (especially helpful where the line is fairly unclear in the video image). I originally obtained nearly 700 seconds of video (over 11 minutes), which was over 21,000 frames. After manually getting rid of blurry images and others (such as those with little to no visible lines within the area for which the perspective transformation would occur), I had over 14,000 images. In order to account for similar images within small spans of time, I am currently using only 1 in 10 images, or 3 frames out of each second of video. As such, just over 1,400 images were used.
* Improved the original neural network substantially by augmenting the road images and labels prior to feeding the network. See within `combine_and_augment.py`.
* Improved the lane detection for very curved lines by changing `make_labels.py` to end the detection of a line once it hits the side of the image (the previous version would subsequently only search vertically further, messing up the detection by often crossing the other lane line).
* Further improved `make_labels.py` to look at two rotations of the image as well, and taking the average histgram of the three images. This helps with a lot of curves or certain perspective transforms where the road lines are not fairly vertical in the image, as the histogram is looking specifically for vertical lines. The big trade-off here is that the file is much slower (around 1 minute previously to almost 15 minutes now). I'll add this as a potential improvement to try other methods of this to re-gain speed; however, given that this is done outside of the true training or usage of the final model it is not a high priority item.
* Made the `lane_lines.py` file to take in the trained neural network model for perspective transformed images, predict lines, and draw the lines back onto the original image. 
* Created and trained a neural network (see `road_NN.py`) capable of detecting lane lines on road images without perspective transformation. 
* Using keras-vis (see documentation [here](https://raghakot.github.io/keras-vis/), created activation heatmaps by layer in order to see whether the model was looking in the correct place for the lines. See `layer_visualize.ipynb` for more.
* Created a fully convolutional neural network, whereby a regular road image is fed into the network and the expected lane to be drawn (as opposed to just the specific lane coefficients) is returned. See `fully_conv_NN.py`.
* Made the file to re-draw the lanes based off the model's predictions. See `draw_detected_lanes.py`.
* Compared the performance of my model to the previous CV-based one. The CNN is much more robust, and can work in a wider variety of situations - the Challenge video essentially failed completely with my CV-based model.
* Compared the speed of the CNN vs. the CV model. The CNN model is nearly real-time, generating between 25-29 frames each second for a 30 fps video. This is significantly faster than the CV model, which only managed 4.5 fps generated. Note that this speed increase is dependent on GPU acceleration - without GPU, the speed is roughly 5.5 fps generated, which is still an improvement, but by much less.

## Current Status
I am finally getting to the wrap-up stage! I have now created a robust fully convolutional neural network which can detect lane lines in videos it has never seen before, and is faster than my previous pure computer vision-based model. I am working on cleaning up some files as well as on preparing the final project write-up.

## Image statistics
* 21,054 total images gathered from 12 videos (a mix of different times of day, weather, traffic, and road curvatures)
* 17.4% were clear night driving, 16.4% were rainy morning driving, and 66.2% were cloudy afternoon driving
* 26.5% were straight or mostly straight roads, 30.2% were a mix or moderate curves, and 43.3% were very curvy roads
* The roads also contain difficult areas such as construction and intersections
* 14,235 of the total that were usable of those gathered (mainly due to blurriness, hidden lines, etc.)
* 1,420 total images originally extracted from those to account for time series (1 in every 10)
* 227 of the 1,420 unusable due to the limits of the CV-based model used to label (down from 446 due to various improvements made to the original model) for a total of 1,193 images
* Another 568 images (of 1,636 pulled in) gathered from more curvy lines to assist in gaining a wider distribution of labels (1 in every 5 from the more curved-lane videos; from 8,187 frames)
* In total, 1,761 original images
* I pulled in the easier project video from Udacity's Advanced Lane Lines project (to help the model learn an additional camera's distortion) - of 1,252 frames, I used 1 in 5 for 250 total, 217 of which were usable for training
* A total of 1,978 actual images used between my collections and the one Udacity video
* After checking histograms for each coefficient of each label for distribution, I created an additional 4,404 images using small rotations of the images outside the very center of the original distribution of images. This was done in three rounds of slowly moving outward from the center of the data (so those further out from the center of the distribution were done multiple times). 6,382 images existed at this point.
* Finally, I added horizontal flips of each and every road image and its corresponding label, which doubled the total images. All in all, there were a total of 12,764 images for training.

## Issues / Challenges so far
#### General
* File ordering - using `glob.glob` does not pull in images in a natural counting fashion. I needed to add in an additional function (see Lines 13-16 in `load_road_images.py`) to get it to pull the images in normally. This is crucial to make sure the labelling is matched up with the same image later on so that it is easier for me to know which image is causing issues (especially in the `make_labels.py` file, which fails if it cannot detect the line).

#### Model
* The initial model I chose had issues due to perspective transformation still being used to re-draw the lines - if the horizon line changed, or if a different camera was used (needing a different transformation), the lines would look off even if the shape was fairly correct.
* I tried to fix the above issue by using keras-vis activation heatmaps, but found these were too slow (especially given different images had heatmaps with differing issues - some learned the label off of only one of the lines [curves], or learned from the road space instead of the line specifically [straights]), or could not generalize well enough for which layer heatmap to use.
* I also used transfer learning for the above issue from my Behavioral Cloning project, but the model paid more attention to the entire road surface as opposed to the lane lines themselves.

#### Images
The below issues often caused me to have to throw out the image:
* Image blurriness - although road bumpiness is the main driver of this, it is pronounced in raining or nighttime conditions. The camera may focus on the rain on the windshield or on reflections from within the car.
* Line "jumping" - driving on bumpy roads at highway speeds tends to cause the lane lines to "jump" for an image or two. I deleted many of these although tried to keep some of the better ones to help make a more robust model.
* Dirt or leaves blocking lines
* Lines blocked by a car
* Intersections (i.e. no lane markings) and the openings for left turn lanes
* Extreme curves - lane line may be off to the side of the image
* Time series - especially when going slower, frame to frame images have little change and could allow the final model to "peek" into the validation data
* Lines not extending far enough down - although the lane lines may be visible in the regular image, they may disappear in the perspective-transformed image, making it impossible to label
* Given that I am manually drawing lines in red (for putting through the CV-based model for labelling of line data purposes), tail lights at night could potentially add unwanted data points in the thresholded images. Additionally, blue LED lights at night could also cause problems if I were to draw lines in blue. I have not as of yet looked at how much green is in each image, but assume grass or leaves could also cause issues.
* Certain images failed with the histogram and had to be slightly re-drawn, in a select few cases meaning the drawn line needed to be extended further than the original image showed. Isolating those with issues was made easier by including a counter in the middle of the file to make labels (not in the finished product) which identified which image failed the histogram test
* The CV-based model can still fail at creating good labels, leading to large differences that have an out-sized effect on the neural network training (especially when using mean squared error compared to other loss types). Prior to finalizing the model I will use the `check_labels.py` file to go back and either fix the images or remove them so that training can be improved.
* The CV-based model is bad with curved lines as any that fall off the side of the image cause the sliding windows to behave incorrectly. The sliding windows will begin only searching vertically in the image, and often will cross the other line than the original line detected, causing the polyfit to be way off. I updated `make_labels.py` to end the sliding windows at the side of the image to account for this.
* The CV-based model I am using for initial labeling struggles when lines start under the car at some other angle than vertical - such as often happens with big curves. This leads the model to not start the detection until mid-way up the line, wherein in then tends to think the direction of the line is completely different than actual. Both the CV-based model and my images need to be fiddled with to improve this issue.

## Upcoming
* Complete final project write-up

#### Minor potential improvements
* The function `natural_key` is currently contained in both `load_road_images.py` and `make_labels.py`. This should be consolidated down (probably in a separate file; may also consolidate other helper functions within there).
* The `make_labels.py` file is now a lot slower as I added some image rotations to assist with the histograms used in initial detection of the lines in the computer vision-based model - it looks for vertical lines. These rotations have significantly slowed down the file.
