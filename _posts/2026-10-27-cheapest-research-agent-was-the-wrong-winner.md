---
layout: post
title: "The Cheapest Research Agent Was the Wrong Winner"
description: "What Echo-Bio taught us about optimizing the harness around biomedical research agents without confusing lower cost with better science."
date: 2026-10-27 09:00:00 +0800
tags: [Scientific Agents, AI for Science, Biomedical AI]
reading_time: 16
image: /assets/images/blog/echo-bio/echo-bio-hero.svg
---

<p class="post-kicker">Evidence-Constrained Harness Optimization for Efficient and Verifiable Biomedical Research Agents</p>

<p class="post-lede">In our first paired experiment, an “optimized” biomedical research agent reduced provider cost by <strong>14.08%</strong>. It also lowered its scientific score, produced fewer reviewable outputs, and introduced <strong>five new hard violations</strong>.</p>

If we had optimized only for tokens, latency, or cost, we would have promoted it.

We did not.

> A biomedical research agent is not efficient if it becomes cheaper by silently doing less science.

That failure became the central idea behind **Echo-Bio**: optimize the machinery around an execution-capable research agent, but make evidence and scientific reviewability constraints of the optimization—not decorations added at the end.

<figure class="post-figure post-figure-wide post-hero-art">
  <img src="{{ '/assets/images/blog/echo-bio/echo-bio-hero.svg' | relative_url }}" alt="Echo-Bio surrounds a fixed biomedical research model with a governed loop connecting planning, execution, evidence, verification, and harness evolution." width="1600" height="900">
  <figcaption>Echo-Bio keeps the research model fixed and studies the harness around it: what the agent sees, executes, records, verifies, and learns from.</figcaption>
</figure>

## The model is only part of the agent

