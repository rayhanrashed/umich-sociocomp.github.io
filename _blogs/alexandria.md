---
title: "Alexandria: Choosing Your Own Values for Social Media Feeds"
authors:
  - name: Akaash Kolluri
    affiliation: Stanford University
    equal: true
  - name: Renn Su
    affiliation: Stanford University
    equal: true
  - key: farnazjahanbakhsh
  - name: Dora Zhao
    affiliation: Stanford University
  - name: Tiziano Piccardi
    affiliation: Stanford University
  - name: Michael S. Bernstein
    affiliation: Stanford University
venue: icwsm
year: 2026
date: 2025-05-16
doi: 10.48550/arXiv.2505.10839
tags:
  - human-computer interaction
  - social computing
  - personalization
  - social media
teaser: What if you could choose which values shape your social media feed? Alexandria offers a library of 78 values—from well-being to civic engagement—that users can mix and match to re-rank their X/Twitter feeds in real time, moving beyond engagement-driven algorithms toward user-controlled, values-based curation.
materials:
  - name: Paper (HTML)
    url: https://arxiv.org/html/2505.10839
    type: file-pdf
  - name: Paper (PDF)
    url: https://arxiv.org/pdf/2505.10839.pdf
    type: file-pdf
  - name: arXiv
    url: https://arxiv.org/abs/2505.10839
    type: link
---

