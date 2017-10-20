# RoboND-P04-Follow-Me
##Udacity Robotics Nanodegree.


[//]: # (Image References)
[title_image]: ./images/model-performance-4.png
[image0]: ./images/lr01.png
[image1]: ./images/lr001.png
[image2]: ./images/lr0001.png
[image3]: ./images/lr01-100.png

[image5]: ./images/sim.png
[image6]: ./images/model-performance-1.png
[image7]: ./images/model-performance-2.png
[image8]: ./images/model-performance-3.png

![Model Performance][title_image]
Left is image, Middle is Ground Truth, Right is Model


### Network Architecture
A Fully Connected Network (FCN) is a network that can be used to run inference on every single pixel in an image. FCN's are built up of a set of convolution layers ending with a 1x1 convolution. This is the first half of the network called the encoder. The shape of the 1x1 convolution will be a small in height and width (pixels) but with lots of depth (filters). Note that just because the last part of the encoder is a 1x1 convolution, this doesn't mean that the output shape will be 1 pixel high and 1 pixel wide. The 1x1 convolution height and width will be the same as the layer preceding it (which could be 1x1 pixels). The reason for using a 1x1 convolution is that it acts just like a fully connected layer, but it retains the spatial information of all the pixels.

The second half of the network is the decoder. It is built up of layers of transposed convolutions whose outputs increase the height and width of the input while decreasing it's depth. Skip connections are also used at all points in the decoder which in this case concatenates the input to the encoder layer of the same size with the decoded layer.

For this specific problem I've chosen a model with 3 encoder layers, the 1x1 convolution, and then 3 decoder layers. I originally started with 2 encoder and 2 decoder layers but the I felt the 40x40 output size of the last layer was too large and wanted to bring it down. Using the an image with a height and width of 160 pixels, the output shape of each layer can be seen below. Skip connections are indicated by connecting lines.

	Features Layer (?, 160, 160, 3) - - |
	Encoder 1 (?, 80, 80, 32) - - - -|  |
	Encoder 2 (?, 40, 40, 64)  - -|  |  |
	Encoder 3 (?, 20, 20, 128)    |  |  |  Skip
	1x1 Conv (?, 20, 20, 128)     |  |  |  Connections
	Decoder 1 (?, 40, 40, 128) - -|  |  |
	Decoder 2 (?, 80, 80, 64) - - - -|  |
	Decoder 3 (?, 80, 80, 64) - - - - - |
	Output Layer (?, 160, 160, 3)


### Hyper Parameters
The steps per epoch was always set to the total number of images in the training set divided by the batch size so that every epoch was approximately one run through all the training images. I used a batch size of 32, which is quite large, as I wanted relatively stable training. Given the type of information the model is working with I felt a small batch size would produce large inconsistenancies when training from batch to batch. If training didn't go well at these learning rates I could always tune batch size further. A batch size of 32, compared to 16 or 8, would mean that more epochs would be required to do the same amount of training (since number of steps per epoch decreases with increasing batch size), but that was fine since I was using my own GPU to train and had plenty of time.

I then tested 3 different learning rates, 0.01, 0.001, and 0.0001 and ran them all for 40 epochs. All other hyper parameters were identical. Before I looked at the results I was aware that if I saw a decrease in performance for the decreasing learning rate, that didn't necessarily mean that the larger learning rates were better. It could just mean that 40 epochs wasn't long enough training.

The results of the learning rate testing are shown below:

| LR  | Loss | Val Loss  | Score |
| ------------- | ------------- | ----- | ---- |
| 0.01| 0.0146 | 0.0308 | 0.3567 |
| 0.001 | 0.0159 | 0.0265 | 0.414 |
| 0.0001 | 0.0395 | 0.0372 | 0.2782 |

![LR 0.01][image0]
LR 0.01

![LR 0.001][image1]
LR 0.001

![LR 0.0001][image2]
LR 0.0001

Based on these results it could be concluded that 0.001 is the best learning rate based on it's validation score and IoU score. However looking at the graph above it seems that even after 40 epochs the loss or val loss didn't settle for any of the learning rates, as seen by the 0.01 figure. The other too figures are too zoomed out to see if change is still happening but based on the model printouts they were still changing.

It should also be noted that the IoU Score is very sensitive. For the 0.01 model the epochs 36 to 39 had the following val loss: 0.0265, 0.0349, 0.0344, 0.0198. This would indicate that epoch 40 by chance happened to be a bad one since training wasn't stable yet and the sensitive IoU score just happened to drop. Note that the loss fucntion is lower than 0.001's which could indicate over fitting or what is more plausible that it's farther along in training.

Based on these results I decided to run a learning rate of 0.01 for 100 epochs. Then if training had stabilized, I would try the lower learning rate of 0.001 and see if model fit could be improved further. The results are shown below.

![LR 0.01 - 100][image3]

Because of the large spike it's hard to see but the loss and val loss decrease until about epoch 80 after which they begin to settle.

After this, the learning rate was dropped to 0.001 and the model was trained for 15 epochs. The last 5 epochs were trained one at a time so the score could be evaluated to ensure the model had stabilized. This information is shown below.

	Epoch 100 - Score: 0.4070 loss: 0.0109 - val_loss: 0.0184
	Epoch 110 - Score: 0.4266 loss: 0.0092 - val_loss: 0.0280
	Epoch 111 - Score: 0.4263 loss: 0.0092 - val_loss: 0.0268
	Epoch 112 - Score: 0.4230 loss: 0.0092 - val_loss: 0.0289
	Epoch 113 - Score: 0.4301 loss: 0.0092 - val_loss: 0.0273
	Epoch 114 - Score: 0.4356 loss: 0.0091 - val_loss: 0.0297
	Epoch 115 - Score: 0.4270 loss: 0.0091 - val_loss: 0.0272
	
A considerable performance increase was gained by dropping the learning rate. This fine tuning with a small learning rate helped go through the plateau the 0.01 learning rate was stuck on.

I then trained for another 30 epochs at a further reduced learning rate of 0.0001, but no signifcant improvemennt in val loss or score emerged. However, slight drop in loss was noticed. At this point the model may have begun to overfit so I reverted back the the epoch 115 model for final selection. Even though epoch 114 has a higher score, it's val loss is also higher which indicates that this must just be noise because of the datasets and not indicating a better model. Thus I stuck with epoch 115.

### Results
The model info and final results for the selected model are shown below.

Epochs trained: 115
Batch Size: 32
Learning Rate: 0.01 (first 100 epochs), 0.001 (last 15).

Loss: 0.0091, Validation Loss: 0.0272

**IoU Scores**
*When Quad is following behind hero*
true positives: 539, false positives: 0, false negatives: 0

*When Quad is on patrol and no target visable*
true positives: 0, false positives: 40, false negatives: 0
 
*When Quad can detect hero far away*
true positives: 116, false positives: 1, false negatives: 185

Weight: 0.7435
Final IoU: 0.5743
Final Score: 0.4270

### Performance on Simulator
When operating on the simulator, the model is able to pick up the hero from a reasonable distance, and once it closes in never lose sight of it. This model had 100% performance once it got close to the hero (evidenced by 539 true positives), and thus never lost it even in big crowds.

