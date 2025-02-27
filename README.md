# ComfyUI IPAdapter plus
[ComfyUI](https://github.com/comfyanonymous/ComfyUI) reference implementation for [IPAdapter](https://github.com/tencent-ailab/IP-Adapter/) models.

IPAdapter implementation that follows the ComfyUI way of doing things. The code is memory efficient, fast, and shouldn't break with Comfy updates.

## Important updates

**2023/11/08**: Added attention masking. 

**2023/11/07**: Added three ways to apply the weight. [See below](#weight-types) for more info. **This might break things!** Please let me know if you are having issues. When loading an old workflow try to reload the page a couple of times or delete the `IPAdapter Apply` node and insert a new one.

**2023/11/02**: Added compatibility with the new models in safetensors format (available on [huggingface](https://huggingface.co/h94/IP-Adapter)).

**2023/10/12**: Added image weighting in the `IPAdapterEncoder` node. This update is somewhat breaking; if you use `IPAdapterEncoder` and `PrepImageForClipVision` nodes you need to remove them from your workflow, refresh and recreate them. In the examples you'll find a [workflow](examples/IPAdapter_weighted.json) for weighted images.

**2023/9/29**: Added save/load of encoded images. Fix minor bugs.

*(previous updates removed for better readability)*

## What is it?

The IPAdapter are very powerful models for image-to-image conditioning. Given a reference image you can do variations augmented by text prompt, controlnets and masks. Think of it as a 1-image lora.

## Example workflow

![IPAdapter Example workflow](./ipadapter_workflow.png)

## Video Introduction

<a href="https://youtu.be/7m9ZZFU3HWo" target="_blank">
 <img src="https://img.youtube.com/vi/7m9ZZFU3HWo/hqdefault.jpg" alt="Watch the video" />
</a>

**:nerd_face: [Basic usage video](https://youtu.be/7m9ZZFU3HWo)**

**:rocket: [Advanced features video](https://www.youtube.com/watch?v=mJQ62ly7jrg)**

## Installation

Download or git clone this repository inside `ComfyUI/custom_nodes/` directory.

The pre-trained models are available on [huggingface](https://huggingface.co/h94/IP-Adapter), download and place them in the `ComfyUI/custom_nodes/ComfyUI_IPAdapter_plus/models` directory.

For SD1.5 you need:

- [ip-adapter_sd15.bin](https://huggingface.co/h94/IP-Adapter/resolve/main/models/ip-adapter_sd15.bin)
- [ip-adapter_sd15_light.bin](https://huggingface.co/h94/IP-Adapter/blob/main/models/ip-adapter_sd15_light.bin), use this when text prompt is more important than reference images
- [ip-adapter-plus_sd15.bin](https://huggingface.co/h94/IP-Adapter/resolve/main/models/ip-adapter-plus_sd15.bin)
- [ip-adapter-plus-face_sd15.bin](https://huggingface.co/h94/IP-Adapter/resolve/main/models/ip-adapter-plus-face_sd15.bin)

For SDXL you need:
- [ip-adapter_sdxl.bin](https://huggingface.co/h94/IP-Adapter/resolve/main/sdxl_models/ip-adapter_sdxl.bin)
- [ip-adapter_sdxl_vit-h.bin](https://huggingface.co/h94/IP-Adapter/resolve/main/sdxl_models/ip-adapter_sdxl_vit-h.bin) **This model requires the use of the SD1.5 encoder despite being for SDXL checkpoints**
- [ip-adapter-plus_sdxl_vit-h.bin](https://huggingface.co/h94/IP-Adapter/resolve/main/sdxl_models/ip-adapter-plus_sdxl_vit-h.bin) Same as above, use the SD1.5 encoder
- [ip-adapter-plus-face_sdxl_vit-h.bin](https://huggingface.co/h94/IP-Adapter/resolve/main/sdxl_models/ip-adapter-plus-face_sdxl_vit-h.bin) As always, use the SD1.5 encoder

Please note that now the models are also available in safetensors format, you can find them on [huggingface](https://huggingface.co/h94/IP-Adapter).

Additionally you need the image encoders to be placed in the `ComfyUI/models/clip_vision/` directory:

- [SD 1.5 model](https://huggingface.co/h94/IP-Adapter/resolve/main/models/image_encoder/model.safetensors) (use this also for all models ending with **_vit-h**)
- [SDXL model](https://huggingface.co/h94/IP-Adapter/resolve/main/sdxl_models/image_encoder/model.safetensors)

You can rename them to something easier to remember or put them into a sub-directory.

**Note:** the image encoders are actually [ViT-H](https://huggingface.co/laion/CLIP-ViT-H-14-laion2B-s32B-b79K) and [ViT-bigG](https://huggingface.co/laion/CLIP-ViT-bigG-14-laion2B-39B-b160k) (used only for one SDXL model). You probably already have them.

## How to

There's a basic workflow included in this repo and a few examples in the [examples](./examples/) directory. Usually it's a good idea to lower the `weight` to at least `0.8`.

The `noise` paramenter is an experimental exploitation of the IPAdapter models. You can set it as low as `0.01` for an arguably better result.

<details>
<summary><strong>More info about the noise option</strong></summary>

<img src="./examples/noise_example.jpg" width="100%" alt="canny controlnet" />

Basically the IPAdapter sends two pictures for the conditioning, one is the reference the other --that you don't see-- is an empty image that could be considered like a negative conditioning.

What I'm doing is to send a very noisy image instead of an empty one. The `noise` parameter determines the amount of noise that is added. A value of `0.01` adds a lot of noise (more noise == less impact becaue the model doesn't get it); a value of `1.0` removes most of noise so the generated image gets conditioned more.
</details>

### IMPORTANT: Preparing the reference image

The reference image needs to be encoded by the CLIP vision model. The encoder resizes the image to 224×224 **and crops it to the center!**. It's not an IPAdapter thing, it's how the clip vision works. This means that if you use a portrait or landscape image and the main attention (eg: the face of a character) is not in the middle you'll likely get undesired results. Use square pictures as reference for more predictable results.

I've added a `PrepImageForClipVision` node that does all the required operations for you. You just have to select the crop position (top/left/center/etc...) and a sharpening amount if you want.

In the image below you can see the difference between prepped and not prepped images.

<img src="./examples/prep_images.jpg" width="100%" alt="prepped images" />

### KSampler configuration suggestions

The IPAdapter generally requires a few more `steps` than usual, if the result is underwhelming try to add 10+ steps. `ddmin`, `ddpm` and `euler` seem to perform better than others.

The model tends to burn the images a little. If needed lower the CFG scale.

The SDXL models are weird but the `noise` option sometimes helps.

### IPAdapter + ControlNet

The model is very effective when paired with a ControlNet. In the example below I experimented with Canny. [The workflow](./examples/IPAdapter_Canny.json) is in the examples directory.

<img src="./examples/canny_controlnet.jpg" width="100%" alt="canny controlnet" />

### IPAdapter Face

IPAdapter offers an interesting model for a kind of "face swap" effect. [The workflow is provided](./examples/IPAdapter_face.json). Set a close up face as reference image and then input your text prompt as always. The generated character should have the face of the reference. It also works with img2img given a high denoise.

<img src="./examples/face_swap.jpg" width="50%" alt="face swap" />

### Masking

The most effective way to apply the IPAdapter to a region is by an [inpainting workflow](./examples/IPAdapter_inpaint.json). Remeber to use a specific checkpoint for inpainting otherwise it won't work. Even if you are inpainting a face I find that the *IPAdapter-Plus* (not the *face* one), works best.

<img src="./examples/inpainting.jpg" width="100%" alt="inpainting" />

### Image Batches

It is possible to pass multiple images for the conditioning with the `Batch Images` node. An [example workflow](./examples/IPAdapter_batch_images.json) is provided; in the picture below you can see the result of one and two images conditioning.

<img src="./examples/batch_images.jpg" width="100%" alt="batcg images" />

It seems to be effective with 2-3 images, beyond that it tends to *blur* the information too much.

### Image Weighting

When sending multiple images you can increase/decrease the weight of each image by using the `IPAdapterEncoder` node. The workflow ([included in the examples](examples/IPAdapter_weighted.json)) looks like this:

<img src="./examples/image_weighting.jpg" width="100%" alt="image weighting" />

The node accepts 4 images, but remember that you can send batches of images to each slot.

### Weight types

You can choose how the IPAdapter weight is applied to the image embeds. Options are:

- **original**: The weight is applied to the aggregated tensors. The weight works predictably for values greater and lower than 1.
- **linear**: The weight is applied to the individual tensors before aggretating them. Compared to `original` the influence is weaker when weight is <1 and stronger when >1. **Note:** at weight `1` the two methods are equivalent.
- **channel penalty**: This method is a modified version of Lvmin Zhang's (Fooocus). Results are sometimes sharper. It works very well also when weight is >1. Still experimental, may change in the future.

The image below shows the difference (zoom in).

<img src="./examples/weight_types.jpg" width="100%" alt="weight types" />

In the examples directory you can find [a workflow](examples/IPAdapter_weight_types.json) that lets you easily compare the three methods.

**Note:** I'm not still sure whether all methods will stay. `Linear` seems the most sensible but I wanted to keep the `original` for backward compatibility. `channel penalty` has a weird non-commercial clause but it's still part of a GNU GPLv3 software (ie: there's a licensing clash) so I'm trying to understand how to deal with that.

### Attention masking

It's possible to add a mask to define the area where the IPAdapter will be applied to. Everything outside the mask will ignore the reference images and will only listen to the text prompt.

It is suggested to use a mask of the same size of the final generated image.

In the picture below I use two reference images masked one on the left and the other on the right. The image is generated only with IPAdapter and one ksampler (without in/outpainting or area conditioning).

<img src="./examples/masking.jpg" width="512" alt="masking" />

In the examples directory you'll find a couple of masking workflows: [simple](examples/IPAdapter_mask.json) and [two masks](examples/IPAdapter_2_masks.json).

## Troubleshooting

**Error: 'CLIPVisionModelOutput' object has no attribute 'penultimate_hidden_states'**

You are using an old version of ComfyUI. Update and you'll be fine. **Please note** that on Windows for a full update you might need to re-download the latest standalone version.

**Error with Tensor size mismatch**

You are using the wrong CLIP encoder+IPAdapter Model+Checkpoint combo. Remember that you need to select the CLIP encoder v1.5 for all v1.5 IPAdapter models AND for all models ending with `vit-h` (even if they are for SDXL).

**Is it true that the input reference image must have the same size of the output image?**

No, that's a metropolitan legend. Your input and output images can be of any size. Remember that all input images are scaled and cropped to 224x224 anyway.

## Diffusers version

If you are interested I've also implemented the same features for [Huggingface Diffusers](https://github.com/cubiq/Diffusers_IPAdapter).

## Credits

- [IPAdapter](https://github.com/tencent-ailab/IP-Adapter/)
- [ComfyUI](https://github.com/comfyanonymous/ComfyUI)
- [laksjdjf](https://github.com/laksjdjf/IPAdapter-ComfyUI/)
- [fooocus](https://github.com/lllyasviel/Fooocus/blob/main/fooocus_extras/ip_adapter.py)

## IPAdapter in the wild

Let me know if you spot the IPAdapter in the wild!

- For German speakers you can find interesting YouTube tutorials on [A Latent Place](https://www.youtube.com/watch?v=rAWn_0YOBU0).
- [Scott Detweiler](https://www.youtube.com/watch?v=xzGdynQDzsM) covered this extension.
