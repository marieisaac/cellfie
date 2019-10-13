# cellfie

<a href="https://www.teepublic.com/tank-top/2147895-cell-fie"><img src="images/cellfie.png" align=right></a>

`cellfie` (**CELL** **FI** nd **E** r) automatically segments neurons in calcium imaging videos using fully convolutional neural networks.

Neuroscientists use [calcium imaging](https://en.wikipedia.org/wiki/Calcium_imaging) to monitor the activity of large populations of neurons in awake, behaving animals (like in [this](https://www.youtube.com/watch?v=Nxa19uWC_oA) beautiful example). However, calcium imaging can be very noisy, making neuron identification challenging. `cellfie` solves this problem using a two stage convolutional neural network approach. First, a *region proposal* network identifies potential neurons. Next, an *instance segmentation* network iteratively identifies individual neurons.

`cellfie` is a work in progress. I'm collaborating with [Eftychios Pnevmatikakis](https://www.simonsfoundation.org/team/eftychios-a-pnevmatikakis/) at the Simon's Foundation to see if neural networks combined with matrix factorization techniques (as used by [CaImAn](https://github.com/flatironinstitute/CaImAn/blob/master/README.md)) outperform current approaches to calcium imaging segmentation.<br/><br/>


#### region proposal
<img src="images/rp_sample.png" align="center">

First, a *region proposal* network segments all neurons and the background. Rather than passing entire videos into the network (which can be gigabytes of data!), three summary images are created that collapse the videos across time:

1. *correlation images* representing the correlation of each pixel to all neighboring pixels. The idea here is that neurons within a cell will be correlated with neighboring within-cell pixels.
2. *standard deviation images* that capture the variability of each pixel. Pixels belonging to neurons will have higher variability due to fluctuations in neural activity.
3. *median images*

I use a [U-Net](https://arxiv.org/abs/1505.04597)-style fully convolutional neuronal network that takes these summary images as input and produces a probability map where each pixel represents the probability that the pixel belongs to a neuron.</br>

#### instance segmentation
<img src="images/is_sample.png" align="right">
But how can we find *individual* neurons? I train a second *instance segmentation* network that takes small subframes within the summary images as input. This network outputs:

1. a segmentation of the neuron centered within the current subframes (omitting other neurons in the frame!)
2. a likelihood that there is in fact a neuron centered at that location


#### putting it all together
By putting together the region proposal and instance segmentation networks we can easily find all of the neurons in a video. First, I find potential neurons by looking for local maxima in the probability map generated by the region proposal network. Next, Pass subframes centered at these maxima to the instance segmentation network, keeping only segmentations with high likelihood scores!

#### cellfie philocellphy
This work is inspired by [this](https://arxiv.org/abs/1506.06204) approach to instance segmentation.
