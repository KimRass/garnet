- Github: https://github.com/KimRass/garnet

# Paper Summary
- Paper: [The Surprisingly Straightforward Scene Text Removal Method With Gated Attention and
Region of Interest Generation: A Comprehensive Prominent Model Analysis](https://www.ecva.net/papers/eccv_2022/papers_ECCV/papers/136760436.pdf)
## Related Works
- To the best of our knowledge, there is no other previous study that considered both TSR and TSSR while performing STR.
- *EnsNet: Simple and fast because it doesn’t need any auxiliary inputs. However, its results are blurry and of low quality. Furthermore, its practical applications are limited because it is impossible to only erase text in a certain region.*
- *MTRNet: Requires the generation of text box region masks through manual or automatic means.*
- MTRNet++ and EraseNet: Proposed a coarse-refinement two-stage network. While the results are of higher quality, the model is much bigger, slower, and too complicated.
- *Zdenek et al.: Used an auxiliary text detector to retrieve the text box mask, then attempted to erase text through a general inpainting method.*
- *Tang et al.: Can train their model in a rather efficient manner because they erase text by cropping only the text regions. However, this method ignores all global contexts other than the cropped region and has difficulty precisely cropping the region of curved texts.*
## Experiment
- We use the same standardized training/testing dataset to evaluate the performance of several previous methods after standardized re-implementation.
## Methodology
- We also introduce a simple yet extremely effective Gated Attention (GA) and Region-of-Interest
Generation (RoIG) methodology in this paper.
- *First, our proposed method takes a text box mask as the input to our model, leading to the option for users to selectively erase only the text that they wish to.* Second, our proposed method can localize the TSR and TSSR to erase text in a surgical manner.
### Gated Attention
- *GA uses attention to focus on the text stroke as well as the textures and colors of the surrounding regions to remove text from the input image much more precisely.*
- GA is a simple yet extremely effective method that uses attention to focus on the text stroke and its surroundings.
- GA not only distinguishes between the background and the text stroke, but also utilizes attention to identify both Text Stroke Region (TSR) and the Text Stroke Surrounding Region (TSSR).
- Previous STR models [18, 17] used a text box region as well as a text stroke region in an attempt to perform precise text removal. However, TSSR was mostly overlooked. After finding inspiration from observing how humans must alternate between paying attention to the text stroke regions and the surrounding regions of the text while manually performing STR, we devised the GA.
### Region-of-Interest Genration
- *RoIG is applied to focus on only the region with text instead of the entire image to train the model more efficiently.*
- *Because all previous studies performed STR on the entire image, artifacts frequently occurred in non-text regions. Thus, we devised the RoIG, which allows our STR model to only generate a result image from within the text box region instead of wasting resources attempting to perform STR on the full image.*
### Architecture
- The generator G takes the image and corresponding text box mask as its input and produces a non-text image that is visually plausible. The discriminator D takes both the input of the generator and the target images as its input and differentiates between real images and images produced by the generator.
#### Generator
- The generator has an FCN-ResNet18 backbone and skip connections between the encoder and decoder. The model is composed of five convolution layers paired with five deconvolution layers with a kernel size of 4, stride of 2, and padding of 1. The convolution pathway is composed of two residual blocks, which contains the proposed Gated Attention (GA) module.
#### Discriminator
- For training, we use a local-aware discriminator proposed in EnsNet, which only penalizes the erased text patches. The discriminator is trained with locality-dependent labels, which indicate text stroke regions in the output tensor. It guides the discriminator to focus on text regions.
## Train
- *We generated a pseudo text stroke mask, automatically calculated by taking the pixel value difference between the input image and ground truth image.* The TSR masks help train the TSR attention module to distinguish the TSR.
- *The TSSR mask is the intersection region between the exterior of the pseudo text stroke mask and the interior of the text box mask.* It makes the TSSR attention module train to focus on the colors and textures of the TSSR.
### Loss
- The GA learns the gate parameter on its own, and can thus adjust the respective attention ratios allocated to TSR and TSSR during training. The loss function is designed to be applied only within the text box regions, not to the entire score map.