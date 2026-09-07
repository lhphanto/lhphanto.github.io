---
title: "Projects"
permalink: /projects/
layout: single
author_profile: true
classes: wide
---

## diffusion_llm

A from-scratch implementation of a **masked diffusion language model** — text generation
by iterative denoising over discrete tokens rather than left-to-right autoregression.

Trained on [TinyStories](https://huggingface.co/datasets/roneneldan/TinyStories) with
DeepSpeed; an example checkpoint is on the
[Hugging Face Hub](https://huggingface.co/lhphanto/diffusion_llm/tree/main).

The `multimodal/` extension pairs image and text in a single joint diffusion model,
using the generative process suited to each state space — **flow matching** for images
(continuous) and a **masked continuous-time Markov chain** for text (discrete) — driven
by one shared transformer on a single shared timestep. It supports text→image,
image→text, and unconditional joint sampling.

The construction follows Holderrieth et al., ["Generator Matching: Generative Modeling
with Arbitrary Markov Processes"](https://arxiv.org/abs/2410.20587) (2024), specifically
the product-space construction for combining modalities over different state spaces.

[GitHub](https://github.com/lhphanto/diffusion_llm) ·
[Checkpoint](https://huggingface.co/lhphanto/diffusion_llm/tree/main)

<!-- Add further projects as `##` sections below. -->
