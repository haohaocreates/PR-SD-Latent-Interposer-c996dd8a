# SD-Latent-Interposer
A small neural network to provide interoperability between the latents generated by the different Stable Diffusion models.

I wanted to see if it was possible to pass latents generated by the new SDXL model directly into SDv1.5 models without decoding and re-encoding them using a VAE first.

## Installation
To install it, simply clone this repo to your custom_nodes folder using the following command: `git clone https://github.com/city96/SD-Latent-Interposer custom_nodes/SD-Latent-Interposer`.

Alternatively, you can download the [comfy_latent_interposer.py](https://github.com/city96/SD-Latent-Interposer/raw/main/comfy_latent_interposer.py) file to your `ComfyUI/custom_nodes` folder as well. You may need to install hfhub using the command `pip install huggingface-hub` inside your venv.

If you need the model weights for something else, they are [hosted on HF](https://huggingface.co/city96/SD-Latent-Interposer/tree/main) under the same Apache2 license as the rest of the repo.

## Usage
See the image below for an example on how to use it. xl=>v1 conversion is almost flawless, **v1=>xl seems to produce artifacts.**

![LATENT_INTERPOSER_V3 1_TEST](https://github.com/city96/SD-Latent-Interposer/assets/125218114/849574b4-2565-4090-85d3-ae63ab425ee2)

Without the interposer, the two latent spaces are incompatible:

![LATENT_INTERPOSER_V3 1](https://github.com/city96/SD-Latent-Interposer/assets/125218114/13e2c01f-580e-4ecb-af1f-b6b21699127b)

## Training
Most of the training/preprocessing code is a 1:1 mirror from my latent upscaler. The folder layout it expects is also the same.

### Interposer v3.1
This is basically a complete rewrite. Replaced the mediocre bunch of conv2d layers with something that looks more like a proper neural network. No VGG loss because I still don't have a better GPU.

Training was done on combined Flickr2K + DIV2K, with each image being processed into 6 1024x1024 segments. Padded with some of my random images for a total of 22,000 source images in the dataset.

I think I got rid of most of the XL artifacts, but the color/hue/saturation shift issues are still there. I actually saved the optimizer state this time so I might be able to do 100K steps with visual loss on my P40s. Hopefully they won't burn up.

v3.0 was 500k steps at a constant LR of 1e-4, v3.1 was 1M steps using a CosineAnnealingLR to drop the learning rate towards the end. Both used AdamW.

![INTERPOSER_V3 1](https://github.com/city96/SD-Latent-Interposer/assets/125218114/daff0ae2-4739-4cef-ba54-ac1d156d3388)

### Older versions

<details><summary>Interposer v1.1</summary>

### Interposer v1.1
This is the second release using the "spaceship" architecture. It was trained on the Flickr2K dataset and was continued from the v1.0 checkpoint.
Overall, it seems to perform a lot better, especially for real life photos. I also investigated the odd v1->xl artifacts but in the end it seems [inherent to the VAE decoder stage.](https://github.com/comfyanonymous/ComfyUI/issues/1116)

![loss](https://github.com/city96/SD-Latent-Interposer/assets/125218114/e890420f-cebd-4f88-b243-62560b8384e5)

</details>

<details><summary>Interposer v1.0</summary>

### Interposer v1.0 
Not sure why the training loss is so different, it might be due to the """highly curated""" dataset of 1000 random images from my Downloads folder that I used to train it.

I probably should've just grabbed LAION.

I also trained a v1-to-v2 mode, before realizing v1 and v2 shared the same latent space. Oh well.

![loss](https://github.com/city96/SD-Latent-Interposer/assets/125218114/f92c399b-a823-4521-b09b-8bdc3795f1ea)
  
![xl-to-v1_interposer](https://github.com/city96/SD-Latent-Interposer/assets/125218114/0d963bc5-570f-4ebe-95db-16e261f05e48)
  
</details>

</details>
