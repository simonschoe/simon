---
date: 2022-12-30
title: The DreamBooth Technique
subtitle: Fine-Tunig Technique for Personalized Text-to-Image Models
summary: |
  DreamBooth is a fine-tuning technique for large, pretrained text-to-image models (e.g., DALL-E2, Imagen, Stable Diffusion).
  Based on a small reference set of training images of a given subject or object (henceforth *concept*), the DreamBooth technique learns a custom identifier for the given concept and implants the concept embedding into the model's output domain.
  This enables the model to synthesize images of the underlying concept in different contexts and settings with very high-quality.
slug: dreambooth-technique
categories:
- TIL
---

## DreamBooth

DreamBooth is a few-shot personalization technique for fine-tuning large, pretrained text-to-image models (e.g., DALL-E2, Imagen, Stable Diffusion).
Based on a small reference set of training images of a given subject or object (henceforth *concept*), the DreamBooth technique learns a custom identifier for the given concept and implants the concept embedding into the model's output domain.
This enables the model to synthesize images of the underlying concept in different contexts and settings with very high-quality.


####   Key Components

**Fine-tuning dataset:** Limited set of fine-tuning images depicting the concept of interest. The concept images should contain variations in terms of pose, background, angle, etc. to promote learning of the concept representation. Text prompt corresponding to these images should take the form: "A photo of a [concept identifier] [concept class]". The reference to the concept class ensures that the model can leverage prior knowledge about the general concept class.

**Concept identifier:** Encoding of the concept via a *rare-token identifier* in the text encoder's vocabulary (e.g., `ðŁĴŁ` for a CLIP text encoder). This token representation is overriden during fine-tuning and acts as a unique concept identifier.

**Loss function:** Combination of reconstruction loss to learn new concept and **prior preservation** loss to prevent overfitting and *language drift* (i.e., ability to generate images of other concepts of the same class)


## DreamBooth Hackathon by Huggingface

I fine-tuned two personalized DreamBooth models for the [Huggingface DreamBooth Hackathon](https://github.com/huggingface/diffusion-models-class/tree/main/hackathon).

#### **[The Pokéball Machine](https://huggingface.co/simonschoe/pokeball-machine)**

**Model checkpoint:** `CompVis/stable-diffusion-v1-4`  
**Training images:** Images of the original, red-and-white Pokéball  
**Rare-token identifier:** `pkblz`

|   |   |   |
|---|---|---|
|![](pokeball/pokeball%20(1).png)|![](pokeball/pokeball%20(2).png)|![](pokeball/pokeball%20(3).png)|
|![](pokeball/pokeball%20(4).png)|![](pokeball/pokeball%20(5).png)|![](pokeball/pokeball%20(6).png)|
|![](pokeball/pokeball%20(7).png)|![](pokeball/pokeball%20(8).png)|![](pokeball/pokeball%20(9).png)|


#### **[Iridescent Jellyfish](https://huggingface.co/simonschoe/iridescent-jellyfish)**

**Model checkpoint:** `runwayml/stable-diffusion-v1-5`  
**Training images:** Images of fluorescent jellyfishes  
**Rare-token identifier:** `ðŁĴŁ`

|   |   |   |
|---|---|---|
|![](jellyfish/jelly%20(1).jpg)|![](jellyfish/jelly%20(2).jpg)|![](jellyfish/jelly%20(3).jpg)|
|![](jellyfish/jelly%20(4).jpg)|![](jellyfish/jelly%20(5).jpg)|![](jellyfish/jelly%20(6).jpg)|
|![](jellyfish/jelly%20(7).png)|![](jellyfish/jelly%20(8).png)|![](jellyfish/jelly%20(9).png)|

#### Experiment Log

- Input images with same resolution as model checkpoint (e.g., `512x512`) enhances image augmentations and improves performance.
- Use of rare-token identifier that is so rare that is has not yet embedded an own unique concept (e.g., `pkblz` versus `pokeeee` where the latter seems to encode a visual prior that relates to Japanese/Asian culture).
- Choice of relatively low learning rate (`2e-06`) and relatively larger number of steps (`800`).
- At inference time, a lower guidance scale (e.g., `7`) allows more creativity while a larger guidance scale (e.g., `11`) ensures that the model adheres to the text prompt more strongly.
- TODO: more heterogeneity in text prompts
- TODO: human images

## References

Ruiz, N., Li, Y., Jampani, V., Pritch, Y., Rubinstein, M., & Aberman, K. (2022). Dreambooth: Fine tuning text-to-image diffusion models for subject-driven generation. arXiv preprint arXiv:2208.12242.
