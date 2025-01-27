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
    url: https://arxiv.org/abs/2501.14723
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

  <p>Today, we're releasing CodeMonkeys, a system designed to solve software engineering problems by scaling test-time compute.</p>

  <img src="/imgs/blog/codemonkeys/system_overview.png" alt="CodeMonkeys system overview. There are 3 subtasks: context, generation, and selection" style="width: 100%; height: auto;">
  

  <p>CodeMonkeys scores 57.4% on SWE-bench Verified using Claude Sonnet 3.5.</p>
  <p>Our <a href="#selection">candidate selection</a> method can also be used to combine candidates from different sources. Selecting over an ensemble of edits from existing top SWE-bench Verified submissions obtains a score of 66.2%, outperforming the best member of the ensemble on its own and coming 5.5% below o3's reported score of 71.7%.</p>

  <p>We are releasing:</p>
  <ul>
      <li><strong><a href="https://arxiv.org/abs/2501.14723">A paper</a></strong> that describes our method in more detail.</li>
      <li><strong><a href="https://github.com/scalingintelligence/codemonkeys">The CodeMonkeys codebase</a>.</strong> This includes scripts for reproducing all results from our paper and running CodeMonkeys on SWE-bench problems.</li>
      <li><strong><a href="https://github.com/swe-bench/experiments/pull/171">All trajectories generated while solving SWE-bench problems.</a></strong> These trajectories contain all model outputs, candidate edits, test scripts, and execution traces generated while running CodeMonkeys on SWE-bench Verified.</li>
      <li><strong><a href="#costs">A careful accounting of the cost of running CodeMonkeys.</a></strong> Software engineering agents are expensive: running CodeMonkeys on SWE-bench Verified cost $2300 USD. </li>
      <li><strong><a href="https://huggingface.co/datasets/ScalingIntelligence/swe-bench-verified-codebase-content">A companion dataset containing complete Python codebase snapshots for all problems in SWE-bench Verified.</a></strong> This dataset makes it easier to work with SWE-bench by providing direct access to repository contents, without needing to clone and manage large Git repositories.</li>
  </ul>

  <section id="swebench">
      <h2>SWE-bench</h2>
      <p><a href="https://swebench.com">SWE-bench</a> is a benchmark that measures how well AI systems can solve real-world GitHub issues. Each instance in SWE-bench consists of an issue from a popular open-source Python repository (like Django or SymPy) along with the complete codebase at the time the issue was reported.</p>

      <img src="/imgs/blog/codemonkeys/swebench.png" alt="SWE-bench problem overview." style="width: 100%; height: auto;">
        
      <p>To solve an instance, a system must appropriately edit the given codebase in order to resolve the corresponding issue. An edit can be automatically evaluated for correctness using a set of unit tests that are hidden from the system.</p>

      <p>In this work, we've focused on <a href="https://openai.com/index/introducing-swe-bench-verified/">SWE-bench Verified</a>, a subset of SWE-bench where human annotators have filtered out low-quality instances (e.g. those with ambiguous issue descriptions).</p>
  </section>
    <section id="motivation">
        <h2>Let's talk about Monkeys</h2>
        <p>We began working on SWE-bench in our previous work, <a href="https://scalingintelligence.stanford.edu/pubs/large_language_monkeys/">Large Language Monkeys</a>. In that paper, we investigated the simple technique of using an LLM to sample many candidate solutions to a problem.</p>
        
        <p>When solving SWE-bench instances (in addition to tasks from other datasets), we found that coverage, the fraction of instances that are solved by at least one sample, often increases log-linearly with the number of samples drawn.</p>
        
        <p>More precisely, we found that the relationship between coverage and the number of samples can often be modeled using an exponentiated power law.</p>
        
        <img src="/imgs/blog/monkeys/coverage.png" alt="Coverage (percent of problems solved by any sample) increases across five coding and math reasoning tasks." style="width: 100%; height: auto;">

        <p>These results showed clear potential for how we might use test-time compute to improve performance on SWE-bench; by investing more test time compute in larger sample collections, we can steadily increase the probability that these collections contain correct solutions. However, acheiving high coverage does not mean that a system can solve SWE-bench issues with a high rate of success. Benefiting from coverage requires that a system can select a correct solution out of its collection of generations.</p>

        
        <p>Additionally, in Large Language Monkeys we generated candidate edits by repeatedly sampling from an existing framework (<a href="https://github.com/aorwall/moatless-tools">Moatless Tools</a>) which designed for generating only a single edit.</p>
        
        <p>This raised the question: if we were instead to build a system from scratch that was designed to solve SWE-bench instances by scaling test-time compute, what might that system look like?</p>
    </section>

    <section id="codemonkeys">
        <h2>CodeMonkeys</h2>
        <p>This question led us to build CodeMonkeys, a system designed to solve software engineering problems by scaling test-time compute. Similar to existing approaches like <a href="https://github.com/OpenAutoCoder/Agentless?tab=readme-ov-file">Agentless</a>, we decomposed solving SWE-bench issues into 3 subtasks:</p>

        <ol>
            <li><strong>Context:</strong> can we identify the codebase files that need to be edited and put them in the
