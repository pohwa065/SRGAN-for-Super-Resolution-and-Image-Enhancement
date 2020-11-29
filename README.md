# SRGAN-for-Super-Resolution-and-Image-Enhancement
A Tensorflow implementation of SRGAN based on CVPR 2017 paper [Photo-realistic single image super-resolution using a generative adversarial network](https://arxiv.org/abs/1609.04802) to generate high resolution (HR) images from low resolution (LR) image. In this work, we use SRGAN to up-scale 32x32 images to 128x128 pixels. Meanwhile, we evaluate the impact of different camera parameters on the quality of final up-scaled (high resolution) images and infer from these stimuli to understand what the network is able to learn.

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
Generator and Discriminator in SRGAN. Ledig, Christian, et al. "Photo-realistic single image super-resolution using a generative adversarial network." Proceedings of the IEEE conference on computer vision and pattern recognition. 2017(https://arxiv.org/abs/1609.04802). <br>

## S-CIELAB representation
We use as evaluation matrix of image quality. [S-CIELAB](http://scarlet.stanford.edu/~brian/scielab/introduction.html) is an extension of the CIE L*a*b* DeltaE color difference formula and provides human spatial sensitivity difference between a reference and a corresponding test image. The key components of calculating S-CIELAB representation include color transformation and the spatial filtering steps that simulates the human visual system before the standard CIELAB Delta E calculations.<br>
![Picture11](https://user-images.githubusercontent.com/65942005/100526525-7bf18c00-317e-11eb-9b3f-887526b5aa36.jpg) <br>

## Result
### Effect of missing pixels
SRGAN model is able to deal with missing/noise pixels (about 10% in our experiment) and generate HR images not only have smooth edges but also restore the details. <br>

![Picture12](https://user-images.githubusercontent.com/65942005/100526718-71d08d00-3180-11eb-8f10-63aa751a9317.jpg) <br>


### HR images from trained SRGAN models
SRGAN model can be trained to perform super-resolution and image enhancement (including color correction and de-blurring) simultaneously <br>

The following matrices summarize the results from all four trained SRGAN models. The 'input' rows shows the input LR images (original and processed images using three different camera settings as described in method section). Each cell in the matrix represent the result HR images generated by a given model from a given input LR. For example, in the each 4x4 matrix, the 3rd row, 2nd column image is the result generated by Model_SR_Pixel using LR images processed by camera setting1. It is obvious that the images in the diagonal have higher qualities (closer to the target images) compared with the off-diagonal images because these are how the models were originally trained for.<br>

**Model_SR:** this model is trained to perform super resolution only. As expected, the result looks simply upscaled without changing other characteristics of the input images such as color and focus.<br>
**Model_SR_Color:** the outline and details looks similar to Model_SR. Also, because this model is trained to do color correction, the color tone is different between the input and output images (becomes 'brighter' in general).<br>
**Model_SR_Pixel:** unlike Model_SR and Model_SR_Color, result from this model looks relatively un-natural. However, when the input image is from camera setting 3 (reduction of system MTF due to large sensor pixel), the resulting HR image improved a lot - it learns how to restore spatial resolution to some extent.<br>
**Model_SR_Deblur:** This model successfully learned how to de-blur. It is also interesting that all of its output images seems to remain in good focus regardless whether the input image is within/out of focus.<br>

![Picture17](https://user-images.githubusercontent.com/65942005/100526346-beb26480-317c-11eb-9a84-55fcf1beb2a6.jpg)<br>

### S-CIELAB delta E maps
The S-CIELAB delta E maps show the difference between target images and the model generated images. Consider these difference as 'residue' (mainly at the edges) that the model can improve, it might be interesting in future work to replace/add S-CIELAB representation to the generator's loss function. The reason being, one of the major changes that a more advanced version of SRGAN model (called Enhanced SRGAN, [SRGAN](tps://arxiv.org/abs/1809.00219)) have done is to use feature maps before activation for calculating content loss. As we extract feature maps from relatively deep layer in VGG19 layer, some of the features after activation becomes inactive and contains fewer information. It is possible that S-CIELAB can provide additional information, especially from human spatial sensitivity point of view, to the generator during training and create a new class of super resolution images that focus more on how accurate the reproduction of a color is to the original when viewed by a human observer.<br>

![Picture27](https://user-images.githubusercontent.com/65942005/100526352-cffb7100-317c-11eb-83a0-1e664f367add.jpg)<br>

##  Proposed future work for model improvement
We proposed in the future work to use S-CIELAB delta E maps as generator loss function to incoporate human spatial color sensitivity during training. This could enable a new class of super resolution images that focus more on the reproduction of a color patterns when viewed by a human observer.
