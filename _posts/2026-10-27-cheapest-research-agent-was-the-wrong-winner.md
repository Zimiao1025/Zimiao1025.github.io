---
layout: post
title: "Evidence-Constrained Harness Optimization for Verifiable Scientific Generation: Explaining the ‘Cheapest Wrong Winner’ Phenomenon"
description: "A controlled Biomni harness study showing why lower cost is not an efficiency gain unless scientific capability, executable evidence, and reviewability are preserved."
date: 2026-10-27 09:00:00 +0800
tags: [Scientific Agents, AI for Science, Biomedical AI]
reading_time: 19
title_style: compact
image: /assets/images/blog/echo-bio/echo-bio-hero.svg
---

<p class="post-kicker">Echo-Bio · a controlled study of biomedical-agent harnesses</p>

<p class="post-lede">Suppose a biomedical research agent becomes 14.08% cheaper. Is its harness better? In our first paired experiment, the cheaper agent also produced fewer scientifically reviewable results, lost 0.167 in scientific score, and introduced five new hard failures. A cost-only evaluation would have selected the wrong winner.</p>

That result motivated **Echo-Bio**, a framework for optimizing the harness around an execution-capable biomedical research agent while treating scientific evidence as a constraint—not as a score added after the search is over.

The central question is simple:

> Can we reduce the cost of an agent such as Biomni by changing its harness, while preserving the scientific work that makes its answer trustworthy and reviewable?

This article explains what we compared, how we decided whether a candidate could advance, why two plausible approaches failed, and what the final experiments actually support.

<figure class="post-figure post-figure-wide post-hero-art">
  <img src="{{ '/assets/images/blog/echo-bio/echo-bio-hero.svg' | relative_url }}" alt="Echo-Bio surrounds a fixed biomedical research policy with a governed loop connecting planning, execution, evidence, verification, and harness evolution." width="1600" height="900">
  <figcaption>The research policy is held fixed. Echo-Bio studies the surrounding harness: what the agent sees, executes, records, verifies, and uses to decide whether to continue.</figcaption>
</figure>

## Start here: what exactly did we compare?