context window? We can measure outcome of this subtask with <strong>recall</strong>: the fraction of
problems where all needed files have been identified.</li>
            <li><strong>Generation</strong>: can we produce a correct codebase edit among any of our sampled candidates?
We can measure this outcome of this subtask with <strong>coverage</strong>: the fraction of problems where
at least one generated edit is correct.</li>
            <li><strong>Selection</strong> can we choose a correct codebase edit from our collection of candidates? After
completing this subtask, we can measure our final <strong>score</strong>: the fraction of problems in the
dataset that are resolved by the edit our system submits.</li>
        </ol>

        <h3>Task 1: Context</h3>

        
        <div class="component-details">
            <p><strong>Inputs:</strong> Issue Description, Entire Codebase (up to millions of tokens of context)</p>
            <p><strong>Outputs:</strong> Relevant Files (120,000 tokens)</p>
        </div>

        <p>One of the key challenges when solving SWE-bench instances is managing the large volume of input context. Most SWE-bench codebases contain millions of tokens worth of context: this exceeds the context lengths of most available models. Further, it would be prohibitively expensive to process using frontier models.</p>

        <p>We need to filter the codebases down to something that fits into current models context windows. As we generate multiple candidate solutions, we can amortize this cost of context identification across all downstream samples.</p>
        
        <p>We use a simple but effective approach to find relevant files: we let a model (specifically, <a href="https://huggingface.co/Qwen/Qwen2.5-Coder-32B-Instruct">Qwen2.5-Coder-32B-Instruct</a>) read every Python file in the codebase in parallel and independently label each file as "relevant" or "not relevant". Then, we used <a href="https://claude.ai">Claude Sonnet-3.5</a> to rank the relevant files by importance, allowing up to 120,000 tokens of context.</p>

        <h3>Task 2: Generation</h3>
        
        <div class="component-details">
            <p><strong>Inputs:</strong> Issue Description, Relevant Files</p>
            <p><strong>Outputs:</strong> 10 (candidate edit, candidate test) pairs</p>
        </div>

        <center>
          <img src="/imgs/blog/codemonkeys/testing_and_editing_sm.png" alt="The testing and editing state machine."  style="width: 90%; height: auto; margin: auto 0;">
        </center>

        <p>We decompose this stage of our system into two back-to-back state machines:</p>
        
        <ol>
            <li>An initial <strong>testing state machine</strong> which iterates on an initial draft of the test script.</li>
            <li>A follow-up <strong>editing state machine</strong> which iterates on a codebase edit. This state machine is seeded with the output of a testing state machine and can also revise the testing script as needed.</li>
        </ol>
        
        <p>These state machines iteratively refine their outputs based on execution feedback. This provides two ways to scale test-time compute: we can increase the number of iterations per state machine ("serial scaling") or increase the number of independent state machines per problem ("parallel scaling").</p>

        <h3 id="selection">Task 3: Selection</h3>
        
        <div class="component-details">
            <p><strong>Inputs:</strong> Issue Description, Relevant Files, multiple (candidate edit, candidate test) pairs</p>
            <p><strong>Outputs:</strong> Final edit to codebase</p>
        </div>

        <center>
          <img src="/imgs/blog/codemonkeys/selection_sm.png" alt="The selection state machine."  style="margin-top: 4px; width: 50%; height: auto;">
        </center>

        <p>Finally, we select among the candidate solutions. We combine two approaches: using the model-generated tests to vote on solutions, and running a dedicated selection state machine that can write additional tests to differentiate between top candidates. Similar to the editing and testing state machines, this state machine can iterate based on execution feedback.</p> 
    </section>

