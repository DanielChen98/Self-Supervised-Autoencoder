## Self-Supervised Autoencoder
This project uses the principle of self-supervision to enforce semantic awareness in the encoder which result into having better bottleneck representation. 

Three Autoencoder variants are trained.
1. Vanilla autoencoder trained using 224x224 images
2. Autoencoder trained on 9 tiles of 96x96 of an image
3. Autoencoder trained with an auxiliary task to solve jigsaw puzzle (https://arxiv.org/abs/1603.09246) where a classifier is attached to bottleneck representation of the autoencoder to predict the permutation number which is used to shuffle tiles of an image. 


### Implementation details
Self-supervised task used is a jigsaw puzzle, it involves creating 9 tiles out of each image and then shuffle them using some permutation. Now Autoencoder is provided a batch of tiles to recontruct them but we also attach a small classifier at the bottleneck which has to use bottleneck representation to predict the permutation index used for that particular set of tiles, permutation indexes are the labels for classifier. 

This image may help to visualize it. It has a classifier attached to bottleneck which has to predict if image if vertically flipped. 

![alt text](https://github.com/SharadGitHub/Self-Supervised-Autoencoder/blob/master/images/img.jpg)


### Training
I used YFCCM100 dataset, it has nearly 94 million images , the network is trained for 94 epochs where each epoch iterates over unique 1 million images. Data augmentation is used as the network was overfitting and various measures are taken to not let the network find shortcuts to classify images such as classifying using edge continuity in tiles. 

### Multi task Learning
Since two tasks are jointly trained, autoencoder and classifier, this is a use case of multi task learning. Classifier acts as auxiliary task to the main task of autoencoder. Therefore, there are two losses, L1 loss for Autoencoder and Crossentropy loss for classifier. To train two losses jointly such that one loss does not overwhelm the other loss and have more influenced over gradients I used a technique to weigh both losses as per this paper  (https://arxiv.org/abs/1705.07115) 

After training for 94 epochs, classifier gives 80% accuracy while predicting permutation index applied to the image.

### Input Images for network.
These are some input images tiles which are shuffled. 

Image 1                    |  Image 2
:-------------------------:|:-------------------------:
![](https://github.com/SharadGitHub/Self-Supervised-Autoencoder/blob/master/Jigsaw%20Task/skeleton/res/saved_test_input/input_grid_rank_6_3907.png)  |  ![](https://github.com/SharadGitHub/Self-Supervised-Autoencoder/blob/master/Jigsaw%20Task/skeleton/res/saved_test_input/input_grid_rank_4_3907.png)


### Recontructed Images
These are reconstructed images from Autoencoder.

Image 1                    |  Image 2
:-------------------------:|:-------------------------:
![](https://github.com/SharadGitHub/Self-Supervised-Autoencoder/blob/master/Jigsaw%20Task/skeleton/res/saved_test_output/output_grid_rank_6_3907.png)  |  ![](https://github.com/SharadGitHub/Self-Supervised-Autoencoder/blob/master/Jigsaw%20Task/skeleton/res/saved_test_output/output_grid_rank_4_3907.png)

### Evaluation
To evaluate if auxilliary task of predicting permutation index has helped Autoencoder we take two different pretrained encoder parts of autoencoders which are basically VGG16 and fine tune them on Imagenet to compare. First we took an autoencoder which is trained on full images without any auxiliary task and another which is trained with self-supervised technique. For both, we only took the encoder weights and plug them in VGG to see how they perform relative to each other on Imagenet. 

self-supervised autoencoder accuracy|  vanilla Autoencoder accuracy
:-------------------------:|:-------------------------:
![](https://github.com/SharadGitHub/Self-Supervised-Autoencoder/blob/master/Imagenet/skeleton/res/plots/static_weight/metric.png)  | ![](https://github.com/SharadGitHub/Self-Supervised-Autoencoder/blob/master/Imagenet/skeleton/res/plots/vanilla_ae/metric.png)

### Results 
As it could be seen, the images regenerated from self-supervised Autoencoder are very good and it is not so easy to differentiate between actual and regenerated images. From evaluation plots, it could be noticed that self-supervision technique did help to improve autoencoder, as compared to vanilla autoencoder it has nearly double the accuracy on Imagenet. Such pretext tasks of training a network by creating labels by using some property of the data could be later used to train  

Note: Here, while training on Imagenet data we only trained last FC layers of VGG while freezing all previous layers, that's why accuracy is not so great. One could also try to fine tune earlier layers with smaller learning rate to get improved accuracy. 
