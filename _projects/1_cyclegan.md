---
layout: page
title: Cartoon-to-Realistic Image Translation using CycleGAN
description: Unpaired image-to-image translation pipeline in PyTorch for photorealistic synthesis from artistic cartoons.
importance: 1
category: machine learning & ai
related_publications: false
---

### Overview

Translating stylized artistic cartoons into photorealistic natural images is an ill-posed vision challenge due to the lack of paired training samples: an artist's drawing rarely corresponds pixel-for-pixel to a real-world photograph.

In this project, I developed an end-to-end unpaired image-to-image translation pipeline leveraging **Cycle-Consistent Generative Adversarial Networks (CycleGAN)** implemented in **PyTorch**. The system learns bidirectional mappings between the source domain $\mathcal{X}$ (cartoons/drawings) and the target domain $\mathcal{Y}$ (photorealistic images) without requiring paired input-output examples.

---

### Architecture & Methodology

The architecture comprises two mirrored generator-discriminator pairs:
- **$G: \mathcal{X} \to \mathcal{Y}$**: Translates cartoon representations into photorealistic scenes.
- **$F: \mathcal{Y} \to \mathcal{X}$**: Inverts real images back into cartoon stylizations.
- **$D_{\mathcal{Y}}$**: Discriminates generated realistic images from actual natural photographs.
- **$D_{\mathcal{X}}$**: Discriminates synthesized cartoons from authentic cartoon artwork.

#### Generator Architecture
The generators utilize an encoder-bottleneck-decoder design:
1. **Downsampling / Feature Extraction**: Strided convolutions with Instance Normalization and ReLU activations to capture multi-scale spatial semantics.
2. **Residual Bottleneck**: A sequence of 9 ResNet residual blocks ensuring feature retention without gradient degradation.
3. **Upsampling / Decoding**: Fractional-strided transpose convolutions restoring native resolution with continuous visual textures.

#### PatchGAN Discriminators
Rather than assessing global image fidelity with a single scalar output, 70x70 **PatchGAN** discriminators are employed. This evaluates localized image patches for photorealism, penalizing high-frequency spatial artifacts while promoting crisp textural reconstruction.

---

### Objective Formulation

The joint objective balances three complementary loss objectives:

1. **Adversarial Loss**: Encourages distribution matching across the domains:
   $$\mathcal{L}_{GAN}(G, D_{\mathcal{Y}}, \mathcal{X}, \mathcal{Y}) = \mathbb{E}_{y \sim p_{data}(y)}[\log D_{\mathcal{Y}}(y)] + \mathbb{E}_{x \sim p_{data}(x)}[\log (1 - D_{\mathcal{Y}}(G(x)))]$$

2. **Cycle Consistency Loss**: Prevents mode collapse and ensures semantic invertibility ($F(G(x)) \approx x$ and $G(F(y)) \approx y$):
   $$\mathcal{L}_{cyc}(G, F) = \mathbb{E}_{x \sim p_{data}(x)}[\|F(G(x)) - x\|_1] + \mathbb{E}_{y \sim p_{data}(y)}[\|G(F(y)) - y\|_1]$$

3. **Identity Mapping Loss**: Preserves color histograms and localized composition:
   $$\mathcal{L}_{identity}(G, F) = \mathbb{E}_{y \sim p_{data}(y)}[\|G(y) - y\|_1] + \mathbb{E}_{x \sim p_{data}(x)}[\|F(x) - x\|_1]$$

Total loss:
$$\mathcal{L}_{total} = \mathcal{L}_{GAN}(G, D_{\mathcal{Y}}) + \mathcal{L}_{GAN}(F, D_{\mathcal{X}}) + \lambda_{cyc} \mathcal{L}_{cyc}(G, F) + \lambda_{id} \mathcal{L}_{identity}(G, F)$$

---

### Implementation Details & Key Learnings

- **Data Pipeline**: Formulated an unpaired dataset loader supporting non-aligned image augmentations (random cropping, horizontal flips, photometric jittering).
- **Training Dynamics**: Stabilized GAN convergence using an image replay buffer of historical generated samples, least-squares loss formulation (LSGAN), and the Adam optimizer with linear learning rate decay.
- **Evaluation**: Assessed visual fidelity, structural retention, and color consistency across test sets, studying edge boundary preservation under substantial domain shifts.
