---
title: "RoboMonkey: Scaling Test-Time Sampling and Verification for Vision-Language-Action Models"
authors:
  - key: jackykwok
    affiliation: Stanford University
  - name: Christopher Agia
    affiliation: Stanford University
  - name: Rohan Sinha
    affiliation: Stanford University
  - name: Matt Foutter
    affiliation: Stanford University
  - name: Shulu Li
    affiliation: UC Berkeley
  - key: ionstoica
    affiliation: UC Berkeley
  - key: azaliamirhoseini
    affiliation: Stanford University
  - key: marcopavone
    affiliation: Stanford, NVIDIA
venue: corl
year: 2025
date: 2025-06-21
has_pdf: true
doi: 10.48550/arXiv.2506.17811
tags:
  - robotics
  - machine learning
  - generative ai
  - scaling laws
teaser: RoboMonkey is a test-time scaling framework that improves the robustness and generalization of Vision-Language-Action (VLA) models. RoboMonkey achieves significant performance improvements across both in-distribution and out-of-distribution tasks, as well as on new robot setups. Our findings show that scaling test-time compute through a generate-then-verify paradigm provides a practical and effective path towards building general-purpose robotics foundation models.
materials:
  - name: Paper
    url: https://arxiv.org/abs/2506.17811
    type: file-pdf
  - name: Codebase
    url: https://github.com/robomonkey-vla/RoboMonkey
    type: code
  - name: Datasets and Models
    url: https://huggingface.co/robomonkey-vla
    type: database
  - name: Serving Engine
    url: https://github.com/robomonkey-vla/sglang-vla
    type: code
---

Vision-Language-Action (VLA) models have demonstrated remarkable capabilities in visuomotor control, yet ensuring their robustness in unstructured real-world environments remains a persistent challenge. In this paper, we investigate test-time scaling through the lens of sampling and verification as means to enhance the robustness and generalization of VLAs. We first demonstrate that the relationship between action error and the number of generated samples follows an exponentiated power law across a range of VLAs, indicating the existence of inference-time scaling laws. Building on these insights, we introduce RoboMonkey, a test-time scaling framework for VLAs. At deployment, RoboMonkey samples a small set of actions from a VLA, applies Gaussian perturbation and majority voting to construct an action proposal distribution, and then uses a Vision Language Model (VLM)-based verifier to select the optimal action. We propose a synthetic data generation pipeline for training such VLM-based action verifiers, and demonstrate that scaling the synthetic dataset consistently improves verification and downstream accuracy. Through extensive simulated and hardware experiments, we show that pairing existing VLAs with RoboMonkey yields significant performance gains, achieving a 25% absolute improvement on out-of-distribution tasks and 9% on in-distribution tasks. Additionally, when adapting to new robot setups, we show that fine-tuning both VLAs and action verifiers yields a 7% performance increase compared to fine-tuning VLAs alone.
