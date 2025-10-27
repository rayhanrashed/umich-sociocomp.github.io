---
title: "DIY-MOD: What If Moderation Didn't Mean Suppression?"
authors:
  - key: rayhanrashed
  - key: farnazjahanbakhsh
venue: preprint
year: 2025
date: 2025-01-15
doi: 10.48550/arXiv.2509.22861
tags:
  - human-computer interaction
  - content moderation
  - personalization
  - natural language processing
teaser: What if content moderation didn't require suppression? We introduce DIY-MOD, a browser extension that transforms sensitive content instead of removing it, enabling safer online engagement while preserving social connections and informational value. Our approach empowers users to define their own content boundaries and applies personalized transformations—from artistic rendering to selective inpainting—that address individual sensitivities while keeping surrounding context intact.
materials:
  - name: Paper (arXiv)
    url: https://arxiv.org/abs/2509.22861
    type: file-pdf
#   - name: Paper (PDF)
#     url: https://arxiv.org/pdf/2509.22861.pdf
#     type: file-pdf
  - name: Code
    url: https://github.com/UMichHCI/DIY_MOD
    type: link
---

> **🏆 This work was awarded [Best Application of AI]() at the Michigan AI Symposium 2025!**

<!-- > **🎉 Our paper is available on arXiv: [What If Moderation Didn't Mean Suppression?](https://arxiv.org/abs/2509.22861)** -->

## The Problem with Current Content Moderation

<!-- <img src="/imgs/teasers/diymod.png" alt="DIY-MOD Teaser" style="width: 100%; height: auto;"> -->

Imagine Aisha, a mother still reeling from a recent miscarriage, scrolling through her social media feed. Then, suddenly, it appears: a pregnancy announcement from a friend. For most, it's a moment of shared joy. But for Aisha, the experience is devastating. The platform offers no mechanism to filter such deeply personal sensitivities. Her options are stark: unfollow her friend and sever a valued connection, or remain and endure repeated distress.

Or consider Rex, recovering from binge-eating disorder. Food images trigger intense cravings and obsessive thoughts he cannot control. Yet his feed overflows with food content—every restaurant visit, every home-cooked meal, every dessert becomes a photo opportunity. He wants to see his friends, but not the food that dominates every frame.

These scenarios highlight a fundamental challenge in online content moderation: **what counts as harmful varies drastically by user, over time, and across situations**. Centralized, one-size-fits-all policies struggle to accommodate the deeply personal and contextual nature of content sensitivity. Existing moderation tools rely on blunt interventions that effectively boil down to suppression—whether through removal, algorithmic downranking, muting keywords, or unfollowing. These interventions force users into an impossible trade-off.

## A New Paradigm: Personalized Content Transformation

This work argues for and explores a new direction: **personalized content transformation**. Instead of suppressing entire posts, what if we could selectively transform the distressing elements while preserving the valuable surrounding content? This approach could allow users to remain engaged with their communities and access valuable information while mitigating exposure to aspects of content they find distressing.

We designed and developed **DIY-MOD** (Do-It-Yourself Moderation), a browser extension that operationalizes this concept. DIY-MOD empowers users to create their own moderation layer on top of mainstream platforms. It moves beyond binary show-or-hide filtering to offer a palette of nuanced, multi-modal interventions.

## How DIY-MOD Works

### 1. Natural Language Filter Creation

Users describe their sensitivities in their own words through a conversational interface. The system engages in grounding dialogue to help refine filters:

- "Does this sensitivity apply to all jewelry or specific types?"
- "Does this apply to all political topics, or specific ones like discussions of civilian casualties?"

This allows users to define sensitivities at their own level of specificity rather than being constrained to predetermined categories.

### 2. Diverse Transformation Palette

When content matches a user's filters, DIY-MOD applies context-appropriate transformations:

**For Images:**
- **Semantic Inpainting**: Removes triggering elements while reconstructing the background seamlessly
- **Visual Euphemism**: Replaces distressing objects with benign alternatives
- **Artistic Rendering**: Applies styles like Impressionism, Cubism, or Studio Ghibli to reduce photorealism
- **Obfuscation**: Blurs or occludes sensitive regions

**For Text:**
- **Selective Blurring**: Obscures specific triggering words or phrases
- **Rewriting**: Rephrases content to remove triggering concepts while preserving meaning
- **Personalized Warnings**: Provides context-aware overlays for entire blocks

### 3. Intelligent Selection

The system uses a two-stage pipeline to select the most appropriate transformation:

1. **Pruner**: Predicts which interventions would likely work best for the specific user and content
2. **Scorer**: Evaluates top candidates along four dimensions:
   - Transformation seamlessness
   - Semantic fidelity
   - Predicted emotional impact
   - Transformation appropriateness

This personalized evaluation considers the user's filter description, sensitivity level, and chat history.

## Real-World Evaluation

We evaluated DIY-MOD through two complementary user studies with 15 participants experiencing diverse personal sensitivities, including phobias, PTSD, and various life events.

### Study 1: In-Situ Evaluation

Participants browsed Reddit with DIY-MOD active, encountering real transformations in naturalistic contexts. Key findings:

**Newfound Agency**: Participants consistently described feeling empowered. As one participant noted: *"The ball now is in my court."* They could calibrate sensitivity levels, customize filter metadata, and click through to view original content when needed.

**Trust Through Transparency**: Visual indicators marking modified content were universally seen as essential. One participant explained: *"The indicators built trust and, even when things failed, they helped me understand that this was related to my filters."*

**Self-Care Without Social Friction**: Participants framed using DIY-MOD as an act of self-care that didn't require publicly penalizing content creators through platform reporting.

### Study 2: Preference Elicitation

To understand principles behind effective transformations, we presented participants with pairs of modified content. Our analysis revealed two critical insights:

**Context-Dependent Preferences**: The same participant made different trade-offs depending on the content. For informational content, they preferred transformations that preserved context. For low-value or severe triggers, they prioritized maximum distance from the trigger.

**Cognitive Closure Matters**: Successful interventions provide **cognitive closure**—either enough context to understand a scene without revealing harmful details (informational closure) or a complete, benign replacement that allows immediate mental disengagement (emotional closure). Failed interventions create cognitive gaps that provoke curiosity or imagination to fill in the missing pieces.

Our selection framework achieved 63% alignment with user preferences for images and 79.1% for text, significantly better than random selection.

## Broader Implications

### Transformation Enables Access, Not Avoidance

By transforming how content appears rather than removing it entirely, DIY-MOD enables users to engage with content they would otherwise avoid completely. As one participant put it:

> *"I think what you've done is life-impacting for people. Even if it's not perfect, I'm still thankful for it."*

### The Case for Platform Integration

While DIY-MOD demonstrates that personalized moderation is viable as middleware, platforms could implement this far more efficiently with direct content access, computational resources, and existing user behavioral signals. The question becomes one of priorities: **How much more could be achieved if platforms recognized emotional safety as a core user need rather than an afterthought?**

### Applications Beyond Personal Triggers

This transformation-based approach may extend to civic discourse. When valuable perspectives are obscured by hostile framing, repackaging content (e.g., by softening antagonistic language or highlighting shared values) could make it easier to engage with counter-attitudinal information constructively.

For more details, please see our [full paper on arXiv](https://arxiv.org/abs/2509.22861)!

---

*This research was conducted at the University of Michigan Social Computing Lab. Currently under review at ACM Conference on Human Factors in Computing (CHI) 2026! Wish us luck!*
