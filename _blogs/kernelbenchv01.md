---
title: "KernelBench v0.1"
authors:
  - key: nataliakokoromyti
  - key: soksereysun
  - key: anneouyang
  - key: azaliamirhoseini
tags:
  - ml systems
  - gpu programming
  - generative ai
venue: none
year: 2025
date: 2025-07-27
teaser: KernelBench v0.1 is out
materials:
  - name: KernelBench
    url: https://github.com/ScalingIntelligence/KernelBench
    type: code
---

# KernelBench v0.1

# TL;DR: KernelBench v0.1 is out

What's new:

* We provide a guideline that makes it easier to analyze the validity of results and quickly rule out physically impossible performance claims.  
* Support for randomized testing beyond normal distributions.   
* Fixed problem sizes and improved numerics to avoid disproportionately large kernel launch overhead time and trivial kernels. 

Since the initial v0 release, the benchmark has been used in a variety of promising ways to study and improve LLMs' ability to write kernel code. 

As always, thank you to the community –– including teams from [Cognition AI](https://arxiv.org/abs/2507.11948), [Meta](https://huggingface.co/facebook/KernelLLM), [METR](https://metr.org/blog/2025-02-14-measuring-automated-kernel-engineering/), [Nvidia](https://developer.nvidia.com/blog/automating-gpu-kernel-generation-with-deepseek-r1-and-inference-time-scaling/), [Prime Intellect](https://x.com/MatternJustus/status/1931221482153333019), [SakanaAI](https://sakana.ai/ai-cuda-engineer/), and many others whose work we may not yet be aware of  –– for working on the benchmark and helping surface both its strengths and limitations. Your feedback directly informed many of the improvements in v0.1 and we’re excited to see what you’ll build with the new version. Hopefully, v0.1 pushes these efforts even further and inspires many more to come\!

In what follows, we walk through the key changes behind KernelBench v0.1. We start by outlining the most common reward hacking patterns we’ve observed, along with a guideline for checking whether performance claims make actual sense. We then describe the changes we made to improve task quality and testing robustness.

# Reward Hacking

So far here are some examples of common reward hacking patterns we've seen:

* **Not generating real CUDA code**: The model skips writing raw CUDA kernels and instead calls high-level operators like torch or cuBLAS, which pass correctness checks but sidestep the challenge entirely.  
* **Generating real CUDA code but not calling it:** The model writes CUDA kernels but never calls it and calls a simpler naive implementation or a higher level operator instead  
* **No-op kernels with memory reuse**: The generated code is a no-op. Since the evaluation code reads the output from the same memory region as the reference implementation, the code appears correct even though it performs no actual computation.  
* **Timing synchronization issues**: Incorrect synchronization points setup for timing, or computing runs on one CUDA stream while timing is done on another, creating artificially short runtimes that mislead latency-based reward metrics  
* ...

Unfortunately, models are exceptionally good at finding edge cases that technically satisfy the reward function while bypassing the real goal. Even when we patch known exploits, new ones emerge, often even harder to detect. Designing robust, adversary-resistant reward signals remains an open problem. As a result, even with all available automated checks, we strongly recommend carefully inspecting any outputs that achieve high scores on your reward metrics.

# Sanity Checking Kernels Against Theoretical Hardware Limits

In addition to the common reward hacks we look for, another important way to sanity check whether your generated CUDA kernel is doing "real work" is to compare its achieved performance to theoretical hardware limits. We can estimate the upper bound on a kernel's performance by considering both compute and memory throughput limits of the operation, and then take the maximum of the two.

Here's an example of estimating the theoretical limit (speed of light performance) for multiplying two BF16 4096×4096 matrices on an Nvidia H100 SXM5 GPU.

* The compute requires \~137 GFLOPs (2 \* N^3)  
* The memory traffic is \~96MB (assuming each of the input matrices A and B is read once and the output matrix C is written once to global memory (HBM). For BF16 (2 bytes per element): 3 \* N^2 \* 2\)  
* Based on the [H100 spec sheet](https://www.megware.com/fileadmin/user_upload/LandingPage%20NVIDIA/nvidia-h100-datasheet.pdf), the H100 SXM5 GPU has a peak compute throughput of \~990 TFLOPs for BF16 tensor core operations, and a peak HBM bandwidth of \~3.35 TB/s  
* This means compute time should be at least \~0.138 milliseconds (137 GFLOPs / 990 TFLOPs), and memory transfer time should be at least 0.029 milliseconds (96MB / 3.35TB). Taking the max, if your kernel is faster than the speed of light of 0.138 ms, something is wrong.

Note: this is a very conservative estimate, it assumes perfect utilization of tensor cores, no launch overhead, perfect data reuse, and only HBM memory transfers take time in the memory hierarchy. In practice, most well-optimized kernels will take longer. So if you're approaching this limit exactly, that may itself be a sign to dig deeper. As you'll see later in this blog post, we’ve added a new automated testing mechanism that increases our ability to flag suspicious cases like that in practice.

# Problem Fixes

On an H100, we found dozens of Level‑1/2 problems executed in the microsecond range (we would like to thank Tri Dao for identifying this). At this level, kernel launch overhead can dominate the total measured time. This masks the real performance differences between an optimized kernel and a naive one. The results are speedup numbers that don't accurately reflect a kernel's quality.

To fix this, we scaled up the tensor dimensions and batch sizes across level 1 and 2\. We also made sure to scale down problem sizes runtimes larger than 15ms. Our goal was to standardize every test in level 1 and 2 into an ideal 1-to-15 millisecond window on an H100. This timing range we hope will be a sweet spot:  Long enough to make launch overhead negligible, yet short enough to keep the entire suite fast and practical. This ensures we are measuring true kernel performance without wasting developer time or expensive compute cycles.

A few example problems for reference:

Square Matrix Multiplication: 0.347 ms → 3.81 ms (\~11x longer)  
Batched Matrix Multiplication: 0.104 ms → 8.47 ms (\~81x longer)  
ReLU Activation: 0.013 ms → 6.97 ms (\~530x longer)  
Sum Reduction: 0.0148 ms → 5.41 ms (\~365x longer)

This change makes Kernel-Bench far more informative. It’s now easier to:

* Clearly distinguish a highly-optimized, tensor-core-aware kernel from a basic implementation.  
* Properly analyze performance limiters like memory bandwidth and SM occupancy, which only become visible with sustained workloads.  
* Provide a clean, reliable performance signal for automated systems (like RL-based compilers) that learn from runtime data.

When METR re‑evaluated KernelBench earlier this year, their quality filters removed forty‑three Level 1–3 tasks. The criteria included outputs sitting in the  10^-2 noise floor, variance that barely changed across random seeds, tensors that were uniform along whole axes, algebraic reductions to constants such as mean(softmax(x)), and RNNs that carried hidden state across calls.

In filtering KernelBench tasks, our rule was: only replace a task if it was fundamentally unfixable. Aggressive reductions and unusual or redundant operations flagged by METR were kept if they reflect standard real-world usage (e.g., classification) or surface non-trivial edge cases. In all other cases, we patched tasks to restore meaningful signals, and only replaced them when no fix was possible.

**Level 1.** Fixed issues like loss functions carrying extra broadcast dimensions, summing instead of averaging, or multiplying inputs by scalars that collapsed outputs to near zero. In two cases (product reduction, cosine‑similarity loss), we introduced replacements with operators that deliver usable gradient signals.

**Level 2.** We fixed the sequence of operations in composite pipelines that tended to cancel themselves out. Examples include a softmax placed after a global mean (producing all 1s), stride‑2 deconvolutions followed by pooling layers that undid the upsampling, or normalization blocks where the second pass had no effect. While originally included to add complexity, these patterns turned out to be uninformative for evaluating kernel quality. We moved reductions in meaningful orders, restored missing linear layers, reshaped biases to follow standard broadcast rules, and eliminated silent CPU tensors. Each task should now produce outputs that vary across inputs and remain inside the dynamic range that our tolerances assume.

**Level 3.** All RNN, LSTM, and GRU variants now take hidden (and cell) state as explicit inputs and return them as outputs. We removed internal buffers like self.h0 and self.c0 to prevent state leakage. Biases are now initialized with larger variance to avoid vanishing gradients during flow tests.

# More Robust Random Testing

Real-world models don't just run on perfectly behaved data. Testing kernels only with standard normal Normal(0, 1\) inputs can miss numerical bugs, so we’ve added a method called torch.rand\_mix to increase the likelihood of uncovering edge cases.This utility creates tensors by drawing from a diverse pool of distributions, allowing for users to probe different potential failure modes. Users can pick whether to randomly sample from this pool or to pick a specific distribution when calling rand\_mix. Note that by default the problems use the standard torch.randn, but you can swap it any time with torch.rand\_mix trivially. 
```
    # Standard baseline  
    ("normal",      lambda shape: Normal(0, 1).sample(shape)),

    # Bounded inputs  
    ("uniform",     lambda shape: Uniform(-1, 1).sample(shape)),  
    # Heavy-tailed inputs  
    ("laplace",     lambda shape: Laplace(0, 1).sample(shape)),

    # Strictly positive inputs  
    ("exponential", lambda shape: Exponential(1).sample(shape)),  
    ("lognormal",   lambda shape: LogNormal(0, 1).sample(shape)),
```