Systems such as [Biomni](https://www.biorxiv.org/content/10.1101/2025.05.30.656746v1) combine a language model with biomedical resources, planning, code execution, and iterative observation. This makes them capable of doing work that a model answering from text alone cannot do. But it also means that the surrounding **harness** becomes part of the scientific method.

The harness decides:

- which tools, libraries, and data sources the model sees;
- how plans become executable actions;
- how much prior context and tool output is replayed;
- when the loop continues, repairs, or stops;
- which artifacts survive the run;
- and whether a final claim can be traced back to an actual execution.

[SoL-Pi](https://nvlabs.github.io/SoL-Pi/) showed that a fixed agent can become substantially more efficient by optimizing this layer. Echo-Bio starts from the same broad observation, but changes the optimization problem for biomedical research: **execution success is necessary, but it is not enough**. A compressed log can still support a wrong statistic. A valid file can still come from an inappropriate analysis. A fluent conclusion can still outrun the evidence.

Echo-Bio is therefore not a new biomedical foundation model. It is a governed way to search over the harness around a research agent.

## Rewrite the objective before optimizing it

Ordinary efficiency optimization asks for the harness (h) with the lowest expected cost. Echo-Bio adds a scientific capability floor.

<div class="formula-card" role="img" aria-label="Choose the harness with minimum expected cost, subject to scientific score and G2 reviewability non-inferiority, zero new hard violations, and complete claim-to-execution reachability.">
  <div class="formula-label">Evidence-constrained objective</div>
  <div class="formula-main"><i>h</i><sup>★</sup> = arg min<sub><i>h</i> ∈ H</sub> 𝔼[ C(<i>h</i>) ]</div>
  <div class="formula-rule"></div>
  <div class="formula-constraints">
    <span>Δ ScientificScore(<i>h</i>) ≥ −δ<sub>s</sub></span>
    <span>Δ G2(<i>h</i>) ≥ −δ<sub>g</sub></span>
    <span>NewHardViolations(<i>h</i>) = 0</span>
    <span>Claim → Evidence → Execution reachability = 1</span>
  </div>
</div>

This distinction matters because scheduled-task cost can reward incomplete work. We therefore also track a capability-adjusted quantity:

<div class="formula-card formula-card-compact" role="img" aria-label="Valid-output-adjusted cost equals total settled provider cost divided by the number of G2-valid outputs.">
  <div class="formula-main">C<sub>valid</sub> = settled provider cost ÷ number of G2-valid outputs</div>
</div>

For our first candidate, the apparent 14.08% cost advantage shrank to **0.86%** after normalizing by valid outputs. Most of the “saving” came from producing fewer acceptable results.

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/false-winner-frontier.svg' | relative_url }}" alt="A capability-versus-cost diagram showing P1 below the scientific capability floor despite being cheaper, while P5 remains a bounded promising result." width="1440" height="900" loading="lazy">
  <figcaption>Cost-only selection would have promoted P1. The capability floor turns it into a false winner.</figcaption>
</figure>

## A fair comparison needs a common substrate

There was a methodological problem before we could compare two harnesses. A baseline research agent did not necessarily expose the provider interface, durable observer, replay semantics, scientific runtime, and external verifier required for a controlled experiment. Adding those facilities only to the candidate would make the comparison meaningless.

We therefore built **Common Evaluation Substrate v1**. Both arms share:

- the same provider, model, thinking policy, request budget, and price snapshot;
- the same resource-selection and action transport;
- the same Python/R/Bash scientific runtime and data mounts;
- the same lossless observer, content-addressed artifacts, and replay rules;
- the same evaluator, hidden from the agent and read-only during the run.

The experiment then becomes:

<div class="formula-card formula-card-compact" role="img" aria-label="B0 is the Biomni research policy plus a common substrate and identity harness; B1 is the same policy and substrate plus an Echo-Bio treatment; the harness effect is B1 minus B0.">
  <div class="formula-main">B0 = research policy + common substrate + identity harness</div>
  <div class="formula-main">B1 = research policy + common substrate + candidate harness</div>
  <div class="formula-rule"></div>
  <div class="formula-main">Harness effect = B1 − B0</div>
</div>

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/common-evaluation-substrate.svg' | relative_url }}" alt="A paired experiment in which baseline and candidate share provider, model, runtime, observer, replay, and verifier, while differing only in the harness treatment." width="1440" height="900" loading="lazy">
  <figcaption>The substrate is not credited as an Echo-Bio efficiency gain. It is the common measurement layer that makes the treatment identifiable.</figcaption>
</figure>

This reframed a long list of apparently mundane engineering failures—provider protocols, artifact identity, scientific dependencies, writable directories, replay alignment—as **substrate qualification**. They were not candidate wins or losses. They were prerequisites for measuring either one.

## What does the verifier actually verify?

“A verifier checks the science” sounds reassuring and says almost nothing. Echo-Bio instead separates four levels of authority.

<div class="table-scroll">
<table class="evidence-table">
  <thead>
    <tr><th>Level</th><th>Question</th><th>Example</th></tr>
  </thead>
  <tbody>
    <tr><td><span class="level-chip">L0</span></td><td>Did execution and persistence succeed?</td><td>Exit status, artifact hash, schema and locator integrity.</td></tr>
    <tr><td><span class="level-chip">L1</span></td><td>Did domain-specific invariants hold?</td><td>Required scRNA QC fields, GWAS outputs, or bulk RNA-seq partitions exist and are internally consistent.</td></tr>
    <tr><td><span class="level-chip">L2</span></td><td>Does a claim follow from the linked evidence?</td><td>A reported count or effect matches a typed value produced by successful execution.</td></tr>
    <tr><td><span class="level-chip">L3</span></td><td>Does the result satisfy the scientific rubric?</td><td>Method choice, limitations, causal language, and expert-review routing.</td></tr>
  </tbody>
</table>
</div>

The chain is explicit:

<div class="provenance-chain" aria-label="Material claim to evidence provenance chain">
  <span>Material claim</span><i>→</i><span>Typed evidence value</span><i>→</i><span>Artifact locator</span><i>→</i><span>Execution receipt</span><i>→</i><span>Code + data digest</span>
</div>

Consider a simplified scRNA example. An agent reports the number of cells remaining after QC, but does not persist the filtered matrix and cannot resolve the number to executed output. An outcome-only evaluator may accept the text. Echo-Bio's stricter gate marks it **not scientifically reviewable**.

That still does not prove biological truth. The verifier establishes bounded contract satisfaction, provenance, and rubric compliance. Hidden confounding, biological interpretation, and external replication remain scientific work.

## Three lineages changed how we thought about control

Echo-Bio did not arrive at its mechanism in one step. The original candidate pool contained twelve ideas from trace mining, SoL-Pi-inspired seeds, and human hypotheses. Ten were materialized as isolated lineages. The important story is not how many ideas existed; it is how successive failures changed the design.

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/failure-driven-lineage.svg' | relative_url }}" alt="A timeline from P1 to P4 to P5: P1 is cheaper but scientifically degraded, P4 preserves G2 but increases cost, and P5 moves termination into a deterministic typed control plane." width="1600" height="940" loading="lazy">
  <figcaption>Each successor answers a failure in the previous lineage. The lineage is causal, not a stack of features.</figcaption>
</figure>

### P1 — cheaper because capability was lost

P1 asked the agent to stop when required evidence appeared complete. Across six tasks—two each in scRNA, bulk RNA-seq, and GWAS—with three paired repeats per task, it looked efficient.

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>Endpoint</th><th>P1 vs identity</th><th>Interpretation</th></tr></thead>
  <tbody>
    <tr><td>Provider cost / scheduled task</td><td class="metric-good">−14.08%</td><td>Would pass a cost-only objective.</td></tr>
    <tr><td>Provider requests</td><td class="metric-good">−15.22%</td><td>Fewer calls.</td></tr>
    <tr><td>Input / output tokens</td><td class="metric-good">−12.98% / −18.54%</td><td>Less provider traffic.</td></tr>
    <tr><td>Scientific score</td><td class="metric-bad">−0.167</td><td>Outside the −0.03 capability margin.</td></tr>
    <tr><td>G2-valid output rate</td><td class="metric-bad">−0.111</td><td>Fewer reviewable outputs.</td></tr>
    <tr><td>New hard violations</td><td class="metric-bad">+5</td><td>Automatic rejection.</td></tr>
  </tbody>
</table>
</div>

P1 is Echo-Bio's most useful negative result. It shows why “spend less” and “do the same work more efficiently” are not interchangeable.

### P4 — asking for efficiency amplified the loop

P4 tried a more precise natural-language termination directive after resource selection. It preserved G2 on a small prospective screen and introduced no new hard violation. But provider cost **increased by 29.53%** and action count increased.

The instruction encouraged the model to close evidence gaps serially, reconfirm completed work, and spend additional turns explaining that it was finished. The full transcript then made every subsequent turn more expensive.

The lesson was sharper than “the prompt was bad”:

> A natural-language request to be efficient is not an enforceable control mechanism.

### P5 — move completion out of prose

P5 replaced self-reported completion with three machine-checkable components:

1. a shared, content-addressed resource receipt;
2. public typed evidence requirements compiled from the task contract;
3. a deterministic reducer returning only `continue_batch`, `finalize_now`, or `fail_closed`.

The reducer reads actual execution receipts and the durable workspace. It does not trust a sentence saying that an analysis ran or a file exists. When execution evidence is complete, it prevents more scientific actions; when the budget is exhausted before completion, it fails closed.

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/p5-control-plane.svg' | relative_url }}" alt="P5 control plane showing public task and resource receipt feeding typed evidence requirements, executed code and artifacts feeding a deterministic reducer, and the reducer choosing continue batch, finalize now, or fail closed before independent verification." width="1500" height="980" loading="lazy">
  <figcaption>P5 changes stopping and completion control. It does not replace the independent scientific verifier.</figcaption>
</figure>

## What the P5 evidence permits us to say

On the fixed public search-dev bank, P5 reduced provider cost by **21.96%**, requests by **17.24%**, executed actions by **24.29%**, and input tokens by **26.94%**. Candidate G2 passed 18/19 completed pairs versus 16/19 for identity. One additional scheduled pair suffered pre-arm attrition and remained in the immutable record rather than being silently replaced.

On the internal sealed holdout, the more conservative result was:

<div class="key-result" aria-label="P5 internal sealed holdout provider cost result">
  <span class="key-result-label">Provider cost / scheduled task</span>
  <strong>−8.04%</strong>
  <span>task-clustered paired-bootstrap 95% CI: 1.25% to 15.17%</span>
</div>

The scientific pass-rate point estimates were equal, but six independent tasks were not enough to establish inferential scientific non-inferiority. Formal reanalysis also found that the terminal implementation used a narrower definition of “hard violation” than the earlier protocol. Under the original pairwise rule, a strict zero-new-hard-violation claim does not pass.

That leaves a deliberately bounded conclusion:

<div class="claim-grid">
  <section class="claim-panel claim-panel-supported">
    <h3>What the evidence supports</h3>
    <ul>
      <li>Cost-only optimization selected a false winner.</li>
      <li>Evidence constraints changed promotion decisions.</li>
      <li>Typed deterministic termination removed measurable post-completion work.</li>
      <li>P5 reduced provider cost on the internal sealed holdout.</li>
      <li>The finalist was frozen before holdout evaluation.</li>
    </ul>
  </section>
  <section class="claim-panel claim-panel-limited">
    <h3>What it does not establish</h3>
    <ul>
      <li>Statistical scientific non-inferiority.</li>
      <li>Zero new hard scientific failures under every historical definition.</li>
      <li>Scientific truth from verifier output alone.</li>
      <li>Generalization across biomedical tasks, models, or providers.</li>
      <li>Superiority over Biomni, STELLA, ScienceBuddy, or external benchmarks.</li>
    </ul>
  </section>
</div>

I view this boundary as part of the result. A verifier-constrained project should also constrain its own claims.

## After P5: stop compressing fields, remove whole calls

Once P5 was frozen, we mined its remaining public trajectories instead of modifying it from holdout feedback. Several plausible residual mechanisms failed to clear the preregistered practical threshold:

- exact observation projection saved 4.57%;
- a typed finalization pack saved 0.12%;
- claim–evidence output projection increased cost by 9.96%;
- a local cache capsule had no provider-resolvable identity;
- a combined ledger-grounded projection saved 2.865%, below its 7% gate.

The common lesson was that safe representation-only compression had entered a low-return region. Much of the context being removed had to be replaced with state metadata, recall handles, or evidence needed for verification.

The next hypothesis, **A1**, targeted an entire provider call. For registered task types, the public task registry and execution contract already determine a minimal resource bundle. A deterministic registry compiler can produce that bundle without asking a model to rediscover it.

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/a1-resource-compiler.svg' | relative_url }}" alt="A comparison between a provider-based resource selector requiring one model call and a registry-grounded compiler requiring zero model calls before entering the same P5 and evidence-constraint loop." width="1500" height="880" loading="lazy">
  <figcaption>A1 removes a full resource-selection call for registered profiles; unknown or drifting profiles fail closed to the provider path.</figcaption>
</figure>

In the final public search-dev analysis—eight task identities, twelve paired trajectories across scRNA, bulk RNA-seq, GWAS, and proteomics—A1 produced:

<div class="metric-strip">
  <div><strong>−38.90%</strong><span>full-pipeline cost<br>95% CI 33.09–44.64%</span></div>
  <div><strong>12 / 12</strong><span>candidate G2<br>baseline 10 / 12</span></div>
  <div><strong>0</strong><span>resource-selection<br>provider calls</span></div>
  <div><strong>−1.125</strong><span>total requests<br>per task</span></div>
</div>

A1 is a **bounded successor**, not the P5 paper result. It applies to registered profiles under a frozen resource inventory. Its downstream-only cost point estimate favored A1 by 3.37%, but the interval crossed the −5% non-inferiority margin. We therefore froze the primary full-pipeline result and did not claim downstream-only non-inferiority or open-domain generalization.

## What I would carry into the next scientific-agent project

<div class="takeaway-list">
  <div><span>01</span><p><strong>Optimize valid work, not scheduled work.</strong> Cost per task is misleading when one arm silently produces fewer reviewable results.</p></div>
  <div><span>02</span><p><strong>Put control decisions in a control plane.</strong> Prompts can suggest behavior; typed state and deterministic reducers can enforce bounded behavior.</p></div>
  <div><span>03</span><p><strong>Keep verification symmetric.</strong> The same observer, runtime, evidence constraints, and rubric must apply to baseline and candidate.</p></div>
  <div><span>04</span><p><strong>Negative lineages are mechanism evidence.</strong> P1 and P4 explain why P5 is designed the way it is; they are not embarrassing prototypes to erase.</p></div>
  <div><span>05</span><p><strong>Freeze before looking.</strong> Search-dev, qualification, and sealed holdout serve different purposes. Holdout feedback cannot become the next patch.</p></div>
  <div><span>06</span><p><strong>Remove calls before shaving fields.</strong> Once scientific evidence is compactly represented, metadata overhead can consume the theoretical gains from further compression.</p></div>
</div>

Echo-Bio's central contribution is not that one reducer or compiler is universally optimal. It is a way to conduct harness optimization as a scientific process: controlled comparisons, executable evidence, explicit capability floors, governed candidate lineages, and claim boundaries that survive contact with disappointing results.

> The cheapest agent was the wrong winner. The verifier was what allowed us to notice.

---

### References and scope

- K. Huang et al., [“Biomni: A General-Purpose Biomedical AI Agent”](https://www.biorxiv.org/content/10.1101/2025.05.30.656746v1), bioRxiv, 2025; [official code](https://github.com/snap-stanford/Biomni).
- NVIDIA Research, [“SoL-Pi: Recursively Scaling Auto-Research Loops for Efficient Agent Harness”](https://arxiv.org/abs/2609.20519); [project page](https://nvlabs.github.io/SoL-Pi/).
- Echo-Bio results reported here use frozen internal artifacts and claim boundaries dated October 2026. External benchmark and cross-model comparisons remain future work.
