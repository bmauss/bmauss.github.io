---
layout: post
title:      "Under the Neural Hood"
date:       2020-09-20 22:59:49 +0000
permalink:  under_the_neural_hood
---


Did you catch that Batman reference? No? Too bad!

You're probably wondering why I even chose that as a title!  Well, let me tell you!  We just finished with the Module 4 project.  I chose to work on Image Recognition utilizing Deep Learning.  In essense, create a neural network that can identify a patient with pneumonia given a chest x-ray.  I had a great time with it.  

I have to say the best part, for me, was utilizing feature maps to visualize what was happening inside the model after each layer activation, thus, allowing me to analyze what the model was doing.  

## A Tale of Two X-Rays

Let's look at a couple of x-rays.

![Imgur](https://i.imgur.com/8VS0AcO.png)

![Imgur](https://i.imgur.com/p6QYmHG.png)

Both of these were taken from the Test set and had comparable resolutions.  Preprocessing included resizing each to 128 x 128 ppi and normalizing the images to grayscale.  This way they could be fed into the model and I could use model.predict(<image_name>) to pass them through the model.  Then we'd utilize this very nifty function  to  visualize all of the layers (that my computer could handle). 

```
def feature_maps(image_tensor):
    """
    Input an image tensor.
    
    Function will extract the model's layer outputs and create a model that 
    displays the feature maps.  It will then extract the layer names for 
    labeling during the plotting stage.  It will then plot the images in the 
    using the pixel value intensities that the model uses at every layer of the 
    neural network.
    """
    layer_names = []
    for layer in model.layers[:12]:
        layer_names.append(layer.name) # Names of the layers, so you can have them as part of your plot

    activations = activation_model.predict(image_tensor)    

    images_per_row = 16

    for layer_name, layer_activation in zip(layer_names, activations): # Displays the feature maps
        n_features = layer_activation.shape[-1] # Number of features in the feature map
        size = layer_activation.shape[1] #The feature map has shape (1, size, size, n_features).
        n_cols = n_features // images_per_row # Tiles the activation channels in this matrix
        display_grid = np.zeros((size * n_cols, images_per_row * size))
        for col in range(n_cols): # Tiles each filter into a big horizontal grid
            for row in range(images_per_row):
                channel_image = layer_activation[0,
                                                 :, :,
                                                 col * images_per_row + row]
                channel_image -= channel_image.mean() # Post-processes the feature to make it visually palatable
                channel_image /= channel_image.std()
                channel_image *= 64
                channel_image += 128
                channel_image = np.clip(channel_image, 0, 255).astype('uint8')
                display_grid[col * size : (col + 1) * size, # Displays the grid
                             row * size : (row + 1) * size] = channel_image
        scale = 1. / size
        plt.figure(figsize=(scale * display_grid.shape[1],
                        scale * display_grid.shape[0]))
        plt.title(layer_name)
        plt.grid(False)
```
(Note: All credit for this function goes to **Gabriel Pierobon** on **Medium**.  A HUGE thank you to him. Go check out his [article]https://towardsdatascience.com/visualizing-intermediate-activation-in-convolutional-neural-networks-with-keras-260b36d60d0! for a deeper dive into visualizing intermediate activations)

Now that the all due credit has been paid, let's dive in!

## The First Layer
Here is the first layer of the "Normal" chest x-ray:
![Imgur](https://i.imgur.com/3LATFWa.png)

And here is the first layer of the "Pneumonia" x-ray:
![Imgur](https://i.imgur.com/eihRECI.png)

You'll notice that the primary focus of the first layer in both cases is detecting the edges.  You can see it finding the silhouettes of the body and ribcage in each image, setting them apart from the background.  

Another thing the model is doing is detecting the "depth" of the image.  Now, I'm going to be saying things like "detecting the depth" or "creating a negative of the image", but the model isn't REALLY doing that.  It is only given matrices of pixel intensities for red, green, and blue (or in the case of our normalized grayscale images: black, white, and the various shades of gray in each matrix) for the different channels. However, you'll see through the ways that the images are transformed and manipulated, that this is the best way to describe what's happening.

So, yeah, the model is detecting the "depth" of the image.  You can see this as it filters out the more faint pixels that makes up the ribcage (and pneumonia), leaving only the more dense bones (or the pixels with higher intensity).

The model is also adjusting the contrast and creating negatives of the images.  By adjusting the contrast, it can filter out noise and better differentiate between the image subject and the background.  Creating a negative of the image allows the model to highlight noise from pneumonia because the noise will be less dense (more translucent) than the noise from, say, body silhouettes (which will be more opaque).

## Second Layer
Normal:
![Imgur](https://i.imgur.com/BThrk6y.png)

Pneumonia:
![Imgur](https://i.imgur.com/T2zJLLK.png)

The images have become slightly more fuzzy as a result of the filters in the convolution layers.

Here, we're seeing less filtering out the foreground noise.  Instead, the model appears to be searching more in the middle.  Images that used to just be a shadow against a light screen have spots where the background is peeking through.

We're also seeing more instances where the contrast is either turned up (creatings more crisp images) or turned down (creating fuzzy images).  It reminds me of how I used to tune the contrast on my GameBoy (the original fat ones). 

Again, the model is still double checking itself by making negative images.  Overall, in the second layer, we're seeing less focus on the background and more focus on the middle and foreground.

## The Last Layer...that My Computer Can Handle
Normal:
![Imgur](https://i.imgur.com/cqytDRw.png)

Pneumonia:
![Imgur](https://i.imgur.com/7moOTyV.png)

The images are more compressed due to the filters, and yet the model ALMOST looks like it's found a pattern to help it properly classify images.  If it wasn't for the fact that this isn't the TRUE final layer, I'd say that this is the moment where the model says, "Ah-Ha!  Now, we're getting somewhere!"  In each case there are only 3 images (that are perceivable to the human eye) and in each of these images, the same transformations are happening:

* The first is a "standard" image, with the surface noise
* In the second image, the colors are inverted, and the contrast is turned up to bring out the background.  If you look at the pneumonia sample, this actually REALLY highlights the noise created by the virus.  
* The final picture, the contrast is returned to normal.  This, again, highlights the pneumonia, while also helping the model distinguish it from the rest of the body by showing off the translucent nature of the virus.

Now, this is all just my interpretation, though.  I would need to see all of the layers to fully grasp what the model is "thinking".  Still, it's interesting to see the logic and almost organic way that neural networks learn to classify images.  Millions of instances of trial and error lead to highly educated guesses and predictions.  And thanks to the power of Convolutional Neural Networks, we get to take a peek underneath the hood.

