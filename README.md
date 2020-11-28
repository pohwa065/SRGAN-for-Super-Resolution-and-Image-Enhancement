# Deep-Learning-for-Super-Resolution-and-Image-Enhancement
[Super-Resolution Generative Adversarial Networks (SRGAN)](https://arxiv.org/abs/1609.04802) is a deep learning application to generate high resolution (HR) images from low resolution (LR) image. In this work, we use SRGAN to up-scale 32x32 images to 128x128 pixels. Meanwhile, we evaluate the impact of different camera parameters on the quality of final up-scaled (high resolution) images and infer from these stimuli to understand what the network is able to learn.

![Picture1](https://user-images.githubusercontent.com/65942005/100526323-93c81080-317c-11eb-991a-2299057ba5b0.png)
Data processing and model training pipeline: Original image is processed with different camera parameters using [ISET Camera Designer](https://github.com/iset/isetcam/wiki). These images are resized to 32x32x3 and served as the (LR) input to the generator. The target HR images are the original ones which are not processed. Total four models were trained:<br>
**Model_SR:** SRGAN model that does super resolution only <br>
**Model_SR_Color:** SRGAN model that does super resolution and color correction <br> 
**Model_SR_Pixel:** SRGAN model that does super resolution and restore spatial resolution due to reduction of system MTF <br> 
**Model_SR_Deblur:** SRGAN model that does super resolution and deblur<br> 

## DataSet
### Training ### 
1800 cat and dog images were downloaded from Flickr and Pixabay. These images were processed using three different camera settings using ISET Camera Designer developed by David Cardinal, 2020, ''F20-PSYCH-221'', Stanford University, under the framework of [ISETCAM](https://github.com/iset/isetcam/wiki) 

![Picture10](https://user-images.githubusercontent.com/65942005/100526342-b8bc8380-317c-11eb-83d6-06f0e971b66e.jpg)

**Camera setting 1**- F/4; aperture diameter: 1mm, pixel size: 1umx1um; under exposing; default image processor. To produce images that require color correction as we use default image processor while under exposure, these images have warm tone <br>
**Camera setting 2**- F/4; aperture diameter: 1mm, pixel size: 25umx25um. In general, large pixel size is desirable because it results in higher dynamic range and signal-to-noise ratio. However, the reduction in spatial resolution and system MTF introduce severe pixelated effect <br>
**Camera setting 3**- F/22; aperture diameter: 0.176mm; pixel size: 1umx1um. Images looks much blurry compared with original images because they becomes diffraction limited at small aperture value <br>

### Testing ### 
About 100 images, not necessarily cats and dogs, are used for evaluation. As an input to four trained models (Model_SR, Model_SR_Color, Model_SR_Pixel, and Model_SR_Deblur), these images went through the same camera settings and resized to 32x32x3 similar to the training dataset.

## SRGAN Model
In this work, we use Tensorflow implementation of ["Photo-realistic single image super-resolution using a generative adversarial network."](https://arxiv.org/abs/1609.04802). Small modifications in are done as follows:<br>
**PixelShuffler x2:** This is for feature map upscaling. We use 2x ‘deconv2d’ Keras built-in function for implementation. <br>
**PRelu(Parameterized Relu):** PRelu introduces learnable parameter that makes it possible to adaptively learn the negative part coefficient. We use Relu as activation function for simplicity. <br>

![Picture3](https://user-images.githubusercontent.com/65942005/100526325-975b9780-317c-11eb-8528-1785a6659b10.jpg)<br>
Ledig, Christian, et al. "Photo-realistic single image super-resolution using a generative adversarial network." Proceedings of the IEEE conference on computer vision and pattern recognition. 2017(https://arxiv.org/abs/1609.04802). <br>

## S-CIELAB representation
We use as evaluation matrix of image quality. [S-CIELAB](http://scarlet.stanford.edu/~brian/scielab/introduction.html) is an extension of the CIE L*a*b* DeltaE color difference formula and provides human spatial sensitivity difference between a reference and a corresponding test image. The key components of calculating S-CIELAB representation include color transformation and the spatial filtering steps that simulates the human visual system before the standard CIELAB Delta E calculations.<br>
![Picture11](https://user-images.githubusercontent.com/65942005/100526525-7bf18c00-317e-11eb-9b3f-887526b5aa36.jpg) <br>

## Result
### Effect of missing pixels
SRGAN model is able to deal with missing/noise pixels (about 10% in our experiment) and generate HR images not only have smooth edges but also restore the details. <br>

### HR images from trained SRGAN models
SRGAN model can be trained to perform super-resolution and image enhancement (including color correction and de-blurring) simultaneously <br>

### S-CIELAB delta E maps
The S-CIELAB delta E maps show the difference between target images and the model generated images. Consider these difference as 'residue' (mainly at the edges) that the model can improve, it might be interesting in future work to replace/add S-CIELAB representation to the generator's loss function. The reason being, one of the major changes that a more advanced version of SRGAN model (called Enhanced SRGAN, ESRGAN) have done is to use feature maps before activation for calculating content loss. As we extract feature maps from relatively deep layer in VGG19 layer, some of the features after activation becomes inactive and contains fewer information. It is possible that S-CIELAB can provide additional information, especially from human spatial sensitivity point of view, to the generator during training and create a new class of super resolution images that focus more on how accurate the reproduction of a color is to the original when viewed by a human observer.

