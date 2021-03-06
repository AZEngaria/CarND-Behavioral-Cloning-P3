## Writeup
---

**Behavioral Cloning Project**

The goals / steps of this project are the following:
* Use the simulator to collect data of good driving behavior
* Build, a convolution neural network in Keras that predicts steering angles from images
* Train and validate the model with a training and validation set
* Test that the model successfully drives around track one without leaving the road
* Summarize the results with a written report


[//]: # (Image References)

[image1]: ./images/cnn-architecture-624x890.png "Model Visualization"
[image2]: ./images/center.jpg "Center Image"
[image3]: ./images/recovery1.PNG "Recovery Image"
[image4]: ./images/recovery2.PNG "Recovery Image"
[image5]: ./images/recovery3.PNG "Recovery Image"
[image6]: ./images/UnFlipped.jpg "Normal Image"
[image7]: ./images/Flipped.jpg "Flipped Image"

## Rubric Points
### Here I will consider the [rubric points](https://review.udacity.com/#!/rubrics/432/view) individually and describe how I addressed each point in my implementation.  

---
### Files Submitted & Code Quality

#### 1. Submission includes all required files and can be used to run the simulator in autonomous mode

My project includes the following files:
* model.py containing the script to create and train the model
* drive.py for driving the car in autonomous mode
* model.h5 containing a trained convolution neural network 
* writeup_report.md or writeup_report.pdf summarizing the results

#### 2. Submission includes functional code
Using the Udacity provided simulator and my drive.py file, the car can be driven autonomously around the track by executing 
```sh
python drive.py model.h5
```

#### 3. Submission code is usable and readable

The model.py file contains the code for training and saving the convolution neural network. The file shows the pipeline I used for training and validating the model, and it contains comments to explain how the code works.

### Model Architecture and Training Strategy

#### 1. An appropriate model architecture has been employed

My model consists of a convolution neural network with 5x5 and 3x3 filter sizes and depths between 32 and 64 (model.py lines 86-108) 

The model includes RELU layers to introduce nonlinearity (code lines 104, 108) and the data is normalized in the model using a Keras lambda layer (code line 89). 

#### 2. Attempts to reduce overfitting in the model

The model contains dropout layers in order to reduce overfitting (model.py lines 114). 

The model was trained and validated on different data sets to ensure that the model was not overfitting (code line 10-16). The model was tested by running it through the simulator and ensuring that the vehicle could stay on the track.

#### 3. Model parameter tuning

The model used an adam optimizer, so the learning rate was not tuned manually (model.py line 132).

#### 4. Appropriate training data

Training data was chosen to keep the vehicle driving on the road. I used a combination of center lane driving, recovering from the left and right sides of the road. 

For details about how I created the training data, see the next section. 

### Model Architecture and Training Strategy

#### 1. Solution Design Approach

The overall strategy for deriving a model architecture was to predict the appropriate measurement for steering angle as the simulator runs uses the model.

My first step was to use a convolution neural network model similar to the one developed by NVIDIA for End to End Learning for Self-Driving Cars. I thought this model might be appropriate because it was designed particularly for the purpose of mapping raw pixels from a front-facing cameras directly to steering commands.

In order to gauge how well the model was working, I split my image and steering angle data into a training and validation set. I found that my first model had a low mean squared error on the training set but a high mean squared error on the validation set. This implied that the model was overfitting. 

To combat the overfitting, I modified the model by adding a Dropout (of value 30-40%) after the Fully Connected layers which contained high number of parameters (>200K). 

After the model was laid, it was time to test it on the simulator to see how well the car was driving around track one.

There were a few spots where the vehicle did not run as it smoothly/accurately as expected. Instead of driving in a straight line, it went from left to right (and vice-versa) many a times trying to recover and finally at one spot it fell off the track.

Inorder to improve the driving behavior in these cases, I examined the training loss and validation loss for high number of epochs. As the model is trained, the losses tend to fluctuate. Initially it started with high loss but it managed to decrease till a particular minima. On observing the behaviour of the losses, there was a point where after reaching a minimum value, the loss again increased. This helped me chose a the value of epoch for the model. 

Another thing, the incorrect placing of dropouts was also a cause of inaccurate behaviour of the car. Therefore, choosing the dropouts value to 0.3 or 0.4 and placing it after the Fully connected layers helped solve the issue.

At the end of the process, the vehicle is able to drive autonomously around the track without leaving the road.

#### 2. Final Model Architecture

The final model architecture (model.py lines 18-24) consisted of a convolution neural network with the following layers and layer sizes:

| Layer (type)        | Description                                     | 
| ------------------- |:-----------------------------------------------:| 
| Lamda Layer         | Normalization - Input 160x320x3                 | 
| Cropping2D          | Input 160x320x3, Ouput 65x320x3                 |  
| Convolution Layer 1 | 3@65x320, Kernel Size - 5x5, Stride 2x2         |  
| Convolution Layer 2 | 3@31x158, Kernel Size - 5x5, Stride 2x2         |  
| Convolution Layer 3 | 24@14x77, Kernel Size - 5x5  Stride 2x2         |  
| Convolution Layer 4 | 36@5x37,  Kernel Size - 3x3                     | 
| Activation          | Relu                                            | 
| Convolution Layer 5 | 48@3x35,  Kernel Size - 3x3                     |
| Activation          | Relu                                            | 
| Flatten             | 64x1x33, Ouput = 2112                           |
| Fully Conntected 1  | 1164 Neurons                                    |
| Dropout             | 30%                                             |
| Fully Conntected 2  | 100 Neurons                                     |
| Dropout             | 40%                                             |
| Fully Conntected 3  | 50 Neurons                                      |
| Fully Conntected 4  | 10 Neurons                                      |
| Activation          | Relu                                            | 
| Fully Conntected 5  | 1 Neuron                                        |

Total params: 2,648,603
Trainable params: 2,648,603

#### Nvidia Architecture
![alt text][image1]

#### 3. Creation of the Training Set & Training Process

I used the data given by Udacity to train the model. Here is an example image of center lane driving:

![alt text][image2]


As the sample data was used for this, the model also easily recovers and centers the position in lane as and when deviated. The following images show the recovery of the car when it slightly moves towards the right:

![alt text][image3]
![alt text][image4]
![alt text][image5]

To augment the data sat, I also flipped images and angles as this would increase the training data and additionally be helpful to train the model for the right turns corresponding to each left turn and vice versa. Subsequently the steering measurements were also negated. For example, here is an image that has then been flipped:

![alt text][image6]

![alt text][image7]

After the collection process, I had 35133 number of data points. I then preprocessed this data by normalizing it using the formula 
(X-255.0) / 0.5 to center the image values and have a similar distribution for data.

I finally randomly shuffled the data set and put 20% of the data into a validation set. 

I used this training data for training the model. The validation set helped determine if the model was over or under fitting. The ideal number of epochs was 3 as evidenced by the loss value which starts to increase after 3 iterations (Epochs) of training. I used an adam optimizer so that manually training the learning rate wasn't necessary.
