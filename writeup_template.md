**Behavioral Cloning Project**

The goals / steps of this project are the following:
* Use the simulator to collect data of good driving behavior
* Build, a convolution neural network in Keras that predicts steering angles from images
* Train and validate the model with a training and validation set
* Test that the model successfully drives around track one without leaving the road
* Summarize the results with a written report


[//]: # (Image References)

[image1]: ./examples/input_data_set.png "Input data"
[image2]: ./examples/flattening_data_set.png "flattening"
[image3]: ./examples/picture1.jpg "Random1"
[image4]: ./examples/picture2.jpg "Random2"
[image5]: ./examples/picture3.jpg "Random3"

## Rubric Points
###Here I will consider the [rubric points](https://review.udacity.com/#!/rubrics/432/view) individually and describe how I addressed each point in my implementation.  

---
###Files Submitted & Code Quality

####1. Submission includes all required files and can be used to run the simulator in autonomous mode

My project includes the following files:
* model.py containing the script to create and train the model
* drive.py for driving the car in autonomous mode
* model.h5 containing a trained convolution neural network 
* writeup_report.md or writeup_report.pdf summarizing the results

####2. Submission includes functional code
Using the Udacity provided simulator and my drive.py file, the car can be driven autonomously around the track by executing 
```sh
python drive.py model.json
```

####3. Submission code is usable and readable

The model.py file contains the code for training and saving the convolution neural network. The file shows the pipeline I used for training and validating the model, and it contains comments to explain how the code works.

###Model Architecture and Training Strategy

####1. An appropriate model architecture has been employed

Instead of using [nVidia model](https://images.nvidia.com/content/tegra/automotive/images/2016/solutions/pdf/end-to-end-dl-using-px.pdf)
as Udacity suggested, I chosed to try another business model from [comma.ai model](https://github.com/commaai/research/blob/master/train_steering_model.py))

My final model consisted of the following layers:

| Layer         		|     Description	        					| 
|:---------------------:|:---------------------------------------------:| 
| Input         		| 90x320x3 cropped image, normalized to (-1, 1) 							| 
| Convolution 8x8     	| 4x4 stride, same padding, outputs 23x80x16 	|
| ELU					|												|
| Convolution 5x5	    | 2x2 stride, same padding, outputs 12x40x32 	|
| ELU					|	
| Convolution 5x5	    | 2x2 stride, same padding, outputs 6x20x64 	|
| Flatten layers (6x20x64 -> 7680) |
| Dropout 0.2 |
| ELU					|	
| Fully connected		| 7680 -> 512 |
| Dropout 0.5 |
| ELU					|	
| Fully connected		| 512 -> 1 |


####2. Attempts to reduce overfitting in the model

The model contains two dropout layers in order to reduce overfitting. 

####3. Model parameter tuning

The Adam optimizer was chosen with default parameters, so the learning rate was not tuned manually.
and the chosen loss function was mean squared error (MSE).
(as described in method "get_model")

####4. Appropriate training data

Training data was chosen to keep the vehicle driving on the road. 
For collecting enough data and training well, I recorded:
* 3 laps of center lane driving
* 2 laps of recovery driving from the sides
* 2 laps of center lane driving in Counter-Clockwise 

Recovery data is recorded only when the car is driving from the side of the road back toward the center line.
So when the car is not driving perfectly, this will give it a chance to adjust back to the center instead of continuing geting off to the side of the road.
(At first, I thought the road on the bridge is the most easy part for driving, so I missed the recovery data on that.
When the car wander off to the side on the bridge, it can not move back to the center, then it hit the side and stopped moving on...
So do not think AI goes like you think, just give it enough data...)

Recording on driving in counter-clockwise is resistant for left turn bias.

####5. Data preprocessing and augmentation
* crop image by removing top and bottom parts of useless information.
* use side camera images, create adjusted steering measurements (+0.25 for the left, -0.25 for the right).
* flip images, along with the steer angle, if magnitude is > 0.33
* random brightness adjustments, artificial shadows, and horizon shifts on images. (refer to [Jeremy Shannon](https://medium.com/udacity/udacity-self-driving-car-nanodegree-project-3-behavioral-cloning-446461b7c7f9))

Here is an example image of random distort:
![alt text][image3]
![alt text][image4]
![alt text][image5]

####6. Data Distribution Flattening
The test track includes long sections with very slight or no curvature which would cause a biase for straight driving.
So we should flat the data on different steer angles.
Firstly, we category the data by angles. 
Then for the categories with less numbers than the average, we keep them all to training data.
But for the categories with larger count, we discard partly on them.
(More details described in model.py at line 216 - 233).

The distribution of the input data can be observed below,
![alt text][image1]
And the resulting data distribution can be seen in the chart below.
![alt text][image2]
