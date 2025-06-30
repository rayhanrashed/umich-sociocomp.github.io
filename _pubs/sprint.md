---
title: 'SPRINT: Enabling Interleaved Planning and Parallelized Execution in Reasoning Models'
authors:
  - name: Emil Biju
    equal: true
    affiliation: Stanford University
  - key: shayantalaei
    equal: true
    affiliation: Stanford University
  - name: Zhemin Huang
    equal: true
    affiliation: Stanford University
  - name: Mohammadreza Pourreza
    affiliation: Google
  - name: Amin Saberi
    affiliation: Stanford University
  - key: azaliamirhoseini
    affiliation: Stanford University
venue: preprint
year: 2025
date: 2025-06-06
has_pdf: true
doi: 10.48550/arXiv.2506.05745
tags:
  - machine learning
  - artificial intelligence
  - reasoning
teaser: SPRINT enables LRMs to dynamically identify and exploit parallelization opportunities during reasoning, reducing sequential tokens by up to 39% while maintaining performance.
materials:
  - name: Paper
    url: https://arxiv.org/abs/2506.05745
    type: file-pdf
  - name: Codebase
    url: https://github.com/ShayanTalaei/SPRINT
    type: code
---
Large reasoning models (LRMs) excel at complex reasoning tasks but typically generate lengthy sequential chains-of-thought, resulting in long inference times before arriving at the final answer. To address this challenge, we introduce SPRINT, a novel post-training and inference-time framework designed to enable LRMs to dynamically identify and exploit opportunities for parallelization during their reasoning process. SPRINT incorporates an innovative data curation pipeline that reorganizes natural language reasoning trajectories into structured rounds of long-horizon planning and parallel execution. By fine-tuning LRMs on a small amount of such curated data, the models learn to dynamically identify independent subtasks within extended reasoning processes and effectively execute them in parallel. Through extensive evaluations, we show that the models fine-tuned with the SPRINT framework match the performance of reasoning models on complex domains such as mathematics while generating up to ~39% fewer sequential tokens on problems requiring more than 8000 output tokens. Finally, we observe consistent results transferred to two out-of-distribution tasks of GPQA and Countdown with up to 45% and 65% reduction in average sequential tokens for longer reasoning trajectories, while achieving the performance of the fine-tuned reasoning model.