<section id="results">
        <h2>Results by task</h2>
        <img src="/imgs/blog/codemonkeys/results_by_stage.png" alt="Results by stage, see paragraph below for data."  style="width: 100%; height: auto;">
        
        <div class="component-results">
            <h3>Context</h3>
            <p>With the 128k token limit, 92.6% of instances have the correct files in context. </p>
            
            <h3>Generation</h3>
            <p>By running multiple state machines in parallel and allowing each to iterate on their answer, we achieve 69.8% coverage. This means that for about 70% of problems, at least one of our candidate solutions is correct. Notably, in our experiments, we found that different ways of distributing compute between parallel scaling (more state machines) and serial scaling (more iterations per machine) often lead to similar coverage values.</p>
            
            <h3>Selection</h3>
            <p>Our selection method, which combines test-based voting with a dedicated selection state machine, recovers approximately half of the gap between random selection (45.8%) and using an oracle (69.8%). This leads to a final score of 57.4%.</p>
        </div>
  </section>

<section id="costs">
<h2>Cost Analysis</h2>

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


<p>There's a few things to note in our cost breakdown:</p>

<ul class="cost-analysis">
    <li>Our context identification contributes only 15% to total costs with a high recall. A simple linear scan with a small model can work! </li>
    
    <li>Generating edits is the most expensive component (60% of costs), primarily due to cache read costs from including codebase context in prompts.</li>
    
    <li>We reduce costs by separating testing and editing state machines, allowing us to omit codebase context from testing prompts.</li>
    
    <li>Selection contributes less than 10% to total costs while significantly improving final performance (see paper for ablations).</li>
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

  
  <p>Our selection mechanism can also be used to combine candidate edits from heterogeneous sources. We demonstrate this by creating what we call the "Barrel of Monkeys" - an expanded pool of candidate edits that includes solutions from CodeMonkeys along with the submissions from the top-4 entries on the SWE-bench leaderboard (Blackbox AI Agent, CodeStory, Learn-by-interact, and devlo).</p>

  <p>When we run our selection state machine over this expanded pool of candidate solutions, we achieve a score of 66.2%. This outperforms both CodeMonkeys on its own (57.4%) and the previous best ensemble submission (62.8%), showing how our selection method can effectively identify correct solutions even when they come from different frameworks.</p>

  <!-- <p>Something we took away from this is the importance of selection: with oracle selection could yield 10% gains. TODO FIX</p> -->


</section>

<section id="future">

<h2>Limitations & Future Work</h2>
<p>While CodeMonkeys demonstrates that scaling test-time compute is a viable approach for solving software engineering problems, there are clear opportunities for improvement across each component.</p>

<p>Our file-level filtering still misses relevant files in 7.4% of cases, and may have reduced effectiveness on private repositories not represented in model training data. The generation phase could incorporate additional execution feedback beyond issue reproduction tests and coordinate between parallel attempts to increase solution diversity. Most notably, there remains a significant gap between our selection performance and what would be possible with perfect selection - our best method recovers only about half of this gap when selecting CodeMonkeys solutions, and performs even less well when combining solutions from different frameworks in the Barrel of Monkeys.</p>

<p>We look forward to continued progress in methods for scaling test-time compute and their applications to real-world software engineering tasks.</p>

  <p>If our dataset, code, or paper was helpful to you, please consider citing:</p>
  <div style="width: 100%; overflow-x: auto">
  <code style="white-space: pre">
@misc{ehrlich2025codemonkeys,
      title={CodeMonkeys: Scaling Test-Time Compute for Software Engineering}, 
      author={Ryan Ehrlich and Bradley Brown and Jordan Juravsky and Ronald Clark and Christopher Ré and Azalia Mirhoseini},
      year={2025},
      eprint={2501.14723},
      archivePrefix={arXiv},
      primaryClass={cs.LG},
      url={https://arxiv.org/abs/2501.14723}, 
}</code>
</div>
</section>


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

</div>