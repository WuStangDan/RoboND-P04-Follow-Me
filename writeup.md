# RoboND-P04-Follow-Me
Udacity Robotics Nanodegree.


[//]: # (Image References)

[image0]: ./images/raw_points.png

### Network Architecture
A Fully Connected Network (FCN) is a network that can be used to run inference on every single pixel in an image. FCN's are built up of a set of convolution layers ending with a 1x1 convolution. This is the first half of the network called the encoder. The shape of the 1x1 convolution will be a small in height and width (pixels) but with lots of depth (filters). Note that just because the last part of the encoder is a 1x1 convolution, this doesn't mean that the output shape will be 1 pixel high and 1 pixel wide. The 1x1 convolution height and width will be the same as the layer preceding it (which could be 1x1 pixels). The reason for using a 1x1 convolution is that it acts just like a fully connected layer, but it retains the spatial information of all the pixels.

The second half of the network is the decoder. It is built up of layers of transposed convolutions whose outputs increase the height and width of the input while decreasing it's depth. Skip connections are also used at all points in the decoder which in this case concatenates the input to the encoder layer of the same size with the decoded layer.

For this specific problem I've chosen a model with 3 encoder layers, the 1x1 convolution, and then 3 decoder layers. I originally started with 2 encoder and 2 decoder layers but the I felt the 40x40 output size of the last layer was too large and wanted to bring it down. Using the an image with a height and width of 160 pixels, the output shape of each layer can be seen below.




### Hyper Parameters
The steps per epoch was always set to the total number of images divided by the batch size so that every epoch was approximately one run through all the training images. I also started with what seemed like a pretty low learning rate and rather large batch size of 0.005 and 32 respectively. However I found that even with these numbers training seemed to be happening too fast. As shown in the figure below, the loss would spike down quickly and then plateau. With this model I was able to achieve a final score IoU of 0.41 but I thought I could do better than that.

In one of the lecture videos it says that training too fast can cause the network to reach a local minimum and thats it's sometimes better to slow down training which can take longer but help break past the local minimum. Increasing the batch size to 64 and decreasing the learning rate to 0.001 made it so that the training actually could actually be seen when plotting the loss from epoch to epoch.

I ran these hyper parameters and evaluated to models IoU score every 5 epochs so I could monitor how the models performance was increasing over time. This also allowed me to monitor exactly when the model started to overfit the data and thus use a saved model that was produced before overfitting began.

### Results
The model info and final results for the selected model are shown below.



### Performance on Simulator

