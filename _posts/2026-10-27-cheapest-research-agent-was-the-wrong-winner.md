---
layout: post
title: "Echo-Bio: When Cost Minimization Fails for Biomedical Research Agents"
description: "Evidence-Constrained Harness Optimization for Efficient Biomedical Research"
date: 2026-10-27 09:00:00 +0800
permalink: /blog/2026/10/27/echo-bio/
tags: [Scientific Agents, AI for Science, Biomedical AI]
reading_time: 14
image: /assets/images/blog/echo-bio/echo-bio-hero.svg
---

<p class="post-lede">A cheaper research agent is useful only if it still does the research. Echo-Bio began as an attempt to reduce the cost of a biomedical agent—and became a study of what must never be optimized away.</p>

Imagine asking two agents to compare treatment and control cells. Both report the same number of differentially expressed genes. One number comes from executed code, a versioned count matrix, a saved result table, and a multiple-testing procedure that can be inspected. The other appears only in polished prose.

They may look equally convincing in a chat window. They are not equally useful scientific results.

This distinction matters when we try to make research agents more efficient. A self-improving research system has two levers: make each cycle better, or make an equally useful cycle cheaper so the same budget can buy more cycles. Echo-Bio studies the second lever. A less expensive cycle could let an agent test more hypotheses, recover from more failures, or run more rounds of self-improvement—but only if the cheaper cycles remain scientifically usable.

<div class="formula-card formula-card-compact" aria-label="The number of useful research rounds is approximately the total budget divided by the cost of one verifier-accepted research round.">
  <div class="formula-label">The quantity we actually care about</div>
  <div class="formula-main">useful research rounds ≈ total budget / cost of one accepted research round</div>
</div>

The word **accepted** is doing most of the work here.

**Echo-Bio asks one question:** can we make a biomedical research agent cheaper without losing the execution, artifacts, and evidence that make its answer scientifically reviewable?