Readers familiar with [Biomni](https://www.biorxiv.org/content/10.1101/2025.05.30.656746v1) can think of it as a research policy that selects biomedical resources, plans analyses, executes code, observes results, and writes a scientific answer. Echo-Bio is **not** a replacement foundation model and this study is **not** a benchmark of raw Biomni against a new agent.

Instead, we held the Biomni-style research policy and the evaluation environment constant, then changed one harness mechanism at a time.

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>Component</th><th>Baseline arm: B0</th><th>Candidate arm: B1</th></tr></thead>
  <tbody>
    <tr><td>Research policy</td><td>Biomni-style planning and execution</td><td>Exactly the same</td></tr>
    <tr><td>Provider and model policy</td><td>Frozen provider, model, thinking policy, price table, and budgets</td><td>Exactly the same</td></tr>
    <tr><td>Scientific environment</td><td>Frozen Python/R/Bash runtime, data mounts, and execution limits</td><td>Exactly the same</td></tr>
    <tr><td>Measurement</td><td>Shared observer, receipts, artifact store, replay rules, and hidden verifier</td><td>Exactly the same</td></tr>
    <tr><td>Harness treatment</td><td>Identity control: no Echo-Bio optimization</td><td>One explicitly registered optimization mechanism</td></tr>
  </tbody>
</table>
</div>

The common provider/runtime/observer/verifier layer is necessary because an out-of-the-box research agent does not expose every interface required for a controlled experiment. We added that layer to **both** arms. It is measurement infrastructure, not an efficiency advantage credited to Echo-Bio.

In plain language: the experiment asks what happens when the same research agent receives the same task and resources, but one arm uses a different rule for controlling the research loop.

## Why optimize the harness rather than the model?

A scientific agent is more than its language model. Its harness determines:

- which tools, libraries, and data sources are visible;
- how a plan becomes executable code;
- which previous observations are placed back into context;
- whether successful work is repeated;
- when the loop continues, repairs, or stops;
- which artifacts and execution records survive the run;
- and whether a final claim can be traced to an actual computation.

[SoL-Pi](https://nvlabs.github.io/SoL-Pi/) demonstrated that optimizing this layer can substantially improve the efficiency of a fixed agent. Biomedical research introduces an additional difficulty: a shorter transcript or a valid output file does not by itself establish that the analysis was appropriate, that a statistic came from executed code, or that the final prose stayed within the available evidence.

Echo-Bio therefore changes the optimization problem from **“find the cheapest harness”** to **“find the cheapest harness that remains above a scientific capability floor.”**

## The promotion rule, before any candidate is tested

Let a *candidate harness* mean one change to the control logic around the fixed research policy. We minimize provider cost only among candidates that satisfy all scientific and protocol constraints:

<div class="formula-card" role="img" aria-label="Select the candidate harness with the lowest expected provider cost, subject to scientific score and G2-valid output rate declining by no more than 0.03, no new hard violations, and complete claim-to-execution traceability.">
  <div class="formula-label">Evidence-constrained objective</div>
  <div class="formula-main">Choose the candidate with the lowest 𝔼[ provider cost per scheduled task ]</div>
  <div class="formula-rule"></div>
  <div class="formula-constraints">
    <span>Scientific-score change ≥ −0.03</span>
    <span>G2-valid-output-rate change ≥ −0.03</span>
    <span>New candidate-only hard violations = 0</span>
    <span>Material claim → evidence → execution traceability is complete</span>
  </div>
</div>

The operational decision sequence is:

<ol class="decision-steps">
  <li><strong>Qualify the comparison.</strong><span>Both arms must use the same provider policy, scientific runtime, observer, replay semantics, verifier, task bytes, and pre-treatment context.</span></li>
  <li><strong>Reject new hard failures.</strong><span>A candidate-only protocol failure or scientifically unreviewable result is disqualifying; success elsewhere cannot cancel it.</span></li>
  <li><strong>Apply the capability floor.</strong><span>Scientific score and the G2-valid-output rate may not fall by more than the preregistered 0.03 margin.</span></li>
  <li><strong>Only then compare efficiency.</strong><span>The primary endpoint is settled provider cost per scheduled task; a candidate must save at least 5% to be practically interesting.</span></li>
  <li><strong>Check whether the saving survives normalization.</strong><span>Cost per G2-valid output is reported as a diagnostic against apparently cheap but incomplete runs.</span></li>
</ol>

That final diagnostic is:

<div class="formula-card formula-card-compact" role="img" aria-label="Cost per valid output equals total settled provider cost divided by the number of G2-valid outputs.">
  <div class="formula-main">Cost per valid output = total settled provider cost ÷ number of G2-valid outputs</div>
</div>

This ordering is the key methodological choice. Evidence and capability filter the candidate set; cost ranks only the candidates that remain.

## A short dictionary for the evaluation

The original project documents contain several compact labels. Here is what they mean in this article.

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>Term</th><th>Plain-language meaning</th><th>How it is used</th></tr></thead>
  <tbody>
    <tr><td>Scientific score</td><td>A normalized, frozen task-specific rubric score for the produced scientific result.</td><td>Tests whether outcome quality has fallen; it is not a proof of biological truth.</td></tr>
    <tr><td>Receipt-valid output</td><td>Required executions and artifacts have valid exit states, schemas, hashes, and resolvable locations.</td><td>Separates fluent text from a result backed by durable execution records.</td></tr>
    <tr><td>G2-valid output</td><td>An output that passes receipt/artifact integrity plus deterministic scientific hard gates and the scientific-score rule.</td><td>The strict run-level capability and reviewability gate used for promotion.</td></tr>
    <tr><td>Hard violation</td><td>A candidate-only protocol failure, G2 failure, or unreviewable output that was absent in its paired baseline under the original paper rule.</td><td>Causes rejection; it cannot be offset by fixing a different baseline failure.</td></tr>
    <tr><td>Scientifically reviewable</td><td>A reviewer can resolve the reported result to persisted artifacts and successful executions under the frozen contract.</td><td>A necessary condition for scientific audit, but not a claim that the biology is true.</td></tr>
    <tr><td>Scheduled task</td><td>Every preregistered task, including an arm that fails or stops early.</td><td>Prevents failed or cheap incomplete runs from disappearing from the denominator.</td></tr>
  </tbody>
</table>
</div>

The project also uses two families of labels that should not be confused:

<aside class="reader-note">
  <strong>G0/G1/G2 are experimental decision gates.</strong> G0 checks outcome non-inferiority; G1 adds execution-receipt and artifact integrity; G2 adds deterministic scientific gates. <strong>L0/L1/L2/L3 are verifier authority levels</strong> describing what kind of fact a check can establish. G2 and L2 are not the same stage or the same scale.
</aside>

The verifier authority levels are:

<div class="table-scroll">
<table class="evidence-table">
  <thead><tr><th>Level</th><th>Question answered</th><th>Biomedical example</th></tr></thead>
  <tbody>
    <tr><td><span class="level-chip">L0</span></td><td>Did execution and persistence succeed?</td><td>The process exited successfully and the artifact hash and locator resolve.</td></tr>
    <tr><td><span class="level-chip">L1</span></td><td>Did domain-specific structural invariants hold?</td><td>A scRNA output contains required QC fields, or a GWAS result contains the specified harmonized columns.</td></tr>
    <tr><td><span class="level-chip">L2</span></td><td>Does a material claim follow from its linked evidence?</td><td>A reported cell count equals a typed value produced by the executed analysis.</td></tr>
    <tr><td><span class="level-chip">L3</span></td><td>Does the result satisfy the scientific rubric?</td><td>The method, limitations, and causal language satisfy the frozen task rubric or are routed for expert review.</td></tr>
  </tbody>
</table>
</div>

The intended provenance chain is explicit:

<div class="provenance-chain" aria-label="Material claim to executed evidence provenance chain">
  <span>Material claim</span><i>→</i><span>Typed evidence value</span><i>→</i><span>Artifact locator</span><i>→</i><span>Execution receipt</span><i>→</i><span>Code + data digest</span>
</div>

For example, consider a scRNA agent that reports “3,842 cells remained after quality control.” If it did not persist the filtered object, or if the number cannot be resolved to an executed result, a text-only evaluator might still find the answer plausible. Echo-Bio marks it unreviewable. Conversely, a fully traceable number may still be biologically misleading because of confounding or an inappropriate analysis choice. The verifier establishes bounded contract satisfaction and provenance—not biological truth.

## The common evaluation substrate makes the comparison identifiable

Before testing a harness, we qualified five shared components: provider transport, observer, scientific runtime, deterministic replay, and verifier behavior. The qualification corpus contained known-good and known-bad scRNA, bulk RNA-seq, and GWAS fixtures; it was not used to estimate the harness effect.

The resulting experiment is:

<div class="formula-card formula-card-compact" role="img" aria-label="B0 combines the Biomni research policy, common evaluation substrate, and identity harness. B1 uses the same policy and substrate plus one candidate harness mechanism. Their difference estimates the harness effect.">
  <div class="formula-main">B0 = Biomni research policy + common substrate + identity control</div>
  <div class="formula-main">B1 = Biomni research policy + common substrate + one candidate mechanism</div>
  <div class="formula-rule"></div>
  <div class="formula-main">Estimated harness effect = paired outcome of B1 − paired outcome of B0</div>
</div>

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/common-evaluation-substrate.svg' | relative_url }}" alt="A paired experiment in which baseline and candidate share provider, model, runtime, observer, replay, and verifier, while differing only in one harness treatment." width="1440" height="900" loading="lazy">
  <figcaption>Runtime, observer, and verifier are held fixed across both arms. They make the experiment measurable; they are not counted as candidate improvements.</figcaption>
</figure>

This point matters for interpreting every number below. Echo-Bio is not receiving a better model, a more capable scientific environment, or a more forgiving evaluator. The treatment is the harness mechanism named in that experiment.

## A failure-driven search: P1 → P4 → P5

The useful mechanism did not emerge from writing one “better” prompt. Candidate ideas came from prior traces, SoL-Pi-inspired harness categories, and human hypotheses. Each serious successor was isolated as its own lineage, evaluated prospectively, and either rejected or frozen before later data were opened.

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/failure-driven-lineage.svg' | relative_url }}" alt="A timeline from P1 to P4 to P5: P1 is cheaper but scientifically degraded, P4 preserves G2 but increases cost, and P5 moves termination into a deterministic typed control plane." width="1600" height="940" loading="lazy">
  <figcaption>P1, P4, and P5 are three different stopping-control treatments. P5 does not silently accumulate the earlier prompts.</figcaption>
</figure>

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>Lineage</th><th>What changed</th><th>Main observation</th><th>Decision</th></tr></thead>
  <tbody>
    <tr><td>P1</td><td>A prompt-level rule asked the model to act only on open evidence gaps and stop when evidence appeared complete.</td><td>Cheaper, but scientific score and valid-output rate fell; five new hard violations appeared.</td><td>Rejected as a false winner.</td></tr>
    <tr><td>P4</td><td>A phase-scoped natural-language efficiency directive was introduced after resource selection.</td><td>G2 was preserved in a three-pair screen, but cost rose 29.53% and the loop used more actions.</td><td>Rejected as inefficient.</td></tr>
    <tr><td>P5</td><td>Completion moved from model prose to typed evidence state and a deterministic local reducer.</td><td>Public search-dev showed less cost and post-completion work; the candidate was frozen for holdout.</td><td>Promising but bounded.</td></tr>
  </tbody>
</table>
</div>

### P1: the cheaper candidate did less valid science

**What changed.** P1 added a prompt-level evidence-complete stopping rule. The model saw only public required-evidence labels, scientific invariants, and the execution-step limit. It did not see reference answers or evaluator results.

**What we observed.** The frozen experiment contained six task identities—two each in scRNA, bulk RNA-seq, and GWAS—with three independent paired repeats per task: 18 pairs and 36 arms in total.

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>Endpoint per scheduled task</th><th>Identity B0</th><th>P1 candidate B1</th><th>Candidate change</th></tr></thead>
  <tbody>
    <tr><td>Settled provider cost</td><td>¥0.070735</td><td>¥0.060776</td><td class="metric-good">−14.08%</td></tr>
    <tr><td>Provider requests</td><td>5.111</td><td>4.333</td><td class="metric-good">−15.22%</td></tr>
    <tr><td>Input tokens</td><td>28,398.9</td><td>24,711.7</td><td class="metric-good">−12.98%</td></tr>
    <tr><td>Output tokens</td><td>1,742.1</td><td>1,419.1</td><td class="metric-good">−18.54%</td></tr>
    <tr><td>Scientific score</td><td>0.944</td><td>0.778</td><td class="metric-bad">−0.167</td></tr>
    <tr><td>Receipt-valid output rate</td><td>0.833</td><td>0.722</td><td class="metric-bad">−0.111</td></tr>
    <tr><td>G2-valid output rate</td><td>0.833</td><td>0.722</td><td class="metric-bad">−0.111</td></tr>
    <tr><td>New candidate-only hard violations</td><td>—</td><td>5</td><td class="metric-bad">Automatic rejection</td></tr>
  </tbody>
</table>
</div>

When cost was divided by the number of G2-valid outputs, the apparent 14.08% saving shrank to **0.86%**: ¥0.084882 for identity versus ¥0.084152 for P1. The candidate looked cheap primarily because it produced fewer acceptable results.

**What we inferred.** P1 optimized a model's *statement* that evidence was complete. It did not provide an independent control-plane proof that the required execution had been completed and persisted. The evidence-constrained gate therefore rejected a candidate that cost-only selection would have promoted.

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/false-winner-frontier.svg' | relative_url }}" alt="A capability-versus-cost diagram showing P1 below the scientific capability floor despite being cheaper, while P5 remains a bounded promising result." width="1440" height="900" loading="lazy">
  <figcaption>P1 is the “cheapest wrong winner”: it crosses the cost target by falling below the capability floor.</figcaption>
</figure>

### P4: asking the model to be efficient made it more expensive

**What changed.** P4 replaced the broad P1 instruction with a phase-scoped natural-language directive after resource selection. It asked the model to close remaining evidence gaps and stop without repeating successful work.

**What we observed.** In a small preregistered prospective screen of three pairs, P4 preserved G2 and introduced no new hard violation, but provider cost increased **29.53%** and action count increased.

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>Endpoint</th><th>P4 vs identity</th><th>Interpretation</th></tr></thead>
  <tbody>
    <tr><td>Provider cost</td><td class="metric-bad">+29.53%</td><td>Failed the efficiency objective.</td></tr>
    <tr><td>Executed action count</td><td class="metric-bad">Increased</td><td>The directive did not shorten the loop.</td></tr>
    <tr><td>G2-valid output</td><td>Preserved in 3/3 screened pairs</td><td>No observed capability loss in this small screen.</td></tr>
    <tr><td>New hard violations</td><td>0</td><td>Safety gate passed, but passing safety alone is not an efficiency win.</td></tr>
  </tbody>
</table>
</div>

**What we inferred.** Trace inspection showed serial gap closing, reconfirmation, and extra completion prose. Because the full transcript returned to later model calls, each additional turn also enlarged subsequent input. A natural-language request to be efficient was a behavioral suggestion, not an enforceable stopping mechanism.

### P5: completion becomes typed state, not prose

**What changed.** P5 is a new lineage, not another edit to the P4 prompt. It introduces three machine-checkable components:

1. one content-addressed resource-selection receipt shared by both paired arms;
2. a public compiler that converts the task's execution contract into typed evidence requirements without exposing answers or hidden rubric values;
3. a deterministic reducer that returns only `continue_batch`, `finalize_now`, or `fail_closed`.

The reducer reads successful execution receipts and the durable workspace. It does not trust a model sentence such as “the analysis ran” or “the output exists.” It returns all unresolved execution requirements together, finalizes when they are complete, and fails closed if the execution budget is exhausted first.

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/p5-control-plane.svg' | relative_url }}" alt="P5 control plane showing public task and resource receipt feeding typed evidence requirements, executed code and artifacts feeding a deterministic reducer, and the reducer choosing continue batch, finalize now, or fail closed before independent verification." width="1500" height="980" loading="lazy">
  <figcaption>P5 controls whether the research loop continues. The shared external verifier still judges the resulting execution and scientific output in both arms.</figcaption>
</figure>

**What we inferred.** Completion and correctness should be separated. P5 answers “is the public execution contract complete enough to stop?” The independent verifier answers “is the resulting output executable, traceable, and scientifically acceptable under the frozen rubric?”

## The P5 results: search evidence and holdout evidence answer different questions

P5 was first studied on a fixed public search-dev bank, then frozen before an internal sealed holdout was opened. The two datasets have different roles and should not be pooled into one headline number.

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>Result</th><th>Fixed public search-dev bank</th><th>Internal sealed holdout</th></tr></thead>
  <tbody>
    <tr><td>Purpose</td><td>Estimate the candidate's efficiency signal and inspect its mechanism before freezing.</td><td>Test the already-frozen P5 candidate on unseen internal tasks.</td></tr>
    <tr><td>Evaluation set</td><td>20 scheduled pairs; one failed shared resource selection before either arm, leaving 19 paired attempts across scRNA, bulk RNA-seq, GWAS, and proteomics.</td><td>6 independent tasks across scRNA, bulk RNA-seq, and GWAS; repeated runs were aggregated within task.</td></tr>
    <tr><td>Provider cost</td><td class="metric-good">−21.96%<br><small>95% task-cluster interval: 14.50% to 33.05%</small></td><td class="metric-good">−8.04%<br><small>95% task-cluster interval: 1.25% to 15.17%</small></td></tr>
    <tr><td>Provider requests</td><td class="metric-good">−17.24%</td><td>Not the confirmatory headline endpoint.</td></tr>
    <tr><td>Input / output tokens</td><td class="metric-good">−26.94% / −7.87%</td><td>Not used for the final holdout claim because shared-call token accounting was incomplete.</td></tr>
    <tr><td>Executed actions</td><td class="metric-good">−24.29%</td><td>Not the confirmatory headline endpoint.</td></tr>
    <tr><td>Tool wall time</td><td class="metric-good">−11.87%</td><td>Peak memory was unavailable; no complete systems-efficiency claim is made.</td></tr>
    <tr><td>G2-valid attempts</td><td>P5 18/19; identity 16/19</td><td>Aggregate scientific point estimates were equal.</td></tr>
    <tr><td>Hard-violation interpretation</td><td>Zero candidate-only hard violations in this corrected public matrix.</td><td>Zero protocol-invalid outputs, but formal replay found one candidate-only G2/unreviewable failure under the original, stricter paper definition.</td></tr>
    <tr><td>Permitted conclusion</td><td>Aggregate search-dev efficiency signal with run-to-run variation; not a holdout claim.</td><td>Provider-cost reduction is supported internally; scientific non-inferiority and strict zero-new-hard-violation are not established.</td></tr>
  </tbody>
</table>
</div>

The search-dev result was not deterministic: five of the 19 P5 trajectories cost more than identity, and repeated runs of the same task sometimes changed sign. That is why tasks—not individual stochastic trajectories—were the statistical unit, with repeats aggregated inside each task.

The sealed holdout gives the most conservative final statement. P5 reduced provider cost by 8.04%, and the cost interval stayed above zero. However, six independent tasks were not enough to establish scientific non-inferiority at the preregistered margin. In addition, a later audit found that the terminal implementation counted only protocol-invalid outputs, whereas the original protocol also classified a candidate-only G2/unreviewable result as a new hard violation.

The historical artifact was not rewritten. The mismatch remains part of the result.

## What this evidence supports—and what it does not

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>Claim category</th><th>Supported statement</th><th>Boundary</th></tr></thead>
  <tbody>
    <tr><td>Problem formulation</td><td>Cost-only selection promoted P1, while the evidence-constrained gate rejected it.</td><td>This demonstrates a selection failure in the studied setting, not in every agent benchmark.</td></tr>
    <tr><td>Mechanism</td><td>P5 moved stopping from model prose to receipt-backed typed state and removed measurable post-completion work.</td><td>It does not replace scientific verification or improve the model's underlying reasoning.</td></tr>
    <tr><td>Efficiency</td><td>The frozen P5 candidate reduced provider cost by 8.04% on the internal sealed holdout.</td><td>This is an internal result over six independent tasks and one provider/model configuration.</td></tr>
    <tr><td>Scientific capability</td><td>Holdout scientific point estimates were equal.</td><td>Statistical scientific non-inferiority was not established.</td></tr>
    <tr><td>Safety</td><td>P5 produced zero protocol-invalid holdout outputs.</td><td>It did not pass the original strict zero-new-hard-violation definition because formal replay found one candidate-only G2/unreviewable failure.</td></tr>
    <tr><td>Scientific validity</td><td>The verifier checks bounded execution, provenance, invariants, entailment, and rubric compliance.</td><td>It does not prove biological truth, remove confounding, or replace external replication and expert judgment.</td></tr>
    <tr><td>External validity</td><td>No external superiority claim is made.</td><td>The current evidence does not establish generalization across models, providers, open-ended tasks, BiomniBench, STELLA, or ScienceBuddy.</td></tr>
  </tbody>
</table>
</div>

This claim boundary is not an apology after the experiment. It is the same evidence-constrained principle applied to the paper itself.

## After P5: the next useful saving removed a whole provider call

After P5 was frozen, we mined its remaining public trajectories without using sealed-holdout item-level feedback. Several seemingly attractive compression ideas did not meet the preregistered practical threshold:

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>Residual mechanism</th><th>Observed full-pipeline cost effect</th><th>Decision</th></tr></thead>
  <tbody>
    <tr><td>Exact observation projection</td><td>−4.57%</td><td>Below the practical gate.</td></tr>
    <tr><td>Typed finalization pack</td><td>−0.12%</td><td>Negligible saving.</td></tr>
    <tr><td>Claim–evidence output projection</td><td class="metric-bad">+9.96%</td><td>More expensive.</td></tr>
    <tr><td>Local cache capsule</td><td>No provider-resolvable identity</td><td>Not safely deployable.</td></tr>
    <tr><td>Combined ledger-grounded projection</td><td>−2.865%</td><td>Below its 7% gate.</td></tr>
  </tbody>
</table>
</div>

These failures suggested that safe field-level compression had entered a low-return region. Evidence needed for later review could not simply disappear; it had to be replaced by typed state, locators, or recall metadata.

The bounded successor **A1** instead removes an entire resource-selection model call for registered task profiles. A deterministic compiler maps a frozen task type and resource inventory to the minimal resource bundle. Unknown profiles or inventory drift fail closed rather than silently guessing.

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/a1-resource-compiler.svg' | relative_url }}" alt="A comparison between a provider-based resource selector requiring one model call and a registry-grounded compiler requiring zero model calls before entering the same P5 and evidence-constraint loop." width="1500" height="880" loading="lazy">
  <figcaption>A1 changes resource selection for registered profiles. The downstream P5 loop and evidence-constraint substrate remain fixed in both arms.</figcaption>
</figure>

The final public search-dev analysis covered eight task identities and twelve paired trajectories across scRNA, bulk RNA-seq, GWAS, and proteomics:

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>A1 endpoint</th><th>Final public search-dev result</th><th>What may be claimed</th></tr></thead>
  <tbody>
    <tr><td>Full-pipeline provider cost</td><td class="metric-good">−38.901%<br><small>95% task-cluster interval: 33.085% to 44.643%</small></td><td>Strong bounded efficiency signal for registered profiles.</td></tr>
    <tr><td>G2-valid outputs</td><td>A1 12/12; provider-selector baseline 10/12</td><td>Observed capability was preserved or improved in this public matrix.</td></tr>
    <tr><td>Resource-selection provider calls</td><td class="metric-good">0 for A1</td><td>A1 deterministically removed that call for registered profiles.</td></tr>
    <tr><td>Total provider requests</td><td class="metric-good">−1.125 requests per task</td><td>The removed call translated to lower total request count.</td></tr>
    <tr><td>Downstream-only cost</td><td>−3.373%<br><small>95% interval: −6.844% to 12.824%</small></td><td>Downstream-only non-inferiority was not established.</td></tr>
  </tbody>
</table>
</div>

A1 is a **bounded successor**, not the P5 holdout result. It supports a claim about known task profiles under a frozen resource inventory. It does not support open-domain resource selection or general biomedical-agent superiority, and it has not been promoted through the same sealed-holdout path described for P5.

## Six lessons for scientific-agent research

<div class="takeaway-list">
  <div><span>01</span><p><strong>Define the scientific floor before searching.</strong> If the gate is invented after seeing a cheap candidate, the optimization is no longer a controlled scientific comparison.</p></div>
  <div><span>02</span><p><strong>Optimize valid work, not scheduled work alone.</strong> Cost per task can reward early failure; cost per valid output reveals that failure mode.</p></div>
  <div><span>03</span><p><strong>Keep the measurement substrate symmetric.</strong> Provider policy, runtime, observer, replay, and verifier must be identical before a difference can be attributed to the harness.</p></div>
  <div><span>04</span><p><strong>Move enforceable control out of prose.</strong> A prompt can request efficiency. Typed state and a deterministic reducer can decide whether a public execution contract is actually complete.</p></div>
  <div><span>05</span><p><strong>Preserve failed lineages.</strong> P1 explains why capability constraints are necessary; P4 explains why better wording is not enough; together they motivate P5.</p></div>
  <div><span>06</span><p><strong>Remove whole calls before shaving evidence fields.</strong> Once evidence is compact and traceable, further field compression may cost as much metadata as it saves.</p></div>
</div>

Echo-Bio's contribution is therefore not a claim that one reducer or compiler solves biomedical research. It is a method for studying harness optimization under scientific constraints: freeze the comparison, make execution evidence durable, separate stopping from verification, preserve negative results, and restrict every conclusion to the evidence that actually supports it.

> The cheapest agent was the wrong winner. A capability floor was what allowed us to notice.

---

### References and scope

- K. Huang et al., [“Biomni: A General-Purpose Biomedical AI Agent”](https://www.biorxiv.org/content/10.1101/2025.05.30.656746v1), bioRxiv, 2025; [official code](https://github.com/snap-stanford/Biomni).
- NVIDIA Research, [“SoL-Pi: Recursively Scaling Auto-Research Loops for Efficient Agent Harness”](https://arxiv.org/abs/2609.20519); [project page](https://nvlabs.github.io/SoL-Pi/).
- Echo-Bio results reported here use frozen internal artifacts and claim boundaries dated October 2026. External benchmark, independent expert audit, and cross-model replication remain future work.
