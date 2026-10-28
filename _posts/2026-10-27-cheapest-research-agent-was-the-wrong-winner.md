---
layout: post
title: "Echo-Bio: When Cost Minimization Fails for Biomedical Research Agents"
description: "Evidence-Constrained Harness Optimization for Efficient Biomedical Research"
date: 2026-10-27 09:00:00 +0800
permalink: /blog/2026/10/27/echo-bio/
tags: [Scientific Agents, AI for Science, Biomedical AI]
reading_time: 20
image: /assets/images/blog/echo-bio/echo-bio-hero.svg
---

<p class="post-lede">A recursive research agent can improve in two ways: make every iteration better, or make an equally useful iteration cheaper so that the same budget buys more attempts. Echo-Bio studies the second problem—and finds that, in science, two answers are not equivalent merely because their prose looks equally convincing.</p>

<aside class="reader-note">
  <strong>In brief.</strong> Echo-Bio changes the <em>harness</em>—the controller around a fixed model that assembles context, invokes tools, records observations, and decides when to stop—not the biomedical model itself. A naive cost-saving prompt (P1) cut provider cost by 14.08% but also damaged scientific validity. A typed, evidence-aware stopping controller (P5) later reduced aggregate provider cost by 21.96% on 19 paired public development runs without an observed aggregate validity loss. A separate bounded mechanism (A1) removed one model-mediated resource-selection stage for registered task profiles. These are selected public search-dev results, not yet external or long-horizon RSI validation.
</aside>