> **📖 Our paper is available on arXiv: [Alexandria: A Library of Pluralistic Values for Realtime Re-Ranking of Social Media Feeds](https://arxiv.org/abs/2505.10839)**

## The Problem: Engagement Isn't Everything

<img src="/imgs/teasers/alexandria.png" alt="Alexandria Interface" style="width: 100%; height: auto;">

Today's social media algorithms optimize for one thing above all: **engagement**. They show you content that will make you like, share, and comment—regardless of whether that content supports your well-being, promotes healthy discourse, or aligns with values you actually care about.

The result? Many users report feeling that their feeds are:
- Over-optimized for time spent rather than time well spent
- Filled with content that seems arbitrary or misaligned with their preferences
- Frustrating, with limited effective controls to shape what they see

As one participant in our study put it: *"I try to just stay within the people I'm following, but I do often find myself navigating over to a For You page or the Explore page, which can often be a time drain."*

## A Different Approach: Value-Based Ranking

What if instead of letting engagement metrics alone determine your feed, you could explicitly choose which **values** matter to you? Should your feed prioritize:

- **Wisdom** and **Knowledge** over viral content?
- **Cheerful** and **Humorous** posts over rage-inducing ones?
- **Civic Engagement** and **Productive Discourse** over partisan animosity?
- **Well-being** and **Mental Health** over endless doom-scrolling?

This is the vision behind **Alexandria**—named after the historical Great Library of Alexandria, which curated knowledge across the ancient world. Our system similarly aims to enable a pluralistic approach to value-embedded feed curation.

## Building a Library of Values

Rather than choosing a single "correct" set of values, we built a **library** that combines multiple value systems from across the literature:

**Classical Psychology:**
- Rokeach's Value Survey (36 values)
- Maslow's Hierarchy of Needs (8 values)
- Hofstede's Cultural Dimensions (14 values)

**Social Media & Recommender Systems:**
- Stray et al.'s values for recommender systems (31 values)
- Reddit community values (29 values)
- Weibo co-creation values (28 values)

After filtering for post-level applicability and merging strongly correlated values, we arrived at **78 distinct values** that users can configure.

## How Alexandria Works

Alexandria is a Chrome browser extension for X/Twitter that operates in three steps:

### 1. Value Discovery and Selection

When you first install Alexandria, you're shown five diverse starting values:
- "A World at Peace"
- "Education and Entertainment"
- "Knowledge, Informativeness"
- "Appreciation"
- "Collectivism"

For each, you can **uprank** (want to see more), **downrank** (want to see less), or ignore it. The system then recommends similar values based on your selections, letting you iteratively refine your preferences.

### 2. Real-Time Feed Re-Ranking

As you browse X/Twitter, Alexandria:
1. Silently collects ~70 posts from your "For You" feed
2. Sends each post to GPT-4o-mini to label the strength of each value (0 = not present, 1 = weakly present, 2 = strongly present)
3. Computes a score for each post using the dot product of value labels and your configured weights
4. Re-orders your feed based on these scores

The entire process takes 5-10 seconds, adding minimal latency to your browsing experience.

### 3. Iterative Refinement

You can continuously adjust your values:
- Change weights using sliders (0.1 to 1.0)
- Add new values from the full library
- Remove values that aren't working
- See immediate results with each re-ranking

## What We Learned from Users

We conducted two studies to understand how people use Alexandria:

### Qualitative Study (N=12)

**Users appreciate control:**
> *"I felt like I definitely did have control. [I] definitely saw results from updating and re-ranking my feed based on changing the values… I did notice a big difference."* —P12

**The full library offers flexibility:**
10 of 12 participants preferred the full library over a single value system, noting:
> *"I like the one that has a lot of different options, because that helps me really kind of tweak what I'd be most interested in seeing."* —P9

**Values promote awareness:**
Using the tool made participants more conscious of what they wanted to see:
> *"There's one [value] here. It says obedient, and I'm thinking of those trad wife videos, and I'm like, that's not for me."* —P8

### Quantitative Study (N=257)

We compared users who had access to the **full library** (N=44) versus those limited to a **single value system** (N=213).

**Popular Values:**
The most commonly upranked values included:
- **Knowledge, informativeness** (72.3%)
- **Usefulness** (56.8%)
- **Wisdom** (54.5%)
- **Cheerful** (50.0%)

The most commonly downranked values:
- **Spam, Reposts, Bots** (43.2%)
- **Collectivism** (43.2%)
- **Adherence to norms** (40.9%)

**The library accommodates diversity:**
While some values were popular across users, **61.5% of values were selected by less than half of participants**—demonstrating that a large, diverse library is necessary to accommodate different needs.

**Filling gaps:**
When asked what values they wished existed, **65.5% of desired values** from users in single-system conditions were already present in our full library.

## Key Insights

### 1. No Universal "Best" Values

Our findings reinforce that **diverse user needs require diverse values**. While some values (like downranking spam) are nearly universal, many values matter deeply to specific individuals or subsets of users.

### 2. Precision Through Plurality

The full library allows users to be more **precise** about their preferences. For example, users who initially selected "Equality" (weight 0.50) in a single system switched to more specific values like "Broadminded" (0.23), "Care, compassion, empathy" (0.30), or "Tolerance" (0.13) when given access to the full library.

### 3. Control ≠ Filter Bubbles

Participants valued being able to **click through** to see original content and appreciated that the system wasn't perfect. As one noted:
> *"I don't feel that it's psychologically healthy to feel like you have 100% control, because that's not how the real world works."*

## Design Implications

**Toward a Marketplace of Values:**
Alexandria could be enhanced to let users and communities **propose new values**, creating a marketplace where people can subscribe to value configurations that match their needs.

**Balancing Autonomy and Society:**
While user control is valuable, we must thoughtfully design to prevent values-based filter bubbles. Platforms should include mechanisms that encourage exposure to diverse content while respecting user agency.

**Platform Integration:**
While Alexandria works as a browser extension, its approach could be **far more efficient** if integrated directly into platforms, which have access to full content inventories and computational resources.

## Looking Forward

Alexandria demonstrates that the values criticized as missing from social media algorithms **can be operationalized today** through end-user tools. By giving users control over which values shape their feeds, we can move toward social media experiences that align with what people actually care about—not just what keeps them scrolling.

The future questions are:
- How do we govern an expanding library of values?
- What are the long-term effects of value-based ranking on user behavior and societal discourse?
- How can we extend this approach to mobile platforms and other social networks?

For more details, please see our [full paper on arXiv](https://arxiv.org/abs/2505.10839)!

---

<!-- *This research was conducted as a collaboration between Stanford University and the University of Michigan, with support from the Brown Institute for Media Innovation and the Stanford Institute for Human-Centered Artificial Intelligence (HAI).* -->