**Code:** [Zimiao1025/Echo-Bio](https://github.com/Zimiao1025/Echo-Bio)

<figure class="post-figure post-figure-wide post-hero-art">
  <img src="{{ '/assets/images/blog/echo-bio/echo-bio-hero.svg' | relative_url }}" alt="Echo-Bio surrounds a fixed biomedical research policy with planning, execution, evidence, verification, and harness evolution." width="1600" height="900">
  <figcaption>Echo-Bio keeps the model and scientific task fixed, then studies the machinery around them: context, actions, observations, evidence, and stopping.</figcaption>
</figure>

## Why optimize the harness at all?

A system such as [Biomni](https://www.biorxiv.org/content/10.1101/2025.05.30.656746v1) is more than a language model answering a question. The model operates inside a **harness**: software that chooses what context the model sees, exposes tools and data, executes code, returns observations, stores artifacts, and decides whether the loop should continue.

That surrounding machinery can be expensive. A research loop may repeatedly send the same context, ask the model to rediscover an already known tool set, inspect the same output twice, or continue acting after the required analysis is complete. None of these costs necessarily reflects deeper scientific reasoning.

Inspired by [SoL-Pi](https://nvlabs.github.io/SoL-Pi/), we asked whether some of this harness overhead could be removed while keeping the underlying model and task unchanged. The transfer to biomedical research, however, introduced a complication: success is not captured by a final answer alone.

## Our first shortcut was cheaper—and worse

Our first strategy was deliberately simple. We told the agent to focus only on unresolved evidence requirements and stop once the evidence looked complete. It resembled the kind of prompt-level harness optimization that often works in tasks with a direct outcome score.

We compared it with the original loop using six task identities—two each in scRNA-seq, bulk RNA-seq, and GWAS—and repeated every matched comparison three times. The same model, provider, data, scientific environment, and request budget were used in both versions.

At first glance, the strategy looked successful:

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>What changed per scheduled task</th><th>Original loop</th><th>Cost-saving prompt</th><th>Difference</th></tr></thead>
  <tbody>
    <tr><td>Provider cost</td><td>¥0.070735</td><td>¥0.060776</td><td class="metric-good">−14.08%</td></tr>
    <tr><td>Model requests</td><td>5.111</td><td>4.333</td><td class="metric-good">−15.22%</td></tr>
    <tr><td>Input tokens</td><td>28,398.9</td><td>24,711.7</td><td class="metric-good">−12.98%</td></tr>
    <tr><td>Output tokens</td><td>1,742.1</td><td>1,419.1</td><td class="metric-good">−18.54%</td></tr>
    <tr><td>Scientific review score</td><td>0.944</td><td>0.778</td><td class="metric-bad">−0.167</td></tr>
    <tr><td>Accepted results</td><td>15/18 (83.3%)</td><td>13/18 (72.2%)</td><td class="metric-bad">−11.1 points</td></tr>
    <tr><td>New critical failures</td><td>—</td><td>5</td><td class="metric-bad">Rejected</td></tr>
  </tbody>
</table>
</div>

The agent made fewer calls and spent fewer tokens, but it also produced fewer results that met the scientific requirements. Some runs stopped before required evidence existed. Others produced outputs that could not be traced cleanly back to execution.

Once we divided total cost by the number of accepted results, the apparent 14.08% saving shrank to **0.86%**. Most of the saving came from doing less acceptable work.

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/false-winner-frontier.svg' | relative_url }}" alt="A capability-versus-cost diagram showing that the cheapest shortcut falls below the scientific capability floor." width="1440" height="900" loading="lazy">
  <figcaption>The cheapest version was the wrong winner: it crossed the cost target by falling below the scientific capability floor.</figcaption>
</figure>

This failure changed the project. We were no longer asking, “How can we make Biomni cheaper?” We were asking:

> What does it mean for two biomedical research trajectories to be equivalent enough that cost is a fair comparison?

## Scientific efficiency needs an evidence constraint

For an ordinary language task, a final-answer score may be a reasonable approximation of success. For a biomedical research agent, a result can sound correct while the trajectory underneath it is unusable:

- the statistic was never produced by executed code;
- the wrong samples or normalization were used;
- a result file exists, but the final claim does not point to it;
- the agent stopped before quality control or multiple-testing correction;
- causal language goes beyond what the experiment supports.

We therefore built a verifier that asks four progressively harder questions:

<div class="table-scroll">
<table class="evidence-table">
  <thead><tr><th>Question</th><th>What is checked</th><th>Example</th></tr></thead>
  <tbody>
    <tr><td><strong>Did the analysis run?</strong></td><td>Execution status, artifact hashes, schemas, and resolvable locations.</td><td>The differential-expression table exists and can be reopened.</td></tr>
    <tr><td><strong>Was the required analysis completed?</strong></td><td>Task- and domain-specific scientific requirements.</td><td>For scRNA-seq, donor structure is preserved and multiple testing is handled.</td></tr>
    <tr><td><strong>Do the claims follow from the results?</strong></td><td>Every material quantitative claim is linked to typed executed evidence.</td><td>A reported gene count equals the value stored in the executed result.</td></tr>
    <tr><td><strong>Is the answer scientifically reviewable?</strong></td><td>Method choice, limitations, and causal language are checked against a frozen rubric or sent to expert review.</td><td>An observational association is not presented as a demonstrated mechanism.</td></tr>
  </tbody>
</table>
</div>

The provenance path is explicit:

<div class="provenance-chain" aria-label="Material claim to executed evidence provenance chain">
  <span>Claim</span><i>→</i><span>Typed result</span><i>→</i><span>Saved artifact</span><i>→</i><span>Execution record</span><i>→</i><span>Code + data version</span>
</div>

This verifier does **not** prove biological truth. It cannot establish that a finding will replicate in another cohort, that a cell annotation is unquestionably correct, or that an experiment rules out every confounder. It defines a narrower and testable boundary: the work ran, the required evidence exists, the answer is supported by that evidence, and the scientific claims stay within the allowed scope.

Only after a result clears that boundary do we compare efficiency.

<div class="formula-card" aria-label="Choose the least expensive harness only among those that preserve scientific score, accepted result rate, critical safety constraints, and claim-to-evidence traceability.">
  <div class="formula-label">Evidence-constrained harness optimization</div>
  <div class="formula-main">choose the least expensive harness that still produces an accepted research result</div>
  <div class="formula-rule"></div>
  <div class="formula-constraints">
    <span>scientific score does not fall beyond the fixed margin</span>
    <span>accepted-result rate does not fall beyond the fixed margin</span>
    <span>no new critical failure is introduced</span>
    <span>material claims remain traceable to executed evidence</span>
  </div>
</div>

In our experiments, the allowable decline for the two quantitative scientific measures was fixed at 0.03 before the comparison. Cost could choose a winner only among methods that first met these conditions.

## Keeping the comparison honest

The verifier itself is not an efficiency improvement. Both versions of the agent receive exactly the same scientific checking and measurement infrastructure. We also keep the model, provider policy, task data, runtime, resource inventory, and execution limits fixed. The only intended difference is one harness change.

Here, the **original loop** means the Biomni-style research behavior running on this common evaluation setup. It does not mean an unobserved vanilla installation: execution and evidence are recorded in both versions so that the comparison can be audited.

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/common-evaluation-substrate.svg' | relative_url }}" alt="A paired comparison in which the original and modified agents share the model, runtime, observer, replay rules, and verifier." width="1440" height="900" loading="lazy">
  <figcaption>The verifier is a shared scientific floor. It prevents a cheaper but incomplete trajectory from being mistaken for progress.</figcaption>
</figure>

This separation matters. If only the modified agent received extra checking, its cost would be unfairly inflated. If neither version were checked, an incomplete run could win. The fair comparison is:

<div class="formula-card formula-card-compact">
  <div class="formula-main">same research agent + same verifier + one different harness mechanism</div>
</div>

## Letting trajectories tell us where the waste was

We did not jump directly from the failed prompt to the final mechanisms. We analyzed successful and failed trajectories to find where calls, actions, and context accumulated.

The initial search produced **12 candidate directions**: seven came from trace analysis, three were inspired by SoL-Pi, and two were human-proposed. Ten were implemented as isolated candidates. Ideas included lossless observation storage, adaptive observation projection, execution reuse, typed context compaction, diagnostic-log reduction, plan checkpoints, and evidence-aware stopping.

Most did not become positive results. That was useful information.

For example, a narrower natural-language stopping instruction preserved scientific acceptability in a small prospective test, but increased cost by **29.53%**. The model started closing gaps one at a time, reconfirming finished work, and explaining that it was done. Every extra turn then returned in later prompts, making the context larger.

Several later compression ideas also failed to clear their preregistered practical threshold:

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>Idea tested after the main improvement</th><th>What we learned</th></tr></thead>
  <tbody>
    <tr><td>Replace repeated observations with exact recall handles</td><td>Evidence remained recoverable, but estimated saving was 4.57%—below the 5% threshold.</td></tr>
    <tr><td>Compress the final evidence pack</td><td>A scientifically complete pack already contained most of the necessary bytes; saving was only 0.12%.</td></tr>
    <tr><td>Attach claim-to-evidence locations to the final answer</td><td>Useful for provenance, but not an efficiency win: output expanded and cost increased by 9.96%.</td></tr>
    <tr><td>Replace history with typed state and exact aliases</td><td>The state metadata consumed most of the removed context; net saving was 2.87%.</td></tr>
  </tbody>
</table>
</div>

The recurring lesson was simple: asking the model to *talk about* being efficient often creates more text. Safe compression also has a limit because scientific evidence cannot simply disappear.

The larger opportunity was to move decisions out of prose and into deterministic control.

## Stop when the evidence is complete—not when the model says it is

The strongest stopping mechanism represents the task as a small set of typed evidence requirements. After every execution, a deterministic controller reads the real execution records and asks:

1. Are required analyses or artifacts still missing? Continue and return the unresolved items together.
2. Is the required execution evidence complete? Stop generating new scientific actions and finalize.
3. Is the state inconsistent, or is the budget exhausted before completion? Fail closed.

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/p5-control-plane.svg' | relative_url }}" alt="Typed evidence requirements and executed artifacts feed a deterministic controller that chooses whether to continue, finalize, or fail closed." width="1500" height="980" loading="lazy">
  <figcaption>Completion becomes recoverable state outside the conversation, while the independent scientific verifier still judges the resulting work.</figcaption>
</figure>

The controller does not decide whether a biological conclusion is true. It decides whether the public execution contract has been completed well enough to stop. Scientific review remains independent and identical for both versions.

This distinction removes a specific kind of waste: calls and actions that happen **after** the required evidence already exists. It does not remove the analysis needed to produce that evidence.

### What happened on the public development tasks?

We evaluated the frozen mechanism on a fixed public set spanning scRNA-seq, bulk RNA-seq, GWAS, and proteomics. Twenty paired tasks were scheduled. One failed shared resource selection before either version began, leaving 19 completed comparisons. The numbers below describe this selected development set, not an external benchmark.

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>Measure</th><th>Typed stopping vs original loop</th><th>What it means</th></tr></thead>
  <tbody>
    <tr><td>Provider cost</td><td class="metric-good">−21.96%<br><small>95% interval across tasks: 14.50% to 33.05%</small></td><td>Exceeded the 5% practical threshold set before testing.</td></tr>
    <tr><td>Model requests</td><td class="metric-good">−17.24%</td><td>Fewer calls per scheduled task.</td></tr>
    <tr><td>Input tokens</td><td class="metric-good">−26.94%</td><td>Less accumulated context returned to the model.</td></tr>
    <tr><td>Output tokens</td><td class="metric-good">−7.87%</td><td>Less generated text.</td></tr>
    <tr><td>Executed scientific actions</td><td class="metric-good">−24.29%</td><td>Fewer actions after evidence completion.</td></tr>
    <tr><td>Tool execution time</td><td class="metric-good">−11.87%</td><td>Lower measured tool wall time.</td></tr>
    <tr><td>Accepted results</td><td>Typed stopping: 18/19<br>Original loop: 16/19</td><td>No observed aggregate loss on this task set.</td></tr>
    <tr><td>New critical failures</td><td>0</td><td>No new missing-evidence or protocol failure.</td></tr>
  </tbody>
</table>
</div>

Failed runs can stop early and look artificially cheap, so we also examined only comparisons where both versions succeeded. The cost reduction was **15.25%** when both results were accepted, and **15.08%** when both were free of critical failures.

The effect was not deterministic. Five modified trajectories cost more than their paired original run, and repeats of the same task sometimes changed direction. The supported conclusion is aggregate and bounded:

> On this fixed public development set, evidence-aware stopping reduced calls, context, actions, and provider cost without an observed aggregate loss of accepted scientific results.

## A second opportunity: do not rediscover a registered toolbox

Trajectory analysis revealed another large cost surface before the research loop even began. A Biomni-style agent asks the model to select tools and libraries from an inventory. For an open-ended question, that flexibility is valuable. But for a registered task profile with a known execution contract, the system was repeatedly paying the model to rediscover a bounded capability set.

We replaced that one stage with a deterministic resource compiler:

- it reads the public task type and frozen resource inventory;
- it produces a content-addressed resource receipt for the unchanged downstream loop;
- it refuses unknown task profiles or changed inventories instead of guessing;
- it leaves the model, scientific runtime, stopping controller, and verifier unchanged.

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/a1-resource-compiler.svg' | relative_url }}" alt="A registry compiler replaces a paid resource-selection model call for registered task profiles before entering the unchanged evidence-constrained research loop." width="1500" height="880" loading="lazy">
  <figcaption>For registered profiles, a deterministic compiler can remove one complete model-mediated stage without weakening downstream evidence requirements.</figcaption>
</figure>

The final public development analysis covered eight task identities and twelve paired trajectories across the four domains. Tasks—not repeated trajectories—received equal weight.

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>Measure</th><th>Registry compiler vs model selector</th><th>Interpretation</th></tr></thead>
  <tbody>
    <tr><td>Full-pipeline provider cost</td><td class="metric-good">−38.901%<br><small>95% interval across tasks: 33.085% to 44.643%</small></td><td>Strong development-set signal for registered profiles.</td></tr>
    <tr><td>Accepted results</td><td>Compiler: 12/12<br>Model selector: 10/12</td><td>No observed loss on this matrix.</td></tr>
    <tr><td>Resource-selection model calls</td><td class="metric-good">0 for the compiler</td><td>The targeted call was eliminated.</td></tr>
    <tr><td>Total model requests</td><td class="metric-good">−1.125 per task<br><small>95% interval: −1.313 to −0.938</small></td><td>Fewer complete-pipeline requests.</td></tr>
    <tr><td>New critical failures</td><td>0</td><td>No new failure introduced by the compiler.</td></tr>
    <tr><td>Downstream-only cost</td><td>−3.373%<br><small>95% interval: −6.844% to 12.824%</small></td><td>Downstream-only non-inferiority was not established.</td></tr>
  </tbody>
</table>
</div>

The 38.9% figure must be read carefully. It includes the resource-selection call that the compiler was designed to remove. It does **not** mean that downstream scientific reasoning became 38.9% cheaper. The result applies to the registered task profiles and frozen inventory we tested, not to open-domain resource selection.

## What we learned

The project left us with five practical lessons.

1. **Optimize cost per accepted research iteration, not cost per answer.** Otherwise, incomplete work can look like progress.
2. **The verifier is a scientific boundary, not an optimization.** It must be the same for the original and modified systems.
3. **Typed state can be cheaper than conversational self-report.** A model does not need another turn to announce that an externally verifiable contract is complete.
4. **Removing a whole model call can matter more than shaving a few fields from context.** But this is safe only when the stage is deterministic and the fallback boundary is explicit.
5. **Negative results map the real design space.** Several plausible compression strategies preserved evidence but saved too little; others made outputs longer or could not be replayed reliably.

For recursive self-improvement, this work supplies a missing denominator. More rounds under the same budget are meaningful only if they are more **verifier-accepted research rounds**.

Echo-Bio has not yet shown that cheaper accepted rounds lead to a better final discovery. That requires a different long-horizon experiment: give systems the same total budget, let them run different numbers of propose–execute–verify cycles, and compare the final scientific result. Our current results establish a prerequisite for that experiment, not its outcome.

## Scope and code

The evidence here comes from selected public development tasks, not an external benchmark or a claim of universal generalization. The experiments used a fixed provider/model policy and operational verifier. Human scientific judgment, independent replication, and testing across other models and task distributions remain necessary.

The sanitized, user-facing `0.1.0rc1` release is available at [Zimiao1025/Echo-Bio](https://github.com/Zimiao1025/Echo-Bio). It includes the runtime, evidence and trace infrastructure, verifier components, reviewed Biomni integration patch, and DeepSeek setup guide. It excludes experiment-reproduction scripts, benchmark corpora, credentials, private evaluators, human-review records, and raw paid responses.

---

### References

1. Huang, K. et al. [“Biomni: A General-Purpose Biomedical AI Agent.”](https://www.biorxiv.org/content/10.1101/2025.05.30.656746v1) bioRxiv, 2025. [Code](https://github.com/snap-stanford/Biomni).
2. Weng, L. [“Harness Engineering for Self-Improvement.”](https://lilianweng.github.io/posts/2026-07-04-harness/) Lil’Log, 2026.
3. Liu, H. et al. [“SoL-Pi: Recursively Scaling Auto-Research Loops for Efficient Agent Harness.”](https://arxiv.org/abs/2609.20519) arXiv:2609.20519, 2026. [Project page](https://nvlabs.github.io/SoL-Pi/).
4. Jin, R. et al. [“STELLA: Self-Evolving LLM Agent for Biomedical Research.”](https://arxiv.org/abs/2507.02004) arXiv:2507.02004, 2025.
5. Xue, S. et al. [“ScienceBuddy: Recursive-in-Recursive Self-Improvement for Interactive Scientific Agents.”](https://arxiv.org/abs/2609.17523) arXiv:2609.17523, 2026.

### Suggested citation

Hongliang Li and Shuibai Zhang. “Echo-Bio: When Cost Minimization Fails for Biomedical Research Agents.” 2026.<br>
[Article](https://zimiao1025.github.io/blog/2026/10/27/echo-bio/) · [Code](https://github.com/Zimiao1025/Echo-Bio)
