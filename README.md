<p align="center"><img src="images/cellfie_segmentation2.gif"></p></br>

# cellfie

<a href="https://www.teepublic.com/tank-top/2147895-cell-fie"><img src="images/cellfie.png" align=right></a>

`cellfie` (**CELL** **FI** nd **E** r) automatically segments neurons in calcium imaging videos using fully convolutional neural networks.

Neuroscientists use [calcium imaging](https://en.wikipedia.org/wiki/Calcium_imaging) to monitor the activity of large populations of neurons in awake, behaving animals (like in [this](https://www.youtube.com/watch?v=Nxa19uWC_oA) beautiful example). However, calcium imaging can be very noisy, making neuron identification challenging. `cellfie` addresses this problem using a two stage convolutional neural network approach. First, a *region proposal* network identifies potential neurons. Next, an *instance segmentation* network iteratively identifies individual neurons.

`cellfie` is under active development. I'm collaborating with [Eftychios Pnevmatikakis](https://www.simonsfoundation.org/team/eftychios-a-pnevmatikakis/) at the Simon's Foundation to see if neural networks combined with matrix factorization techniques (as used by [CaImAn](https://github.com/flatironinstitute/CaImAn/blob/master/README.md)) outperforms current approaches to calcium imaging segmentation. The datasets used to train `cellfie` were meticulously created by Eftychios and his team.
</br></br>


#### region proposal
<p align="center"><img src="images/rp_sample.png"></p>

First, a *region proposal* network segments all neurons from the background. Rather than passing enormous videos into the network, three summary images are created that collapse videos across time:

1. *correlation images* that represent the correlation of each pixel to all neighboring pixels. The idea here is that pixels within a neuron will be correlated with neighboring within-cell pixels.
2. *standard deviation images* that capture the variability of each pixel. Pixels belonging to neurons should have higher variability due to fluctuations in neural activity.
3. *median images* that simply capture the median pixel value across time..

I use a [U-Net](https://arxiv.org/abs/1505.04597)-style fully convolutional neural network to turn these summary images into a heatmap where each pixel represents the probability that the pixel belongs to a neuron (depicted above).
</br></br>

#### instance segmentation
<p align="center"><img src="images/is_sample.png"></p>

But how can we find individual neurons? I train a second *instance segmentation* network that takes small subframes within the summary images as input. This network outputs a segmentation of the neuron centered within the current subframe (omitting other neurons in the frame!), in addition to a likelihood that there is a neuron centered at that location.
</br></br>

#### putting it all together
<p align="center"><img src="images/cellfie_segmentation2.gif"></p>
Putting together the region proposal and instance segmentation networks allows us to find all neurons in a video. First, I find potential neurons by looking for local maxima in the probability map generated by the region proposal network (the middle image above). Next, I pass subframes centered at these maxima to the instance segmentation network, keeping only segmentations with high likelihood scores.
</br></br>

#### cellfie philocellphy
*Data regularities are your friend!* Many machine learning algorithms are built to handle data 'in the wild', e.g. images from cell phones or self-driving cars that are enormously variable. Scientific datasets, however, have remarkable stereotypy that can be leveraged when building new algorithms. For example, [Mask R-CNN](https://arxiv.org/abs/1703.06870) uses an array of bounding boxes to handle objects of different sizes. But in microscopy data, we already know how big a cell should be! This makes our job way easier, and our algorithms way faster. Similarly, `cellfie` uses pretty shallow networks that are trained from scratch (without needing [transfer learning](https://machinelearningmastery.com/transfer-learning-for-deep-learning/)) in only ~ a half hour (on a crummy GPU!), which would not be possible on standard datasets</br></br>

#### notes

* This work is inspired by [DeepMask](https://arxiv.org/abs/1506.06204), which presents a really neat approach to instance segmentation.
* `cellfie` is under active development. If you are interested in trying `cellfie` on your data, please let me know.
