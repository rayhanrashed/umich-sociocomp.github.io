---
title: 'CodeMonkeys: Scaling Test-Time Compute for Software Engineering'
authors:
  - key: ryanehrlich
    equal: true
  - key: bradleybrown
    affiliation: University of Oxford
    equal: true
  - key: jordanjuravsky
    equal: true
  - name: Ronald Clark
    affiliation: University of Oxford
  - name: Christopher Ré
    affiliation: Stanford
  - key: azaliamirhoseini
venue: preprint
year: 2025
day: 212
has_pdf: true
doi: 10.48550/arXiv.2407.21787
tags:
  - machine learning
  - generative AI
teaser: CodeMonkeys is a system designed to leverage inference time compute in order to solve Github issues in the SWE-Bench Verified dataset.
materials:
  - name: Paper (TODO)
    url: 
    type: file-pdf
  - name: Codebase (TODO)
    url: https://github.com/ScalingIntelligence/codemonkeys
    type: code
  - name: Trajectory Data (TODO)
    url: 
    type: code
    display_name: Trajectory Data
---
Scaling test-time compute represents a new axis for improving LLM capabilities. However, test-time compute can be scaled in a variety of ways, and effectively combining different approaches remains an active area of research. Here, we present CodeMonkeys, a system designed to leverage test-time compute in order to solve real-world GitHub issues from the SWE-bench dataset. Our approach scales both serial and parallel test-time compute by sampling independent multi-turn trajectories that each iterate in response to execution feedback. Leveraging the ability to amortize up-front costs across multiple downstream samples, we identify relevant codebase context by simply letting a model scan every file. In order to decide among multiple candidate edits, we introduce a selection mechanism combining model-written unit tests with a final multi-turn loop dedicated to selection. Overall, CodeMonkeys solves 57.4\% of issues from SWE-bench Verified with a budget of approximately 2500 USD. When testing our selection method on an ensemble of edits from existing top submissions from the SWE-bench leaderboard, our score increases to 66.2\%.