**Code:** [Zimiao1025/Echo-Bio](https://github.com/Zimiao1025/Echo-Bio) — a sanitized, user-facing release with the runtime, evidence and trace infrastructure, verifier components, Biomni integration patch, and DeepSeek setup guide.

Suppose a laboratory has a fixed inference budget of **B**, and one propose–execute–evaluate cycle costs **C**. Ignoring fixed overhead, the budget can support roughly:

<div class="formula-card formula-card-compact" aria-label="The number of recursive research rounds affordable under budget B is approximately the floor of B divided by the expected cost of one round.">
  <div class="formula-label">The budget argument</div>
  <div class="formula-main">N<sub>rounds</sub>(B, h) ≈ floor( B / 𝔼[C<sub>round</sub>(h)] )</div>
</div>

This exposes two complementary directions for recursive self-improvement (RSI):

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>RSI direction</th><th>Question</th><th>Representative idea</th></tr></thead>
  <tbody>
    <tr><td>Improve each round</td><td>Can a better model or harness produce a stronger research result per iteration?</td><td>ScienceBuddy couples inner harness evolution with outer model learning; STELLA accumulates tools and reasoning templates.</td></tr>
    <tr><td>Buy more valid rounds</td><td>Can the same model produce an equivalent, scientifically valid result with fewer calls, tokens, actions, or cost?</td><td>Echo-Bio isolates this efficiency problem for biomedical research agents.</td></tr>
  </tbody>
</table>
</div>

The first direction is important and complementary, but deliberately outside this study: Echo-Bio does not claim to make the model scientifically smarter within one round. It asks whether a fixed research policy can complete an equally valid round with less avoidable harness overhead.

The second direction matters because lower cost compounds across a recursive loop. But it only helps if the cheaper run remains a **valid scientific run**. Consider two agents that report the same differential-expression count. One links that number to executed code, a versioned input, and a persisted result table; the other merely states it in fluent prose. An answer-only score may treat them as equivalent. A scientific workflow cannot.

If an agent saves money by skipping an analysis, losing an artifact, or making a claim that cannot be traced to execution, then the extra iterations are illusory.

That is the motivation for Echo-Bio:

> Reduce the cost of one biomedical research iteration without reducing the scientific capability and executed evidence delivered by that iteration.

<figure class="post-figure post-figure-wide post-hero-art">
  <img src="{{ '/assets/images/blog/echo-bio/echo-bio-hero.svg' | relative_url }}" alt="Echo-Bio surrounds a fixed biomedical research policy with planning, execution, evidence, verification, and harness evolution." width="1600" height="900">
  <figcaption>Echo-Bio keeps the research policy fixed and optimizes the harness around it: context, actions, observations, evidence state, and stopping control.</figcaption>
</figure>

## The subproblem Echo-Bio isolates

Consider a [Biomni](https://www.biorxiv.org/content/10.1101/2025.05.30.656746v1)-style research loop. Biomni is a general-purpose biomedical agent that lets a language model select scientific resources, write and execute analysis code, inspect observations, and synthesize an answer:

1. interpret a biomedical question;
2. select tools, libraries, and data resources;
3. plan and execute analyses;
4. observe results and decide whether more work is needed;
5. produce a scientific answer with supporting artifacts.

Echo-Bio does not train a new biomedical model. It asks whether the **harness** around the same research policy can execute this loop more efficiently. Here, “harness” has a concrete operational meaning: the prompt and context assembly, tool/action interface, observation representation, execution controller, and stopping logic surrounding the model.

<div class="table-scroll">
<table class="result-table experiment-control-table">
  <thead><tr><th>Held fixed in both arms</th><th>Allowed to change</th></tr></thead>
  <tbody>
    <tr>
      <td>
        <ul class="table-cell-list">
          <li>Biomedical research policy and base model</li>
          <li>Provider, thinking policy, request budget, and price snapshot</li>
          <li>Scientific runtime, dependencies, data mounts, and execution limits</li>
          <li>Task bytes and pre-treatment resource context</li>
          <li>Observer, execution receipts, artifact store, and replay rules</li>
          <li>Hidden scientific verifier and rubric</li>
        </ul>
      </td>
      <td class="treatment-cell"><strong>One registered harness mechanism</strong><span>For example, how completion is represented and when the research loop is allowed to stop.</span></td>
    </tr>
  </tbody>
</table>
</div>

The baseline is therefore not an uninstrumented installation of Biomni. Both arms receive the same measurement and verification substrate. The baseline uses an **identity harness**; the candidate adds exactly one Echo-Bio optimization. This lets a paired difference be attributed to the harness rather than to a different model, runtime, or evaluator.

## Why SoL-Pi cannot be copied directly into scientific research

[SoL-Pi](https://arxiv.org/abs/2609.20519) provides a compelling demonstration of the second RSI direction. On EdgeBench, it reports performance comparable to Pi while reducing recorded token traffic by 44.7–49.0% and API cost by roughly one third. Its broader lesson is that a fixed model can become much more efficient when action execution, context, observations, and delegated work are optimized at the harness level.

Echo-Bio starts from that lesson, but biomedical research changes what counts as an “equivalent result.” A coding task may often have an executable test or a directly scored outcome. A scientific answer can look correct even when:

- the reported statistic was never produced by executed code;
- the analysis used the wrong sample partition or normalization;
- an output file exists but cannot be linked to the claim;
- a model stopped before producing required evidence;
- causal language is stronger than the experiment supports.

SoL-Pi solves its stated problem; the issue is **transfer**. Scientific harness optimization needs a capability definition that includes the result, the execution, and the claim–evidence relationship.

Our first candidate, **P1**, exposed this difference.

Echo-Bio evaluates every settled trajectory through three nested views of the **same run**. No gate gets a fresh model attempt:

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>Gate</th><th>What it asks</th><th>What it still cannot establish</th></tr></thead>
  <tbody>
    <tr><td><strong>G0: outcome-only</strong></td><td>Is the scored task outcome non-inferior, and is at least one raw efficiency measure better?</td><td>Whether the answer came from valid execution or recoverable evidence.</td></tr>
    <tr><td><strong>G1: receipt-valid</strong></td><td>Does G0 pass, and are exit states, artifact locators, schemas, hashes, and replay records valid?</td><td>Whether material scientific claims are supported by the executed evidence.</td></tr>
    <tr><td><strong>G2: evidence-constrained</strong></td><td>Does G1 pass, do scientific invariants hold, can every material claim be traced to typed executed evidence, and is the frozen scientific score non-inferior?</td><td>Universal biological truth or external replication.</td></tr>
  </tbody>
</table>
</div>

Throughout this article, a **scientifically valid output** is a run that passes G2. A **candidate-only hard violation** occurs when the candidate fails a frozen protocol or scientific-reviewability condition that its paired identity control passes. For example, if the candidate saves a call by finalizing before a required executed artifact exists, that is a candidate-only hard violation. A shared pre-treatment failure affecting both arms is not relabeled as a candidate defect.

### P1 asks: is a cheaper trajectory necessarily an equivalent one?

P1 added a prompt-level rule asking the agent to work only on unresolved evidence requirements and stop when the evidence appeared complete. We tested it against the identity harness on six task identities—two each in scRNA, bulk RNA-seq, and GWAS—with three paired repeats per task.

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>Endpoint per scheduled task</th><th>Identity harness</th><th>P1</th><th>P1 change</th></tr></thead>
  <tbody>
    <tr><td>Settled provider cost</td><td>¥0.070735</td><td>¥0.060776</td><td class="metric-good">−14.08%</td></tr>
    <tr><td>Provider requests</td><td>5.111</td><td>4.333</td><td class="metric-good">−15.22%</td></tr>
    <tr><td>Input tokens</td><td>28,398.9</td><td>24,711.7</td><td class="metric-good">−12.98%</td></tr>
    <tr><td>Output tokens</td><td>1,742.1</td><td>1,419.1</td><td class="metric-good">−18.54%</td></tr>
    <tr><td>Scientific rubric score</td><td>0.944</td><td>0.778</td><td class="metric-bad">−0.167</td></tr>
    <tr><td>Scientifically valid output rate</td><td>0.833</td><td>0.722</td><td class="metric-bad">−0.111</td></tr>
    <tr><td>New candidate-only hard violations</td><td>—</td><td>5</td><td class="metric-bad">Reject</td></tr>
  </tbody>
</table>
</div>

If we had ranked candidates only by cost, requests, or tokens, P1 would have won. But after dividing total cost by the number of scientifically valid outputs, the apparent 14.08% advantage shrank to **0.86%**. Most of the “saving” came from producing fewer acceptable results.

<div class="formula-card formula-card-compact" aria-label="Cost per valid iteration equals total provider cost divided by the number of scientifically valid outputs.">
  <div class="formula-main">cost per valid iteration = total provider cost / number of G2-valid outputs</div>
</div>

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/false-winner-frontier.svg' | relative_url }}" alt="A capability-versus-cost diagram showing P1 below the scientific capability floor despite being cheaper." width="1440" height="900" loading="lazy">
  <figcaption>P1 is the cheapest wrong winner: it crosses the cost target by falling below the scientific capability floor.</figcaption>
</figure>

P1 turned the original motivation into a concrete research problem. Lower per-round cost is useful only when it increases the number of **valid** iterations:

<div class="formula-card" aria-label="Choose the harness with the lowest expected round cost subject to scientific score and valid-output rate declining by no more than 0.03, no new hard violations, and complete claim-to-execution traceability.">
  <div class="formula-label">Echo-Bio optimization problem</div>
  <div class="formula-main"><i>h</i><sup>★</sup> = arg min<sub><i>h</i> ∈ H</sub> 𝔼[ C<sub>round</sub>(<i>h</i>) ]</div>
  <div class="formula-rule"></div>
  <div class="formula-constraints">
    <span>scientific-score change ≥ −0.03</span>
    <span>scientifically-valid-output-rate change ≥ −0.03</span>
    <span>new candidate-only hard violations = 0</span>
    <span>material claims remain traceable to executed evidence</span>
  </div>
</div>

Here, **h** is one candidate harness and **H** is the preregistered candidate set. The optimization target is the expected cost of a research round. The remaining lines are feasibility constraints: a cheaper harness is not eligible if scientific capability, execution integrity, or evidence traceability deteriorates beyond the frozen boundary.

The promotion rule is deliberately ordered:

1. reject any candidate that introduces a new hard violation;
2. reject any candidate whose scientific score or G2-valid rate falls by more than the frozen 0.03 non-inferiority margin;
3. require every material claim to remain traceable to executed evidence;
4. **only among the remaining feasible candidates**, rank efficiency and require at least the preregistered practical saving.

In short: scientific equivalence defines the feasible region; cost chooses a winner only inside that region.

## Echo-Bio: optimize, verify, then promote

Echo-Bio has three functional layers.

### 1. A common evaluation substrate

Both arms use the same provider interface, runtime, observer, artifacts, replay semantics, and hidden verifier:

<div class="formula-card formula-card-compact" aria-label="The baseline is the Biomni research policy plus a common substrate and identity harness. The candidate is the same policy and substrate plus one treatment.">
  <div class="formula-main">Baseline = research policy + common substrate + identity harness</div>
  <div class="formula-main">Candidate = research policy + common substrate + one harness treatment</div>
  <div class="formula-rule"></div>
  <div class="formula-main">Harness effect = paired candidate outcome − paired baseline outcome</div>
</div>

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/common-evaluation-substrate.svg' | relative_url }}" alt="A paired experiment in which baseline and candidate share provider, model, runtime, observer, replay, and verifier, while differing only in one harness treatment." width="1440" height="900" loading="lazy">
  <figcaption>The shared substrate makes the treatment identifiable. It is not counted as an Echo-Bio efficiency gain.</figcaption>
</figure>

For a concrete scRNA task, both arms might receive the same count matrix and question—say, identify cell-type-specific differential expression between two conditions. Both must execute code in the same environment, persist the same classes of QC and result artifacts, and link the final claims back to those artifacts. The candidate may change how the loop represents “what remains to be done,” but it cannot change the data, scientific dependencies, model, execution budget, or grading rules. That is why a difference in cost can be interpreted as a harness effect rather than an easier task or a more permissive evaluator.

### 2. An evidence-aware verifier

Scientific validity is decomposed into checks with different authority:

<div class="table-scroll">
<table class="evidence-table">
  <thead><tr><th>Level</th><th>Question</th><th>Example</th></tr></thead>
  <tbody>
    <tr><td><span class="level-chip">L0</span></td><td>Did execution and persistence succeed?</td><td>Exit state, artifact hash, schema, and locator are valid.</td></tr>
    <tr><td><span class="level-chip">L1</span></td><td>Did domain-specific structural invariants hold?</td><td>For scRNA: required QC fields exist, failed cells and doublets are handled, donor/sample structure is preserved, and multiple-testing correction can be recomputed.</td></tr>
    <tr><td><span class="level-chip">L2</span></td><td>Does a material claim follow from linked evidence?</td><td>A reported count or effect equals a typed value produced by execution.</td></tr>
    <tr><td><span class="level-chip">L3</span></td><td>Does the result satisfy the scientific rubric?</td><td>Method choice, limitations, and causal language satisfy the frozen rubric or are routed for expert review.</td></tr>
  </tbody>
</table>
</div>

The evidence path is explicit:

<div class="provenance-chain" aria-label="Material claim to executed evidence provenance chain">
  <span>Material claim</span><i>→</i><span>Typed evidence</span><i>→</i><span>Artifact locator</span><i>→</i><span>Execution receipt</span><i>→</i><span>Code + data digest</span>
</div>

These checks do not prove that a cell annotation is biologically true, that a statistical model is the only appropriate one, or that a result will generalize to another cohort. They establish bounded execution, provenance, contract satisfaction, and rubric compliance. “Scientifically valid” in this article therefore means **valid under this frozen operational verifier**, not “certified biological truth.” Confounding, biological interpretation, and external replication remain scientific work.

### 3. A governed candidate search

Candidate mechanisms are not accumulated into one ever-changing prompt. Each mechanism receives its own lineage:

1. mine public development traces for a specific source of overhead or failure;
2. formulate one mechanism and freeze its predicted benefit and rejection conditions;
3. use settled historical trajectories for zero-cost replay qualification;
4. run an interleaved paired prospective screen on the public search set;
5. reject the mechanism or freeze it before any later evaluation.

The capability gate is evaluated before the efficiency ranking. This prevents the search from learning that incomplete scientific work is an easy way to save money.

## From failed prompts to a typed control plane

**P1 asked whether a general “stop when evidence looks complete” prompt could cut cost.** It could, but partly by omitting necessary scientific work. That ruled out unconstrained early stopping.

**P4 then asked whether a narrower natural-language instruction could remove only redundant work.** In a three-pair prospective screen, scientific validity was preserved and no new hard violation appeared—but cost **increased by 29.53%** and action count increased.

Trace inspection showed the model closing gaps serially, reconfirming completed work, and spending turns explaining that it was finished. Because the entire transcript returned in later calls, extra actions also inflated later input context.

The conclusion was not merely “write a better prompt.” It was that stopping is a control decision and should not depend on persuasive prose.

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/failure-driven-lineage.svg' | relative_url }}" alt="A timeline from P1 to P4 to P5: P1 is cheaper but scientifically degraded, P4 preserves validity but increases cost, and P5 moves termination into a deterministic typed control plane." width="1600" height="940" loading="lazy">
  <figcaption>Each successor answers a concrete failure in the previous lineage. P5 does not silently retain the rejected prompts.</figcaption>
</figure>

**P5 therefore asked whether completion could become a deterministic state transition rather than another topic for the model to discuss.** It moved completion out of the model transcript and introduced:

- one content-addressed resource receipt shared by both paired arms;
- a public compiler that translates the task's execution contract into typed evidence requirements without exposing expected answers or hidden rubric values;
- a deterministic reducer that reads real execution receipts and returns only `continue_batch`, `finalize_now`, or `fail_closed`.

If execution evidence is missing and budget remains, the reducer returns every unresolved requirement together. If the execution contract is complete, it prevents further scientific actions. If the budget is exhausted first, it fails closed.

<ol class="decision-steps">
  <li><strong>fail_closed</strong><span>The execution ledger is inconsistent, the budget is exhausted before the contract is complete, or required state cannot be recovered.</span></li>
  <li><strong>continue_batch</strong><span>Evidence is incomplete; return the complete unresolved set in one typed batch rather than asking the model to rediscover gaps one by one.</span></li>
  <li><strong>finalize_now</strong><span>All public execution requirements are satisfied; close scientific action generation and move to bounded finalization.</span></li>
</ol>

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/p5-control-plane.svg' | relative_url }}" alt="P5 control plane showing typed evidence requirements and executed artifacts feeding a deterministic reducer that chooses continue batch, finalize now, or fail closed." width="1500" height="980" loading="lazy">
  <figcaption>P5 decides whether the loop should continue. The same independent verifier still evaluates the resulting science in both arms.</figcaption>
</figure>

This separation is essential:

- **P5 completion control:** Has the public execution contract been completed well enough to stop?
- **Scientific verification:** Is the result executable, traceable, scientifically reviewable, and within the allowed claim boundary?

The expected efficiency mechanism is equally specific. A prose-only controller can spend one turn announcing completion, another reconfirming it, and then carry both turns back into every later prompt. P5 stores completion as compact typed state outside the conversational transcript. When the contract is complete, it removes those post-completion actions and their recursively accumulated context; it does **not** remove the scientific actions needed to produce the evidence.

## P5: prospective evidence on a fixed public task bank

We evaluated the frozen P5 mechanism on a fixed public search-dev bank. The schedule contained 20 paired tasks spanning scRNA, bulk RNA-seq, GWAS, and proteomics. One pair failed shared resource selection before either arm began; it was not retried or replaced. The primary analysis therefore contained 19 paired attempts.

Both arms shared the same provider/model policy, scientific runtime, resource receipt, evidence-checking substrate, execution limits, and verifier. The only arm difference was P5 evidence-complete termination enforcement.

<aside class="reader-note">
  <strong>How to read these numbers.</strong> This was a selected public <em>development</em> matrix, not an external benchmark or a sealed generalization test. Repeats of one task are correlated, so the primary uncertainty interval used 20,000 task-cluster, domain-stratified bootstrap replicates rather than treating every trajectory as an independent biological task. The single shared pre-arm failure contributes to the scheduled-task accounting but not to the 19-pair treatment estimand.
</aside>

### Main results

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>Endpoint</th><th>P5 vs identity harness</th><th>Interpretation</th></tr></thead>
  <tbody>
    <tr><td>Settled provider cost</td><td class="metric-good">−21.96%<br><small>task-cluster 95% interval: 14.50% to 33.05%</small></td><td>Passed the preregistered 5% practical threshold.</td></tr>
    <tr><td>Provider requests</td><td class="metric-good">−17.24%</td><td>Fewer model calls per scheduled task.</td></tr>
    <tr><td>Input tokens</td><td class="metric-good">−26.94%</td><td>Less accumulated context returned to the provider.</td></tr>
    <tr><td>Output tokens</td><td class="metric-good">−7.87%</td><td>Moderate reduction in generated text.</td></tr>
    <tr><td>Executed scientific actions</td><td class="metric-good">−24.29%</td><td>Fewer actions after typed evidence completion.</td></tr>
    <tr><td>Tool wall time</td><td class="metric-good">−11.87%</td><td>Lower measured execution time.</td></tr>
    <tr><td>Scientifically valid attempts</td><td>P5: 18/19<br>Identity: 16/19</td><td>No aggregate validity loss in this public matrix.</td></tr>
    <tr><td>New candidate-only hard violations</td><td>0</td><td>Passed the public-matrix hard-failure gate.</td></tr>
  </tbody>
</table>
</div>

Two sensitivity views address the concern that failed runs can stop early and look artificially cheap:

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>Restricted comparison</th><th>Number of paired attempts</th><th>P5 cost reduction</th></tr></thead>
  <tbody>
    <tr><td>Only pairs where both arms produced scientifically valid outputs</td><td>16</td><td class="metric-good">15.25%</td></tr>
    <tr><td>Only pairs where both arms were hard-violation-free</td><td>15</td><td class="metric-good">15.08%</td></tr>
  </tbody>
</table>
</div>

The result is aggregate, not deterministic. Five P5 trajectories cost more than their paired identity run, and repeated executions of the same task sometimes changed direction. The appropriate conclusion is therefore:

> On this fixed public biomedical task matrix, typed evidence-aware termination reduced aggregate provider cost, requests, context, and executed actions without an observed aggregate loss of scientifically valid outputs.

It is not evidence that P5 saves cost on every run, proves scientific truth, generalizes to every biomedical task or model, or already improves a complete multi-generation RSI process.

## A1 asks: can one bounded model-mediated stage be removed entirely?

P5 reduced redundant work *inside* the research loop. We then asked whether a complete model call could be removed before that loop begins. **A1 is a separate, bounded successor experiment; it is not folded into the P5 effect estimate.**

A Biomni-style agent normally asks the provider to select tools and libraries from the available inventory. For task types whose required capability profile is already registered, this repeatedly pays a model to rediscover a bounded resource set. **A1** replaces that step with a deterministic registry compiler:

- input: the public task type, execution contract, and frozen resource inventory;
- output: a content-addressed resource receipt consumed by the unchanged P5 research loop;
- safety boundary: an unknown task profile, inventory drift, or unreplayable projection fails closed instead of guessing;
- treatment isolation: provider/model, downstream prompts, P5 stopping control, scientific runtime, and evidence verifier remain shared between arms.

<figure class="post-figure post-figure-wide">
  <img src="{{ '/assets/images/blog/echo-bio/a1-resource-compiler.svg' | relative_url }}" alt="A1 replaces a paid provider resource-selection call with a deterministic registry compiler for registered task profiles before both arms enter the same P5 and evidence-constraint loop." width="1500" height="880" loading="lazy">
  <figcaption>A1 removes a complete provider call for registered profiles; it does not compress or weaken the downstream scientific evidence.</figcaption>
</figure>

The final public search-dev analysis contained eight task identities and twelve paired trajectories across scRNA, bulk RNA-seq, GWAS, and proteomics. Tasks—not repeated trajectories—received equal weight in the task-cluster analysis. The interval below comes from the exact domain-stratified task-cluster resampling distribution; it is uncertainty over this small selected development set, not a population-level claim about all biomedical tasks.

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>A1 endpoint</th><th>Registry compiler vs provider selector</th><th>Interpretation</th></tr></thead>
  <tbody>
    <tr><td>Full-pipeline provider cost</td><td class="metric-good">−38.901%<br><small>task-cluster 95% interval: 33.085% to 44.643%</small></td><td>Strong public search-dev efficiency signal for registered profiles.</td></tr>
    <tr><td>Scientifically valid outputs</td><td>A1: 12/12<br>Provider-selector control: 10/12</td><td>No observed capability loss in this matrix.</td></tr>
    <tr><td>Resource-selection provider calls</td><td class="metric-good">0 for A1</td><td>The targeted call was deterministically eliminated.</td></tr>
    <tr><td>Total provider requests</td><td class="metric-good">−1.125 per task<br><small>95% interval: −1.313 to −0.938</small></td><td>The removed stage reduced complete-pipeline request count.</td></tr>
    <tr><td>New A1 hard violations</td><td>0</td><td>Passed the public-matrix hard-failure gate.</td></tr>
    <tr><td>Downstream-only cost</td><td>−3.373%<br><small>95% interval: −6.844% to 12.824%</small></td><td>Downstream-only non-inferiority was not established.</td></tr>
  </tbody>
</table>
</div>

This is a solid but intentionally bounded conclusion: **for registered task profiles under a frozen resource inventory, A1 eliminated the resource-selection provider call, preserved observed scientific validity, and reduced full-pipeline cost.** It does not establish open-domain resource selection, nor does it imply that downstream reasoning became 38.9% cheaper. The large full-pipeline effect includes the model call that A1 was designed to remove.

## What this means for agent builders and scientists

For an agent builder, the main lesson is to measure *where* cost enters the trajectory. P5 targets repeated post-completion interaction; A1 targets a whole model-mediated stage that can be compiled safely for registered profiles. Neither result says that prompts, observations, or resource selection should always be shorter.

For a biomedical researcher, the important protection is that efficiency is evaluated only after the run remains inspectable: code actually executed, artifacts can be reopened, quantitative claims point to typed evidence, and domain checks still pass. A cheaper narrative without that chain is not counted as a cheaper scientific iteration.

For RSI research, Echo-Bio supplies a missing denominator. The relevant quantity is not simply tokens per answer, but cost per verifier-accepted research iteration. Only after this denominator is stable does “more rounds under the same budget” become a meaningful experimental claim.

## What this changes about the RSI story

Echo-Bio does not yet claim that more affordable rounds automatically produce a better final discovery. That requires a separate long-horizon experiment in which multiple agents receive the same total budget and are compared after different numbers of propose–execute–verify iterations.

What the present evidence establishes is the prerequisite for that experiment:

<div class="table-scroll">
<table class="result-table">
  <thead><tr><th>Question</th><th>Current evidence</th></tr></thead>
  <tbody>
    <tr><td>Can a cost-only harness search select the wrong biomedical research agent?</td><td>Yes. P1 was 14.08% cheaper but crossed the scientific capability boundary and introduced five new hard violations.</td></tr>
    <tr><td>Does scientific research require a stronger equivalence criterion than final-answer success?</td><td>Yes. Promotion depends on execution receipts, artifact integrity, claim–evidence traceability, and scientific rubric checks.</td></tr>
    <tr><td>Can a deterministic harness mechanism reduce the cost of a valid research iteration?</td><td>P5 provides a positive public search-dev signal: −21.96% aggregate provider cost with 18/19 scientifically valid attempts versus 16/19 for identity.</td></tr>
    <tr><td>Can a whole model-mediated stage be removed safely?</td><td>For registered profiles, A1 removed the resource-selection provider call and reduced full-pipeline cost by 38.901%, with scientifically valid outputs of 12/12 versus 10/12 for the provider-selector control.</td></tr>
    <tr><td>Has Echo-Bio proven end-to-end RSI improvement?</td><td>No. Equal-total-budget multi-round improvement remains the next causal experiment.</td></tr>
  </tbody>
</table>
</div>

The core contribution is thus not “a cheaper agent” in isolation. It is a formulation and evaluation method for **cost-efficient scientific recursion**: optimize the harness only inside a region where scientific execution and evidence remain valid.

> More iterations are useful only when they remain scientific iterations.

## Reproducibility and code-release boundary

The P5 and A1 results in this article are bound to source tag `echo-bio-a1-task-diversity-stage-a-result-v1` (commit `abea8cb`). The sanitized, user-facing `0.1.0rc1` code release is available at [Zimiao1025/Echo-Bio](https://github.com/Zimiao1025/Echo-Bio). It was prepared from the frozen source rather than from the later exploratory working tree.

The public package is intentionally narrower than the research workspace. It includes the user runtime, evidence and trace infrastructure, verifier components, the reviewed Biomni hook patch, and an initial provider guide for **DeepSeek V4.1 Flash** (`deepseek-flash`). It excludes experiment-reproduction scripts, benchmark corpora, credentials, private or sealed evaluators, human-review material, authorization records, raw paid-response receipts, and internal reader/reviewer notes.

This release boundary matters scientifically as well as operationally: readers should be able to reproduce the public method without accidentally receiving the hidden evaluation material that defined the original experimental separation.

---

### References

1. Huang, K. et al. [“Biomni: A General-Purpose Biomedical AI Agent.”](https://www.biorxiv.org/content/10.1101/2025.05.30.656746v1) bioRxiv, 2025. [Code](https://github.com/snap-stanford/Biomni).
2. Jin, R. et al. [“STELLA: Self-Evolving LLM Agent for Biomedical Research.”](https://arxiv.org/abs/2507.02004) arXiv:2507.02004, 2025.
3. Xue, S. et al. [“ScienceBuddy: Recursive-in-Recursive Self-Improvement for Interactive Scientific Agents.”](https://arxiv.org/abs/2609.17523) arXiv:2609.17523, 2026.
4. Liu, H. et al. [“SoL-Pi: Recursively Scaling Auto-Research Loops for Efficient Agent Harness.”](https://arxiv.org/abs/2609.20519) arXiv:2609.20519, 2026. [Project page](https://nvlabs.github.io/SoL-Pi/).

### Suggested citation

Li, H. “Echo-Bio: When Cost Minimization Fails for Biomedical Research Agents.” 2026.<br>
[Article](https://zimiao1025.github.io/blog/2026/10/27/echo-bio/) · [Code](https://github.com/Zimiao1025/Echo-Bio)
