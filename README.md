# Stereobokeh implementation with open cv tools

In this repository, an implementation of Bokeh effect with stereo images is shown, based on Gaussian blur with parameters adjusted by transformations on features obtained from optical flow from the source images.

---
## Optical flow estimation

$$I(x,y,t) = I(x+\Delta x, y+\Delta y, t + \Delta t)$$

$$I(x+\Delta x, y+\Delta y, t + \Delta t) \approx I(x,y,t) + \dfrac{\partial I}{\partial x}\Delta x + \dfrac{\partial I}{\partial y}\Delta y + \dfrac{\partial I}{\partial t}\Delta t$$

$$\dfrac{\partial I}{\partial x}u + \dfrac{\partial I}{\partial y}v + \dfrac{\partial I}{\partial t} = 0$$

$$\nabla I \cdot \vec{V} = -\dfrac{\partial I}{\partial t}$$


---
## Experimental setup

First, a suitable space for capturing stereo image was built. For this, a tripod was disposed on top of a measuring tape as shown: 

![exp1.jpg](attachment:743ce4e9-a01a-49e7-9af7-3fbb327e4dae.jpg)


This disposition aids to keep an strict movement along a single axis with a controlled deplacement. 
With the help of some marks on the floor, it was possible to take stereo captures with a controlled separation of 5cm:

![exp2.jpg](attachment:51e648ff-18a8-4cb4-a64c-8b14c81deb9c.jpg)





---
# Depth estimation using Stereo Block Matching (StereoBM)

The stereo block matching algorithm assumes that the corresponding pixels in the left and right images are found along a scanline, hence sharing the same Y coordinate. Since more than one pixel may match the intensity of the source pixel, the algorithm tries to find the best match to a block centered in such pixel along the scanline in the second image. For more information please refer to the OpenCV documentation, or check the page in [this link](https://learnopencv.com/depth-perception-using-stereo-camera-python-c/).

In order to use the OpenCV StereoBM::compute, both left and right images should be transformed into 8 bit unsigned with only one plane depth.




## Blur application based on estimated optical flow

Now that we have optical flow magnitud for every pixel of the image, we can apply a Gaussian blur proportional to this magnitude.

For this, we will create sub windows of a fixed size and build every of the blur transformation of these windows based on the mean optical flow magnitude in this region.

In other words, we can modify standard deviation of the Gaussian Blur for every pixel neighborhood based on the average optical flow intensity of this region.
This assumption will lead to an effect of camera focus on closest objects.

## Modeling construction of final image (no overlap solution)


An initial purposed solution would be to construct the final images by puting together all transformed frames without overlap. 

Let:

$$\mathbf{X}_f = \mathbf{X}_{(x_i,y_i)}^{(x_j,y_j)}$$

Be an input frame from the original image $\mathbf{X}$ between points $(x_i,y_i)$ and $(x_j,y_j)$, where $w_x = x_j - x_i$ and $w_y = y_j - y_i$ are the corresponding frame widths for this region.


For every frame, the resulting blured output frame will be as follows:

$$\mathbf{Y}_f = \mathcal{G}(\mathbf{X}_f,\mathbf{K}, \sigma_x, \sigma_y)$$

Where: 

$\mathcal{G}$ is a Gaussian blur operation, 

$\mathbf{K}$ is the transformation Kernel, 

$\sigma_x$ is the standard deviation in the X axis, and

$\sigma_y$ is the standard deviation in the Y axis.

If we asume an equal standard deviation in both axis, we can rewrite:

$$\mathbf{Y}_f = \mathcal{G}(\mathbf{X}_f,\mathbf{K}, \sigma)$$

Now, we can obtain $\sigma$ from some sort of transformation of the optical flow.

Let:

$$\mathbf{F}_f = \mathbf{F}_{(x_i,y_i)}^{(x_j,y_j)}$$

Be a frame from the opical flow magnitude previously obtained from the source images. 

Let $\bar{\mathbf{F}}_f$ be the average magnitude of the optical flow in a certain frame.

Then, we can model the Gaussian blur transformation standard deviation ($\sigma$) based on this average magnitude as follows:

$$\sigma  = \Phi(\bar{\mathbf{F}}_f)$$

Where $\Phi$ is some transformation function.

In the previous code cell, we asumed $\Phi$ as a linear function, defined as:

$$\Phi(\bar{\mathbf{F}}_f) = \frac{1}{\alpha} \bar{\mathbf{F}}_f$$

Where $\alpha$ is a scaling factor that handles the sensitivity of bluring by changes in depth.

Finally, we can reconstruct the full output image ($\mathbf{Y}$) based on the several output windows obtained with these rules by applying the same geometrical behavior from the input:

$$\mathbf{Y}_f = \mathbf{Y}_{(x_i,y_i)}^{(x_j,y_j)}$$

