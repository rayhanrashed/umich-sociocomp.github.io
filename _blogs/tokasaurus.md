---
title: "Tokasaurus: An Inference Engine for High-Throughput Workloads"
authors:
  - key: jordanjuravsky
  - name: Ayush Chakravarthy
    affiliation: Stanford
  - key: ryanehrlich
  - name: Sabri Eyuboglu
    affiliation: Stanford
  - key: bradleybrown
  - name: Joseph Shetaye
    affiliation: Stanford
  - name: Christopher Ré
    affiliation: Stanford
  - key: azaliamirhoseini
tags:
  - natural language processing
  - systems
venue: none
year: 2025
date: 2025-06-05
materials:
  - name: Codebase
    url: https://github.com/ScalingIntelligence/tokasaurus
    type: code
---

# TL;DR

We’re releasing Tokasaurus, a new LLM inference engine optimized for throughput-intensive workloads. With small models, Tokasaurus benefits from very low CPU overhead and dynamic [Hydragen](https://arxiv.org/abs/2402.05099) grouping to exploit shared prefixes. For larger models, Tokasaurus supports async tensor parallelism for GPUs with NVLink and a fast implementation of pipeline parallelism for GPUs without. On throughput-focused benchmarks, Tokasaurus can outperform vLLM and SGLang by up to 3x+.

---

## Table of Contents

- [Intro](#intro)
- [Optimizing Small Models](#optimizing-small-models)
- [Optimizing Big Models](#optimizing-bigger-models)
- [Try it Out](#try-it-out)
- [Benchmarking Details](#benchmarking-details)
- [Acknowledgements](#acknowledgements)

---

## Intro

As LLMs get smarter, faster, and cheaper, the community keeps finding new ways to use them. Our own recent work has explored using models to [scan every file in a codebase](https://arxiv.org/abs/2501.14723), [sample 10,000 attempts for math and code problems](https://arxiv.org/abs/2407.21787), and [collaborate with other models to minimize cloud costs](https://arxiv.org/abs/2502.15964). Inference is now also an important part of the training process, where we use models to [generate synthetic data](https://arxiv.org/abs/2404.14219) or as part of [RL pipelines](https://arxiv.org/abs/2504.04736) that generate and train on model completions.

Crucially, these new inference workloads look quite different than the original LLM use case of serving a chatbot. Here, we care primarily about the total time and cost required to complete a large batch of sequences, and we care much less (if at all) about the individual latency of a single generation. In other words, we want high throughput!

Open-source inference engines (i.e. dedicated systems for running efficient LLM inference) like [FlexGen](https://arxiv.org/abs/2303.06865), [vLLM](https://arxiv.org/abs/2309.06180), and [SGLang](https://arxiv.org/abs/2312.07104) have been enormously valuable to the community. Inspired by and learning from these projects, we built a new engine, [Tokasaurus](https://github.com/ScalingIntelligence/tokasaurus/tree/main), designed from the ground up to handle throughput-focused workloads. We’ve optimized Tokasaurus for efficiently serving large and small models alike, allowing it to outperform existing engines on throughput benchmarks. In the rest of this blog, we’ll walk through some of these optimizations and show off a few settings where Tokasaurus really shines.

---

## Optimizing Small Models

To benchmark Tokasaurus with small models, we’ll use two workloads:

- Completing chatbot prompts from the ShareGPT dataset (this is a common benchmark for testing inference engines).
- Reproducing an experiment from [Large Language Monkeys](https://arxiv.org/abs/2407.21787), where we take 128 problems from the [GSM8K](https://arxiv.org/abs/2110.14168) math dataset and sample 1024 answers to every problem. The distinguishing feature of this workload is that there’s a lot of prefix sharing across sequences.

<div style="display: flex; gap: 16px; align-items: center;">
  <img src="/imgs/blog/tokasaurus/small.png" alt="Tokasaurus small models" style="max-width: 49%; height: auto; display: block;">
  <img src="/imgs/blog/tokasaurus/monkeys.png" alt="Tokasaurus large batch sampling" style="max-width: 49%; height: auto; display: block;">
</div>

Tokasaurus outperforms vLLM and SGLang on both of these benchmarks, in particular achieving over 2x the throughput of other engines on the Large Language Monkeys workload. Two main features contribute to these wins with small models:

### Minimizing CPU Overhead

LLM engines perform many different tasks on the CPU, like handling web requests, tokenizing inputs/detokenizing outputs, managing KV cache allocation, and preparing inputs for the model. If these CPU-side tasks cause the GPU-side model to stall, throughput can take a [big hit](https://blog.vllm.ai/2024/09/05/perf-update.html). To avoid these stalls, inference engines commonly make many CPU-side [tasks](https://blog.vllm.ai/2024/09/05/perf-update.html) [asynchronous](https://lmsys.org/blog/2024-12-04-sglang-v0-4/): while the GPU runs a forward pass for batch N, the CPU-side of the engine post-processes the results from batch N-1 and prepares the inputs for batch N+1. 

Tokasaurus goes one step further, making the CPU-side of the engine (what we call the manager) both asynchronous and adaptive. The manager’s goal is to maintain a deep queue of inputs for the model to run forward passes on. The manager monitors the size of this queue and can detect if the model is close to exhausting it (and therefore stalling the GPU). In these cases, the manager will automatically start skipping optional steps (like checking for stop strings and onboarding new sequences) until the model’s input queue is sufficiently deep again. This combination of asynchrony and adaptivity lets Tokasaurus serve small models with much less CPU overhead.

### Dynamic Prefix Identification and Exploration

Prefix sharing comes up all the time in LLM inference — not just when repeatedly sampling like in the Large Language Monkeys benchmark, but also when asking many questions about a long document or reusing a system prompt across many chatbot conversations.

Shared prefixes allow attention to be computed more efficiently. We first explored this idea last year with [Hydragen](https://arxiv.org/abs/2402.05099) (aka [cascade attention](https://flashinfer.ai/2024/02/02/cascade-inference.html) and [bifurcated attention](https://arxiv.org/abs/2403.08845)), but at the time we didn’t address the problem of detecting these shared prefixes in an engine where sequences are constantly starting and finishing. With Tokasaurus, we solve this detection problem by running a greedy depth-first search algorithm before every model forward pass that iteratively finds the longest shared prefixes possible. Hydragen is most impactful for small models, which spend a relatively larger fraction of total FLOPs on attention.

---

## Optimizing Bigger Models

Tokasaurus can also efficiently serve bigger models across multiple GPUs! Here, the most important optimizations are our implementations of pipeline parallelism (PP) and tensor parallelism (TP), which allow us to maximize throughput on GPUs with or without NVLink.

### Pipeline Parallelism for the GPU Poor

One of our original goals with Tokasaurus was to efficiently run multi-GPU inference on our lab’s L40S GPUs, which don’t have fast inter-GPU NVLink connections. Without NVLink, the communication costs incurred running TP across a node of eight GPUs are substantial. Therefore, efficient support for PP (which requires much less inter-GPU communication) was a high priority. PP needs a large batch in order to run efficiently, since batches from the manager are subdivided into microbatches that are spread out across pipeline stages. When optimizing for throughput, we’re generally already using the largest batch size that fits in GPU memory, so PP is often a natural fit for throughput-focused workloads. When benchmarking against vLLM’s and SGLang’s pipeline parallel implementations using Llama-3.1-70B on eight L40S GPUs, Tokasaurus improves throughput by over 3x:

<div style="display: flex; gap: 16px; align-items: center;">
  <img src="/imgs/blog/tokasaurus/pipeline.png" alt="Tokasaurus small models" style="max-width: 98%; height: auto; display: block;">
</div>

### Async Tensor Parallel for the GPU Rich

If you do have GPUs with NVLink (e.g. B200s and certain models of H100s and A100s), Tokasaurus has something for you too! Models in Tokasaurus can be torch compiled end-to-end, allowing us to take advantage of [Async Tensor Parallelism (Async-TP)](https://discuss.pytorch.org/t/distributed-w-torchtitan-introducing-async-tensor-parallelism-in-pytorch/209487). This is a relatively new feature in PyTorch that can overlap inter-GPU communication with other computations, partially hiding the cost of communication. In our benchmarks, we found that Async-TP adds a lot of CPU overhead to the model forward pass and only starts improving throughput with very large batch sizes (e.g. 6k+ tokens). Tokasaurus maintains torch-compiled versions of our models with and without Async-TP enabled, allowing us to automatically switch on Async-TP whenever the batch size is big enough:

<div style="display: flex; gap: 16px; align-items: center;">
  <img src="/imgs/blog/tokasaurus/big.png" alt="Tokasaurus small models" style="max-width: 98%; height: auto; display: block;">
</div>

---

## Try it Out

Tokasaurus started as an internal lab effort to run our inference experiments faster, and we’re excited to share it more broadly! You can check out the Tokasaurus code on [GitHub](https://github.com/ScalingIntelligence/tokasaurus/tree/main) and install the package from PyPI with:

```bash
pip install tokasaurus
```

Currently, we support models from the Llama-3 and Qwen-2 families and support any combination of data, tensor, and pipeline parallel within a single node.

Tokasaurus is written in pure Python (although we do use attention and sampling ops from the excellent [FlashInfer](https://arxiv.org/abs/2501.01005) package). We hope that this makes the engine easier to fork and hack on, a la [GPT-fast](https://github.com/pytorch-labs/gpt-fast).

## Benchmarking Details

The commands for reproducing our benchmarks are available [here](https://github.com/ScalingIntelligence/tokasaurus/blob/main/logs/blog_commands.md). For each benchmark, we configure all engines with the same KV cache size and maximum number of running requests. We’ve made a best effort to tune each engine’s remaining parameters. We report the average throughput across runs after completing a warmup run. For each benchmark, all engines are run on the same machine.

We use [this script](https://github.com/sgl-project/sglang/blob/7e257cd666c0d639626487987ea8e590da1e9395/python/sglang/bench_serving.py) from SGLang for our ShareGPT benchmarks and [this custom script](https://github.com/ScalingIntelligence/tokasaurus/blob/a0155181f09c0cf40783e01a625b041985667a92/tokasaurus/benchmarks/monkeys_gsm8k.py) for the Large Language Monkeys benchmark. To standardize our benchmarking scripts and interface, all experiments send requests through the OpenAI API. We also experimented with vLLM’s Python API (i.e. `LLM.generate()`) on the Large Language Monkeys benchmark with Llama-1B and measured roughly a 5% throughput increase (thanks to the vLLM team for the tip!).

## Acknowledgements

Huge thanks to Prime Intellect and Together AI for providing us with compute for this project.

Also, we’re grateful to Dan Biderman, Simon Guo, Manat Kaur, and Avanika Narayan for beta testing the engine!

---

<p>If you find Tokasaurus useful, please use the following citation:</p>

```bibtex
@misc{juravsky2025tokasaurus,
  author       = {Jordan Juravsky and Ayush Chakravarthy and Ryan Ehrlich and Sabri Eyuboglu and Bradley Brown and Joseph Shetaye and Christopher R{\'e} and Azalia Mirhoseini},
  title        = {Tokasaurus: An Inference Engine for High-Throughput Workloads},
  year         = {2025},
  howpublished = {\url{https://scalingintelligence.stanford.edu/blogs/tokasaurus/}}
}
```
