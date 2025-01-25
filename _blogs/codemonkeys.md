---
title: "CodeMonkeys: Scaling Test-Time Compute for Software Engineering"
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
tags:
venue: none
year: 2025
date: 2025-01-21
teaser: 
materials:
  - name: Paper
    url: https://arxiv.org/
    type: file-pdf
  - name: CodeMonkeys Codebase
    url: https://github.com/ScalingIntelligence/codemonkeys
    type: code
  - name: Trajectories
    url: 
    type: database
  - name: Codebase Content Dataset
    url: https://huggingface.co/datasets/ScalingIntelligence/swe-bench-verified-codebase-content
    type: database
---

<div class="post-content">

  <p>Today, we're releasing CodeMonkeys, an open-source system designed to solve software engineering problems using test-time compute.</p>

  <img src="/imgs/blog/codemonkeys/system_overview.png" alt="" style="width: 100%; height: auto;">
  

  <p>CodeMonkeys scores 57.4% on SWE-bench Verified using Claude Sonnet 3.5. Additionally, CodeMonkeys' candidate selection method can effectively combine solutions from different systems. When selecting over an ensemble that includes solutions from CodeMonkeys and existing top SWE-bench verified submissions, we achieve a score of 66.2% - outperforming all memebers of the ensemble, and coming 5.5% below o3's reported score of 71.7%.</p>

  <p>To promote further research, we are releasing:</p>
  <ul>
      <li><strong><a href="https://github.com/scalingintelligence/codemonkeys">The CodeMonkeys codebase</a>,</strong> including scripts for reproducing all results from our paper and running the system on SWE-bench problems.</li>
      <li><strong><a href="#">All trajectories generated while solving SWE-bench problems.</a></strong> These trajectories contain all model outputs, all file contexts, candidate edits, test scripts, and execution traces generated while running CodeMonkeys on SWE-bench Verified.</li>
      <li><strong><a href="#costs">A careful accounting of the cost of running CodeMonkeys.</a></strong> Software Engineering agents are expensive: running CodeMonkeys on SWE-bench Verified cost $2300 USD. </li>
      <li><strong><a href="https://huggingface.co/datasets/ScalingIntelligence/swe-bench-verified-codebase-content">A companion dataset containing complete Python codebase snapshots for all problems in SWE-bench Verified.</a></strong> This dataset makes it easier to work with SWE-bench by providing direct access to repository contents, without needing to clone and manage large Git repositories.</li>
  </ul>
