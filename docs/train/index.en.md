# Alchemy/Train

>Todo
>材料
>
>https://github.com/huggingface/diffusers/issues/712
>
>https://github.com/AUTOMATIC1111/stable-diffusion-webui/pull/2002
>
>https://github.com/ShivamShrirao/diffusers/tree/main/examples/dreambooth


This is more demanding on the graphics card.

**Should not** disregard the copyright permission agreed by the artist to collect data on the work for training.

**Should not** use the training results to devalue the original artist's work.

At the moment I personally think the results are considered the best by DreamBooth, but its training requires high VRAM (>8GB).


## Cognitive model training

If you start training with the `--medvram` argument, you may get a `RuntimeError: Expected all tensors to be on the same device` error and be unable to create training.
This is a [problem](https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/2399) caused by the optimization mechanism, WebUi in [this commit](https://github.com/AUTOMATIC1111/) stable-diffusion-webui/commit/cbb857b675cf0f169b21515c29da492b513cc8c4) allows for the creation of embedding under `--medvram`. Please update your version beyond this one.

**About batch size**

Larger batch sizes may speed up training slightly and improve it slightly, but they also require larger VRAM.




**About batch size**

A larger batch size may speed up training slightly and improve it slightly, but it also requires a larger amount of VRAM.

### Pickle?

[About pickle](https://docs.python.org/3/library/pickle.html),that Python object serialization

Docs says:
```
The pickle module is not secure. Only unpickle data you trust.

It is possible to construct malicious pickle data which will execute arbitrary code during unpickling. Never unpickle data that could have come from an untrusted source, or that could have been tampered with.
```

Both `.ckpt` and `.pt` files may have malicious pickles in them, AUTOMATIC1111 has done some verification in this regard.

For insecure files, it will prompt 

```
The file may be malicious, so the program is not going to read it. 
You can skip this check with the --disable-safe-unpickle commandline argument.
```


It is not always guaranteed to be 100% secure, and some files may even be false positive, so it is recommended to download the model from a trusted source.

In addition, you can see in [here] (https://github.com/sudoskys/StableDiffusionBook/blob/main/docs/ckpt_safe) is a simple check script.


### Textual Inversion (TI)

A method for extracting [v] from a number of images that have a common semantic [v]. The extracted [v] tensor is called "Embedding". The Embedding is saved as a file and can then be referenced by its filename in the prompt when the image is generated.


#### features

Smaller training product size, webui comes with training support.

Can be used to solve problems with newly created characters that cannot be drawn, or to imitate a specific art style that can be described precisely in words. Because TI is doing the processing on the output of the Text Encoder, it does not allow the model to learn concepts it does not know.


#### Usage

To use, put the embedding (a .pt or a .bin file) into webui's `embeddings` directory and write the name of the embedding to be used (without the extension) in prompt, without restarting webui.


#### is related

[English description and effects given by webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Textual-Inversion)

[Related embeddings](https://gitlab.com/16777216c/stable-diffusion-embeddings) with a preview of the related effects.

[list of Textual Inversion embeddings for webui(SD)](https://rentry.org/embeddings)

[embedding library for HuggingFace](https://cyberes.github.io/stable-diffusion-textual-inversion-models/)


### Hypernetwork (HN)

A class of network that generates weights for models, in this case for LDMs (Latent Diffusion Models). It is a more experimental approach that NAI has pioneered and explored for use on LDMs.


#### Features

Unlike TI, Hypernetwork modifies the weights of the LDM itself, so it can be trained in details that cannot be expressed precisely in words, and is more suitable for training in drawing styles.

The training product size is medium and webui comes with training support.


#### Usage

To use, put Pt into `/models/hypernetworks` and check Enable it in the Settings option.

There are some Hypernetworks trained by NAI in `novelaileak\stableckpt\modules\modules` of NAI Leaks.

!!! tip
    `.pt` files, in general the small ones are embedding and the big ones are hypernetwork.


### DreamBooth (DB)

A way to directly fine-tune the LDM and Text Encoder to suit the user's specific image generation needs.

> Can you imagine your own dog travelling around the world, or your favourite bag on display in the most exclusive showroom in Paris? How about your parrot becoming the protagonist of an illustrated storybook?


#### Features

Unlike TI and HN, DreamBooth can produce images that are highly similar to the training set without losing the ability to generalise, which is particularly good for training specific figurative concepts (e.g. a character wearing a specific outfit). However, unlike TI and HN, which are plug-and-play "plug-ins" with full weights, the strength is not adjustable.

The model is not designed for learning drawing styles (abstract concepts). But it seems to be adaptable to a certain extent to the "drawing style". I leave it to the reader to experiment with the results.


#### Use

Put the DreamBooth trained .ckpt file into the `models\Stable-diffusion` directory of webui and switch to it in the top left corner of webui to use it.

![SAMPLE](https://dreambooth.github.io/DreamBooth_files/high_level.png)

Official website https://dreambooth.github.io/

Papers https://arxiv.org/abs/2208.12242


### Advanced Prompt Tuning (APT)

> "Can super dramatically improve the image quality and diversity"

[Pull Here](https://github.com/AUTOMATIC1111/stable-diffusion-webui/pull/2945)

Adds on-the-fly embedding learning of negatives to dramatically improve the quality of the resulting images. High quality concepts can be learned from individual images.

Add reconstruction loss to improve the detail quality and richness of the generated images.

Adding a discriminator trained by manual annotation (implemented using convnext) allows embedding to be learnt based on the model.


### Aesthetic Gradients

A method for fine-tuning CLIP to suit a particular generation requirement, which can have the same effect of shortening the prompt as TI. The quality of the output may be slightly improved.

This technique provides an ``I don't say you should know where to train'' feature by calculating the weight of each image at generation time. Enables the AI to adjust and add detail more intelligently.

This feature comes from this [repository](https://github.com/vicgalle/stable-diffusion-aesthetic-gradients) and is available in [this commit](https://github.com/AUTOMATIC1111/stable-) diffusion-webui/commit/2b91251637078e04472c91a06a8d9c4db9c1dcf0), this feature was stripped down to a plugin.


#### Features

With this technique, you do not need to enhance the quality of the image with too many cues, but rather maintain the original general composition of the work and improve the aesthetics. It is possible to produce good results with a small number of cues.

According to Shadow-Light [^11], adding less than 25% weighting can slightly improve the aesthetics of the image without affecting the content. Aesthetics and Hypernetworks bring the Ai work closer to the original artist's style, but the aesthetic weighting itself does not work well. It needs to be combined with the Hypernetworks supernetwork.

Training this model is quick, but it is slow to produce images as it is recalculated once for each production run.

Note: When the seed changes, the training results will also change.


#### Usage

You can use the following Git command to install this.

```commandline
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui-aesthetic-gradients extensions/aesthetic-gradients
```

Once installed, create the ``aesthetic-gradients` folder under the ``extensions` folder in webui.

To use it, put Pt in `models/aesthetic_embeddings/`

Then restart the application and you can use this feature in Img2Img.

![Aesthetic_other](https://user-images.githubusercontent.com/75739606/197824140-50dca98e-856f-44ca-99e6-da2fee5bb23c.png)
<!--
![Aesthetic_other](https://raw.githubusercontent.com/sudoskys/StableDiffusionBook/main/resource/Aesthetic_other.png)
-->


[^7]:[风格模型训练](https://www.bilibili.com/video/BV1ae4y1S7v9/)

[^8]:[迭代草图](https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/2473)

[^9]:[交替单词](https://github.com/AUTOMATIC1111/stable-diffusion-webui/pull/1733)

[^10]:[角色与画风tag训练十问](https://www.bilibili.com/video/BV1xt4y1F7Y2/)

[^11]:[WebUI即将引入重磅更新，大幅提升图像品质](https://www.bilibili.com/read/cv19102552)

[^12]:[超网络训练](https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/2284)

[^13]:[超网络训练指南](https://rentry.org/hypernetwork4dumdums)


[^14]:[dreambooth-小鬼峰](https://www.bilibili.com/read/cv18935031?from=search)

[^15]:[dreambooth-论文](https://arxiv.org/abs/2208.12242)

[^16]:[Hypernetwork画风训练经验分享](https://shiina-h.notion.site/shiina-h/Hypernetwork-1fc0b60645284c5e96bb41b583a4e86f)

[^17]:[Installing Stable Diffusion & Dreambooth for Windows](https://pastebin.com/xcFpp9Mr)

[^18]:[Installing Dreambooth for Windows](https://gist.github.com/Summersoff/70861d757a40c153c5802dc8c4ed68c0)

