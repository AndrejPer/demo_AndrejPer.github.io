---
layout: page
title: CorrSeg
description: UNet-inspired CNN for Semantic Segmentation
img: assets/img/well-drill.png
importance: 1
category: competition
related_publications: true
---

### Problem description

This project was developed for a [ChallengeData](https://challengedata.ens.fr) competition, presented by [SLB](https://www.slb.com/) and focused on semantic segmentation. [Semantic segmentation](https://en.wikipedia.org/wiki/Image_segmentation) is a crucial task in computer vision, involving the classification of each pixel in an image into a specific category. You can read more about the competition and the requirements [here](https://challengedata.ens.fr/participants/challenges/144/).

### Network Diagram

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/network_diagram.png" title="Network diagram" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

### How did I do it

The UNet-inspired {% cite  ronneberger2015u %} Convolutional Neural Network architecture was chosen due to its effectiveness in biomedical image segmentation. The model includes an encoder-decoder structure that helps in capturing the context and fine details of the images.
There are three triplets for down-sampling and three for upsamping. There are also two skip conncetions, which help in preventing vanishing gradient.

#### Data Preprocessing

Before training the model, the input data needs to be preprocessed. First, I imputed <code>NaN</code> values with 0's. Next, I applied horizontal and vertical flip to augment the dataset. This is a convenient trick when dealing with CNNs, since it increases your training set and helps the network generalize better. Treating outliers seemed to make the predictions worse, so I didn't modify those pixels.

#### Network Architecture

The downsampling step involves three consecutive convolutional layers, each followed by a ReLU activation function. This helps in capturing the high-level features and reducing the spatial dimensions of the input.

The upsampling step also consists of three convolutional layers, each followed by a ReLU activation function. This helps in recovering the spatial dimensions and generating the final segmentation map. I tried applying [batch normalization](https://en.wikipedia.org/wiki/Batch_normalization) {% cite pmlr-v37-ioffe15%}, but it degraded the perfomance. [Dropout](<https://en.wikipedia.org/wiki/Dilution_(neural_networks)#Dropout>) {% cite  srivastava2014dropout%} was also unhelpful, probably because the network was not that close to being overly complex given the input.

The weights of the network are initialized using a Kiming He normal distribution. The Kaiming initialization method is calculated as a random number with a Gaussian probability distribution with a mean of 0 and a standard deviation of $ \sqrt{\frac{2}{n}} $, where $n$ is the number of inputs to the node. This initialization strategy helps in stabilizing the training process and improving the convergence speed. It works particularly well for the task of semantic segmentation. You can read more about it in their paper {% cite he2015delving %}.

Unlike traditional UNets {% cite  ronneberger2015u %}, this network does not use maxpooling layers due to the small dimensions of the input images. Instead, it relies on the convolutional layers to halve the dimensions while also learning features. You could say, we have too few pixels to lose. Moreover, the dimension being only $36\times 36$, I could only easily make a "three level" network instead of a 5 level one. Once the dimension of the input into a hidden layer becomes odd, the things get a bit more complicated.

#### Training

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/loss.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/jaccard.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

The model was trained using the Adam optimizer {% cite kingma2014adam%} with a learning rate of 0.0001. The loss function used was the binary cross-entropy loss, which is suitable for binary classification tasks like semantic segmentation. The model was trained for 75 epochs with a batch size of 128 for training and 256 for validation. Why different sizes? Training requires roughly twice as much memory (forward pass and backpropagation) than validation (only a forward pass). The training data was split into 80% training and 20% validation sets to monitor the model's performance during training. In the competition, the score is given as the intersection of the union or [Jaccard Index](https://en.wikipedia.org/wiki/Jaccard_index) over the image pixels. Hence, IoU was estimated at every iteration as the average over all batches of validation examples.

### Leaderboard (May 2024)

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/leadboard.png" title="Network diagram" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        <p>If you would like to see my code, checkout the <a href="https://github.com/AndrejPer/CorrSeg" target="_blank">repo</a>. For more details, visit the <a href="https://challengedata.ens.fr/participants/challenges/144/" target="_blank">project page</a>. You can also take a look at a more detailed <a href="https://github.com/AndrejPer/CorrSeg/blob/main/Project_Report.pdf" target="_blank">report</a>.</p>
    </div>
</div>