<!-- 

  <table class="leaderboard">
      <thead>
          <tr>
              <th>Method</th>
              <th>Selection</th>
              <th>Score</th>
          </tr>
      </thead>
      <tbody>
          <tr><td><strong>Barrel of Monkeys</strong></td><td>Oracle (Coverage)</td><td>80.8</td></tr>
          <tr><td>o3</td><td>---</td><td>71.7</td></tr>
          <tr><td><strong>CodeMonkeys</strong></td><td>Oracle (Coverage)</td><td>69.8</td></tr>
          <tr><td><strong>Barrel of Monkeys</strong></td><td>State Machine</td><td>66.2</td></tr>
          <tr><td>Blackbox AI Agent</td><td>---</td><td>62.8</td></tr>
          <tr><td>CodeStory</td><td>---</td><td>62.2</td></tr>
          <tr><td>Learn-by-interact</td><td>---</td><td>60.2</td></tr>
          <tr><td>devlo</td><td>---</td><td>58.2</td></tr>
          <tr><td><strong>CodeMonkeys</strong></td><td>State Machine</td><td>57.4</td></tr>
          <tr><td>Emergent E1</td><td>---</td><td>57.2</td></tr>
          <tr><td>Gru</td><td>---</td><td>57.0</td></tr>
      </tbody>
  </table>
 -->

  <section id="swebench">
      <h2>SWE-bench</h2>
      <p><a href="https://swebench.com">SWE-bench</a> is a benchmark that measures how well AI systems can solve real-world software engineering problems. Each problem in SWE-bench consists of an actual GitHub issue from a popular open-source Python repository (like Django or Sympy) and the complete codebase at the time the issue was reported.</p>

      <img src="/imgs/blog/codemonkeys/swebench.png" alt="SWE-bench problem overview." style="width: 100%; height: auto;">
        
      <p>To solve a SWE-bench issue, systems produce an edit to the codebase. This edit is evaluated for correctness using unit tests from the repo (these unit tests are hidden from the model). </p>

      <p>In this work, we've focused on <a href="https://openai.com/index/introducing-swe-bench-verified/">SWE-bench Verified</a>, a subset of SWE-bench validated by human annotators.</p>
  </section>


    <section id="motivation">
        <h2>Large Language Monkeys</h2>
        <p>We first became interested in SWE-bench during our previous work, <a href="https://scalingintelligence.stanford.edu/pubs/large_language_monkeys/">Large Language Monkeys</a>. In that paper, we demonstrated a promising property of LLMs: when solving software engineering (and other) problems, coverage, the fraction of problems that are solved by at least one attempt, increases log-linearly with the number of solutions drawn from the model.</p>
        
        <img src="/imgs/blog/monkeys/coverage.png" alt="Coverage (percent of problems solved by any sample) increases across five code and math reasoning tasks." style="width: 100%; height: auto;">

        <p><strong>This means that as you spend more test time compute by drawing more samples, the fraction of problems that you have at least one correct solution for increases consistently and predictably.</strong></p>
        
        <p>While these results showed clear potential for improving performance on benchmarks like SWE-bench, we only demonstrated coverage improvements. To achieve actual performance gains, a system needs to select a correct solution from many candidates. Additionally, we generated our candidates by running an existing framework <a href="https://github.com/aorwall/moatless-tools">Moatless Tools</a> multiple times with a positive temperature - although powerful and well-built, the framework wasn't designed for taking multiple attempts per problem.</p>
        
        <p>This raised the question: <em>how would one design a system for solving SWE tasks differently if benefiting from test-time compute scaling was a primary consideration?</em></p>
    </section>

    <section id="codemonkeys">
        <h2>CodeMonkeys</h2>
        <p>This question led us to build CodeMonkeys, a system designed to solve software engineering problems by scaling test-time compute. Similar to existing approaches like <a href="https://github.com/OpenAutoCoder/Agentless?tab=readme-ov-file">Agentless</a>, we decomposed resolving SWE-bench issues into 3 subtasks.</p>

        <h3>Task 1: Context</h3>

        
        <div class="component-details">
            <p><strong>Goal:</strong> Identify which files from the codebase need to be seen to resolve the issue</p>
            <p><strong>Inputs:</strong> Issue Description, Entire Codebase (up to millions of tokens of context)</p>
            <p><strong>Outputs:</strong> Relevant Files (120,000 tokens)</p>
        </div>

        <p>First, we identify relevant codebase context. As we generate multiple solutions, we can amortize the cost of context identification across all downstream samples. This lets us use a simple but effective approach: we let a model (specifically, Qwen2.5-Coder-32B-Instruct) read every Python file in the codebase and label each file as "relevant" or "not relevant". Then, we used Claude Sonnet-3.5 to rank the relevant files by importance, allowing up to 120,000 tokens of context.</p>

        <h3>Task 2: Generation</h3>
        
        <div class="component-details">
            <p><strong>Goal:</strong> Generate candidate solutions to the issue, along with candidate tests</p>
            <p><strong>Inputs:</strong> Issue Description, Relevant Files</p>
            <p><strong>Outputs:</strong> 10 (candidate edit, candidate test) pairs</p>
        </div>

        <center>
          <img src="/imgs/blog/codemonkeys/testing_and_editing_sm.png" alt=""  style="width: 90%; height: auto; margin: auto 0;">
        </center>

        <p>Then, we generate candidate solutions. We run multiple parallel state machines that each generate both a codebase edit and a corresponding testing script. These state machines iteratively refine their edits and tests based on execution feedback. This provides two ways to scale test-time compute: we can increase the number of iterations per state machine ("serial scaling") or increase the number of independent state machines per problem ("parallel scaling").</p>

        <h3>Task 3: Selection</h3>
        
        <div class="component-details">
            <p><strong>Goal:</strong> Select a correct solution from the candidate edits.</p>
            <p><strong>Inputs:</strong> Issue Description, Relevant Files, multiple (candidate edit, candidate test) pairs</p>
            <p><strong>Outputs:</strong> Final edit to codebase</p>
        </div>

        <center>
          <img src="/imgs/blog/codemonkeys/selection_sm.png" alt=""  style="width: 50%; height: auto;">
        </center>

        <p>Finally, we select among the candidate solutions. We combine two approaches: using the model-generated tests to vote on solutions, and running a dedicated selection state machine that can write additional tests to differentiate between top candidates. This selection method recovers approximately half of the gap between random selection and using an oracle to choose the correct solution.</p>
    </section>

