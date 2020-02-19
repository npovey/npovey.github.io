#### A Sparse-View CT Reconstruction Method Based on Combination of DenseNet and Deconvolution.

Starting a new progect. The plan is to try the DD-Net on my sparse-view data. Here is the original paper.

The original paper: https://ieeexplore.ieee.org/document/8331861>

The original project GitHub page: <https://github.com/zzc623/DD_Net>

##### Day 1: What is deconvolution?

Convolution is when we take the image and try to shrink it. Deconvolution is basically the same as a convolution layer, except instead of subsampling we do upsampling and the 'stride' is interpreted oppositely. So stride=2 would do upsampling by 2 (output has twice more things) instead of downsampling by 2.