<section id="results">
        <h2>Results by task</h2>
        <p>We evaluate each component of our system on SWE-bench Verified:</p>

        <img src="/imgs/blog/codemonkeys/results_by_stage.png" alt=""  style="width: 100%; height: auto;">
        
        <div class="component-results">
            <h3>Context</h3>
            <p>With the 128k token limit that we use for later experiments, 92.6% of instances have the correct files in context. .</p>
            
            <h3>Generation</h3>
            <p>By running multiple state machines in parallel and allowing each to iterate multiple times, we achieve 69.8% coverage. This means that for about 70% of problems, at least one of our candidate solutions is correct. Interestingly, we found that different ways of distributing compute between parallel scaling (more state machines) and serial scaling (more iterations per machine) often lead to similar coverage values.</p>
            
            <h3>Selection</h3>
            <p>Our selection method, which combines test-based voting with a dedicated selection state machine, recovers approximately half of the gap between random selection and using an oracle. This leads to a final score of 57.4%.</p>
        </div>
  </section>

<section id="costs">
<h3>Cost Analysis</h3>

<table class="cost-table">
    <thead>
        <tr>
            <th rowspan="2">Stage</th>
            <th colspan="4">Claude Sonnet-3.5 API Costs</th>
            <th>Local Costs</th>
            <th>Total Cost</th>
        </tr>
        <tr>
            <th>Input</th>
            <th>Output</th>
            <th>Cache Read</th>
            <th>Cache Write</th>
            <th>Qwen-2.5</th>
            <th>USD (%)</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Relevance</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>334.02</td>
            <td>334.02 (14.6%)</td>
        </tr>
        <tr>
            <td>Ranking</td>
            <td>0.00</td>
            <td>11.92</td>
            <td>1.10</td>
            <td>6.90</td>
            <td>0.00</td>
            <td>19.92 (0.9%)</td>
        </tr>
        <tr>
            <td>Gen. tests</td>
            <td>10.60</td>
            <td>295.15</td>
            <td>21.60</td>
            <td>112.64</td>
            <td>0.00</td>
            <td>439.99 (19.2%)</td>
        </tr>
        <tr>
            <td>Gen. edits</td>
            <td>14.67</td>
            <td>353.95</td>
            <td>636.82</td>
            <td>360.58</td>
            <td>0.00</td>
            <td>1366.02 (59.6%)</td>
        </tr>
        <tr>
            <td>Selection</td>
            <td>0.52</td>
            <td>51.12</td>
            <td>15.17</td>
            <td>65.14</td>
            <td>0.00</td>
            <td>131.95 (5.8%)</td>
        </tr>
        <tr class="total-row">
            <td><strong>Total</strong></td>
            <td>25.79</td>
            <td>712.14</td>
            <td>674.69</td>
            <td>545.26</td>
            <td>334.02</td>
            <td>2291.90 (100.0%)</td>
        </tr>
    </tbody>
</table>

<style>
.cost-table {
    width: 100%;
    border-collapse: collapse;
    margin: 1rem 0;
    font-size: 0.9rem;
    font-family: system-ui, -apple-system, sans-serif;
}

.cost-table th, .cost-table td {
    padding: 0.5rem;
    text-align: right;
    border: 1px solid #ddd;
}

.cost-table th {
    background-color: #f8f9fa;
    font-weight: bold;
}

.cost-table td:first-child {
    text-align: left;
}

.cost-table tr:hover {
    background-color: #f8f9fa;
}

.total-row {
    font-weight: bold;
    background-color: #f8f9fa;
}

.cost-table caption {
    margin-bottom: 1rem;
    text-align: left;
    color: #666;
    font-style: italic;
}
</style>


<p>The cost breakdown reveals several key insights about our system:</p>

<ul class="cost-analysis">
    <li>Our context identification contributes only 15% to total costs by amortizing this scan across multiple downstream samples.</li>
    
    <li>Generating edits is the most expensive component (60% of costs), primarily due to cache read costs from including codebase context in prompts.</li>
    
    <li>We reduce costs by separating testing and editing state machines, allowing us to omit codebase context from testing prompts.</li>
    
    <li>Selection contributes less than 10% to total costs while significantly improving final performance.</li>
</ul>

<style>
.cost-analysis {
    list-style-type: disc;
    padding-left: 1.5rem;
    margin: 1rem 0;
}

.cost-analysis li {
    margin-bottom: 0.75rem;
    line-height: 1.5;
}
</style>
</section>


<section id="ensemble">
  <h2>Barrel of Monkeys: Combining Solutions from Different Systems</h2>
  
  <p>Our selection mechanism can also be used to combine candidate edits from heterogeneous sources. We demonstrate this by creating what we call the "Barrel of Monkeys" - an expanded pool of candidate edits that includes solutions from CodeMonkeys along with the submissions from the top-4 entries on the SWE-bench leaderboard (Blackbox AI Agent, CodeStory, Learn-by-interact, and devlo).</p>

  <p>When we run our selection state machine over this expanded pool of candidate solutions, we achieve a score of 66.2%. This outperforms both CodeMonkeys on its own (57.4%) and the previous best ensemble submission (62.8%), showing how our selection method can effectively identify correct solutions even when they come from different frameworks.</p>
</section>

<section id="data">
  <h2>Data Release</h2>
  <p>We are releasing two complementary datasets that we hope support different aspects of research.</p>
  
  <h3><a>CodeMonkeys Trajectories</a></h3>
  <p>Our first dataset contains the complete problem-solving trajectories from running CodeMonkeys on SWE-bench Verified. For each of the 500 problems, we release all state data. This includes all LLM outputs.</p>

  <h3><a href="https://huggingface.co/datasets/ScalingIntelligence/swe-bench-verified-codebase-content">SWE-bench Codebase Content</a></h3>
  <p>Our second dataset provides efficient access to the Python codebases required to work on SWE-bench problems. Instead of requiring researchers to manage Git repositories, this dataset contains all Python files from the relevant repositories</p>
</section>

<section id="conclusion">
<h2>Conclusion</h2>
  
 <p>For more details about our methods, analysis of the trade-offs between different scaling approaches, and ablation studies of our selection methods, please read our paper: <a href="#">CodeMonkeys: Scaling Test-Time Compute for Software Engineering</a>.</p>
  <p>How to cite? If our dataset, code, or paper was helpful to you, please consider citing:</p>
</section>
</div>
```bibtex
@misc{ehrlich2025codemonkeys,
      title={Large Language Monkeys: Scaling Inference Compute with Repeated Sampling}, 
      author={Ryan Ehrlich and Bradley Brown and Jordan Juravsky and and Ronald Clark and Christopher Ré and Azalia Mirhoseini},
      year={2024},
      eprint={2407.21787},
      archivePrefix={arXiv},
      primaryClass={cs.LG},
      url={https://arxiv.org/abs/2407.21787}, 
}
```


<style>

.component-results {

}

.component-results h3 {
    /* color: #2c5282; */
    margin: 0
}

.component-results p {
    margin: 0px;
    margin-bottom: 1rem;
}

.post-content {
    max-width: 800px;
    margin: 0 auto;
    font-family: system-ui, -apple-system, sans-serif;
    line-height: 1.6;
    color: #333;
}

h2 {
    margin-top: 2rem;
    margin-bottom: 1rem;
    color: #1a1a1a;
}

h3 {
    margin-top: 1.5rem;
    color: #1a1a1a;
    font-size: 1rem;
}

.component-details {
  background-color: #f8f9fa;
  border-radius: 4px;
  padding: 0.4rem;
}

.component-details > p {
  margin: 0;
}

.figure {
    margin: 2rem 0;
    text-align: center;
    padding: 1rem;
    background-color: #f8f9fa;
    border-radius: 4px;
}

.placeholder {
    color: #666;
    font-style: italic;
}

.goal {
    font-size: 0.9em;
    color: #555;
}

section {
    margin-bottom: 3rem;
}

.leaderboard {
    width: 100%;
    border-collapse: collapse;
    margin: 1.5rem 0;
    font-size: 0.9rem;
}

.leaderboard th, .leaderboard td {
    padding: 0.5rem;
    text-align: left;
    border-bottom: 1px solid #ddd;
}

.leaderboard th {
    background-color: #f8f9fa;
    font-weight: bold;
}

.leaderboard tr:hover {
    background-color: #f8f9fa;
}

.post-content {
    max-width: 800px;
    margin: 0 auto;
    font-family: system-ui, -apple-system, sans-serif;
    line-height: 1.6;
    color: #333;
    padding: 1rem;
}

h1 {
    font-size: 2rem;
    margin-bottom: 1.5rem;
    color: #1a1a1a;
}

ul {
    margin-left: 1.5rem;
}

a {
    color: #2c5282;
    text-decoration: none;
}

a:hover {
    text-decoration: underline;
}
</style>

