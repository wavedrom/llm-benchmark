# WaveDrom LLM Benchmark — Execution Plan

## Rationale

### Why this benchmark

Engineering diagrams are specifications, not decoration. A diagram can render cleanly while placing a transition on the wrong edge, reversing a register field, or connecting the wrong logic inputs. Syntax validity and visual plausibility therefore cannot establish engineering correctness. General coding or conversational scores do not directly answer whether a model can produce and understand these artifacts.

WaveDrom offers a practical evaluation target: a compact text representation that can be parsed, rendered, and checked against explicit engineering requirements. Timing, register, and schematic diagrams exercise different capabilities—temporal reasoning, bit-layout reasoning, and connectivity/logic reasoning—while sharing a common documentation workflow. Testing both generation and comprehension helps distinguish producing plausible notation from understanding its meaning.

The intended contribution is a reproducible, domain-specific capability profile with inspectable failures. Its value does not depend on claiming to be the first hardware-diagram benchmark or on treating results as evidence of general hardware-design competence.

### Why accuracy versus cost

Engineers need to choose a usable model, not merely identify the highest score. A small accuracy gain may require substantially more inference expense or latency; a cheaper configuration may be sufficient for a particular task class. The [Vals Tax Agent Bench](https://www.vals.ai/benchmarks/tax_agent_bench) provides the presentation reference: show these trade-offs directly rather than collapse them into an arbitrary value score.

Use fully correct task rate versus cost per task as the primary overview, with an observed Pareto frontier and separate latency view. Per-task cost captures differences in response length and reasoning usage that token unit prices alone miss. Per-type breakdowns prevent an aggregate score from hiding important weaknesses. Gallery drill-down makes the trade-off concrete: users can inspect what failed before deciding whether a cheaper model is acceptable. These measurements do not include downstream human review or repair cost unless a later study explicitly measures it.

### Why this execution approach

- **Deterministic semantics before scale:** a large dataset with an unreliable grader produces misleading rankings. Canonical representations, executable assertions, and mutation tests establish trust before expanding coverage.
- **Small, three-type MVP:** 30 tasks exercise the complete pipeline and expose type-specific evaluation problems without committing to a large corpus. This validates the benchmark machinery, not a definitive market ranking.
- **Text-only, bounded features first:** separating diagram reasoning from image recognition keeps early failures interpretable. Explicit feature limits make reliable grading feasible for a solo maintainer.
- **Broad access without infrastructure sprawl:** an aggregator supplies breadth; direct APIs provide selected reference runs. Exact endpoint/configuration records preserve attribution. Pre-release partnerships can extend coverage but must not block delivery.
- **Baseline before documentation uplift:** separate tracks distinguish model capability under a fixed prompt contract from gains due to additional reference material. Publish regressions and null results as well as improvements.
- **Open artifacts and versioned results:** transparent tasks, scorers, raw responses, pricing assumptions, and disclosed sponsorship make findings auditable. New held-out task families are needed for future contamination-sensitive claims after public release.

Success means helping engineers make defensible accuracy–cost choices and understand model failure modes—not maximizing model count, producing a favorable ranking, or obtaining privileged access.

## 1. Goal and working assumptions

Help hardware/software engineers answer: **Which models can generate and understand correct WaveDrom diagrams, where do they fail, and how much does documentation in context help?**

Benchmark three diagram types: digital timing, register/bit-field, and schematic. Evaluate engineering meaning, not similarity to one preferred JSON encoding or rendered image.

Deliverables, in dependency order:

1. Versioned task dataset and trustworthy deterministic scorers.
2. Reproducible model runner and failure reports.
3. Accuracy–cost overview for WaveDrom.com, with leaderboard and diagnostic gallery drill-down.
4. Controlled documentation-uplift study.
5. External benchmark integrations and community runs.

Planning assumptions, carried forward from the most developed draft rather than newly confirmed:

- Primary audience: engineers choosing AI tools.
- Solo maintainer, approximately 10–12 hours/week.
- Text-only first release: natural language and WaveJSON inputs; rendered output for validation and inspection.
- Neutral benchmark first; improvements to WaveDrom documentation are a separate experiment.
- Node.js implementation, matching this repository's `package.json` and WaveDrom's JavaScript ecosystem. Add another runtime only for a demonstrated need.
- Schedule: 18 working weeks plus 2 weeks of contingency. Phase gates override dates.

Repository currently contains drafts, a minimal README, and package metadata; no implemented harness or usable test suite. Everything below is planned work.

## 2. Normalization decisions

| Conflicting or overlapping draft ideas | Execution decision |
|---|---|
| 90 versus 100–200 versus 200–300 tasks | 30-task MVP → 40-task pilot → 90-task v0.1 target. Increase scale only after scorer validation. |
| Timing, registers, schematics, and protocols treated as four domains | Three diagram types. Protocol is a content tag, primarily on timing tasks, not a fourth type or duplicate score contribution. |
| Analysis, synthesis, accuracy, prompting styles mixed into task families | Separate diagram type, task family, difficulty, content tags, input modality, and evaluation track. |
| Python grader versus JavaScript renderer | Node.js core; renderer adapter, canonicalizers, and scorers behind independent interfaces. |
| Exact JSON, bit-array matching, image similarity, or LLM judge | Parse + benchmark profile validation + render check + task-specific semantic assertions. Canonical representations support comparison; no JSON/pixel matching as primary score. |
| Alleged official schema and unrestricted WaveJSON support | Verify pinned implementations and documentation first. Define an explicit benchmark-supported subset; do not assume a complete official schema exists. |
| Strict JSON versus JSON5/JavaScript object notation | Strict JSON output contract for baseline. This is a benchmark constraint, not a claim about everything WaveDrom accepts. |
| Historical named models and illustrative leaderboard numbers | Select available models at run time; record exact IDs. Publish measured results only. |
| Zero-shot, few-shot, spec injection, agentic feedback | Baseline first; documentation tracks later. Prompting styles are evaluation tracks, not separate task families. |
| Public reproducibility versus permanently private tests | Hold test tasks back during development; publish frozen v0.1 tasks with results. Label subsequent runs as public-test evaluation. No submission service in MVP. |
| Novelty claims and detailed related-work assertions | Treat draft references as research leads. Verify papers, dates, methods, and numbers before citing; make no “first benchmark” claim without evidence. |
| Website, spec work, integrations all started together | Finish executable benchmark before publication UI; integrations follow stable release. |

## 3. Task model and scope

### Independent dimensions

- **Diagram type:** `timing`, `register`, `schematic`.
- **Task family:** `synthesis`, `comprehension`, `repair`, `consistency`.
- **Difficulty:** D1 local syntax/basic meaning; D2 multiple constraints; D3 coupled constraints or protocol corner cases within the supported subset.
- **Content tags:** clock/reset, valid/ready, UART, field layout, combinational logic, etc.
- **Track:** baseline, existing-docs-in-context, optimized-spec-in-context; agentic feedback deferred.
- **Modality:** text initially; image input deferred.

Protocol synthesis is `synthesis` with protocol tags and executable protocol assertions. Difficulty is assigned from declared reasoning requirements, not changed after observing model rankings.

| Family | Input | Required output | Scoring |
|---|---|---|---|
| Synthesis | Unambiguous engineering description | WaveJSON object | Required structure and semantic assertions; render success |
| Comprehension | WaveJSON + factual question | Task-defined JSON answer | Exact typed values or deterministic checks derived from reference semantics |
| Repair | Broken or incorrect WaveJSON + requested fix | Corrected WaveJSON object | Target assertions plus preservation of explicitly unaffected properties |
| Consistency | Prose requirements + WaveJSON | Structured discrepancy records | Match expected discrepancy tuples; report precision/recall |

Repair includes editing only where the requested change is explicit. No subjective “minimal edit” score based on JSON text distance. Free-form explanation quality and diagram-type selection remain research extensions.

### Supported feature envelope

Define and test exact syntax in Phase 0; initial intent:

- **Timing:** named signals, basic logic/unknown/high-impedance states, simple clocks, bus values/labels, bounded timelines. Add explicit `period`/`phase` cases only after conformance tests. Begin protocol checking with valid/ready and UART scenarios.
- **Registers:** fixed-width layouts, names, reserved ranges, bit positions, widths, and explicitly encoded attributes. Do not infer read-only/write-one-clear behavior from geometry or labels alone.
- **Schematics:** small acyclic combinational circuits expressible by the pinned supported renderer: gates, muxes, small adders. Confirm the actual schematic dialect/API first. Exclude latches, sequential feedback, analog behavior, and unsupported components.

No full AXI, PCIe, CDC correctness, arbitrary setup/hold analysis, or general EDA verification claim. A finite diagram can establish only the properties encoded and observable within its scenario.

## 4. MVP: smallest useful end-to-end benchmark

**MVP = 30 verified tasks, all three diagram types, synthesis and comprehension, deterministic grading, two real model runs, local report/gallery.** Target: end of week 8.

| Diagram type | Synthesis | Comprehension | Total |
|---|---:|---:|---:|
| Timing | 6 | 4 | 10 |
| Register | 6 | 4 | 10 |
| Schematic | 6 | 4 | 10 |
| **Total** | **18** | **12** | **30** |

Mostly D1/D2 tasks. Timing includes at least one valid/ready and one UART synthesis scenario, using explicit assumptions and executable checks.

MVP acceptance:

- One documented command evaluates saved responses without network access; a separate command queries configured providers.
- All 30 reference responses pass; every task has at least one deliberately wrong response that fails the intended check.
- Each canonicalizer has positive equivalence tests and negative semantic-mutation tests.
- At least two exact model IDs evaluated on the same tasks and baseline track, one requested completion per task.
- Raw outputs, normalized responses, diagnostics, run manifests, latency, usage, and available cost data saved.
- Local static report includes an accuracy–cost scatter plot and accessible table, per-type/per-family pass rates, and representative reference-versus-output diagrams; comprehension cases show expected versus returned answers. Two-model MVP plots are functional previews, not broad market comparisons.
- Every observed scorer/model disagreement reviewed and resolved or explicitly excluded before freezing results.

MVP excludes public deployment, hidden-test infrastructure, free-form LLM judging, image input, agentic retries, few-shot optimization, broad protocol coverage, integrations, and publication-quality novelty claims.

**Scope fallback:** narrow supported features, reduce provider count, or postpone later families before weakening grading. If schematic support cannot be validated, ship a clearly labeled two-type technical preview—not the defined three-type MVP.

## 5. Evaluation contract

### Validation pipeline

1. **Capture:** preserve raw model output and provider status unchanged.
2. **Parse:** require exactly one JSON value of the task's expected shape. No `eval`, silent repair, or removal of surrounding prose for the baseline score. Optional lenient extraction is diagnostic only.
3. **Validate:** check the benchmark-supported profile and semantic prerequisites. Unsupported features receive an explicit failure reason, not an invented interpretation.
4. **Render:** diagram-output tasks must render successfully using pinned packages/configuration. Rendering is necessary, not sufficient: ignored invalid keys or missing meaning must still fail other checks.
5. **Canonicalize:** convert supported diagram content into a type-specific intermediate representation (IR).
6. **Check:** evaluate required task assertions, protocol rules, or answer checks; emit localized differences and failure categories.

Comprehension answers use their answer schema and deterministic checker; no diagram-render gate applies to the answer itself. Reference input diagrams are validated during dataset build.

### Type-specific semantics

- **Timing IR:** signal identities, time intervals, transitions, clock edges, bus values and labels. Use exact rational time or a bounded integer grid derived from allowed periods/phases. Do not flatten clocks or unknown/high-impedance states into binary samples. Verify repetition, gaps, labels, and edge behavior against pinned renderer/source tests; visually plausible strings are not automatically equivalent. Set explicit comparison horizon and treatment of unspecified intervals.
- **Register IR:** total width and fields `(lsb, width, name, encoded attributes)`. Check positions, overlap, range, reserved areas, and required labels. Pin bit ordering and multi-row layout conventions.
- **Schematic IR:** ports and labeled connectivity graph plus evaluable combinational semantics. Use exhaustive truth tables below a declared input-count bound. If the prompt requires a topology/component, enforce that separately; functionally equivalent circuits are not sufficient for topology-constrained tasks.

Compare requested properties, not incidental styling. Allow valid alternative solutions. Exact semantic equality is appropriate only when the prompt specifies the complete behavior; otherwise use constraints. Ordering/grouping/annotations matter only when explicitly required.

Protocol checks specify preconditions, sample edges, transaction completion, and observation horizon. Reject vacuous passes such as a valid/ready trace containing no required transfer. Do not penalize legal unconstrained behavior.

### Metrics and denominators

Primary metric: **task pass@1**, requiring all mandatory checks to pass for the first completion. A repair task's provided broken input is part of the task, not permission for iterative retries.

Report a profile:

- Parse, profile-validation, and render pass rates for applicable tasks.
- Task pass rate by diagram type, task family, difficulty, and protocol tag.
- Diagnostic semantic scores: timing interval/edge mismatches, field mismatches, logic counterexamples, failed protocol assertions, discrepancy precision/recall.
- Latency, token usage, and cost where available; unknown values remain unknown.
- Failure counts: invalid JSON, unsupported feature, render failure, timing offset, missing/wrong label, bit-order/width error, connectivity/function error, protocol violation, incorrect answer.

Overall accuracy for the overview chart: arithmetic mean of the three per-type task pass rates on the fixed release manifest. Show the per-type profile alongside it. Do not double-count protocol-tagged tasks or mix evaluation tracks. Clearly state task counts and weighting; no composite “syntax points + semantic points” score.

Invalid model outputs, refusals, and output truncation fail the task. Provider/network failures are infrastructure errors: retry under a declared bounded policy, retain attempts, and mark unresolved runs incomplete rather than silently excluding items or publishing a comparable overall score. Never retry a completed wrong answer in baseline.

Show uncertainty with sample sizes and template-family-aware bootstrap intervals where feasible. Small pilot rankings are exploratory. Repeated completions diagnose variability; report them separately, without selecting the best output for pass@1. Temperature zero is not a determinism guarantee.

### Accuracy–cost presentation and accounting

Design reference: [Vals Tax Agent Bench](https://www.vals.ai/benchmarks/tax_agent_bench). Its page describes a Pareto overview of accuracy, cost, and latency, supported by a leaderboard with accuracy, cost/test, token pricing, and duration. Adopt that decision-oriented presentation, not its tax-specific scoring or LLM-judge methodology.

**Primary user question: which model configuration delivers sufficient correctness within my budget?** Page order: accuracy–cost overview → sortable leaderboard → per-type/task breakdown → reference-versus-output gallery → methodology.

#### Chart contract

- **Y-axis:** accuracy = fully correct task pass@1, 0–100%; never substitute parse/render success or partial credit without changing the label.
- **X-axis:** mean inference cost per benchmark task in USD; logarithmic by default, linear toggle. Upper-left is better. Cost/task reflects generated token volume, not just advertised dollars per million tokens.
- **Point:** one model/version + host + inference configuration, within one benchmark version, split, and evaluation track. Use stable developer colors and explicit labels/tooltips; no meaning conveyed by color alone.
- **Pareto frontier:** highlight configurations for which no other eligible point is at least as accurate and no more expensive, with one strict improvement. Compute from unrounded values. Label as the observed frontier, not proof of statistical superiority; show accuracy intervals and sample counts where supported.
- **Filters:** diagram type, task family, difficulty, protocol, model/developer, and hosted/open-weight status. Track and dataset version are explicit selectors, never silently combined. Recompute accuracy, cost, and frontier on the same selected task IDs and weights for every model. Missing/incomplete runs remain visible in the table, outside the comparable frontier.
- **Interaction:** hover/focus shows accuracy, USD/task, task count, latency, endpoint, reasoning setting, pricing date/basis, and run date. Click opens filtered results and gallery. Provide keyboard navigation and downloadable JSON/CSV.
- **Latency view:** optional X-axis switch to mean wall-clock seconds/task; table also reports p50/p95. Measure from first request through final response, including bounded transport retries/backoff but excluding local scheduler wait and offline scoring. Record concurrency; do not present this as intrinsic model speed.
- Start with all eligible models selected. If label density needs a curated view, make selection rules and hidden-model count visible. Model filtering may change the displayed frontier, so label its scope.

#### Cost contract

Store a per-task cost ledger: input tokens, cache-read/write tokens where applicable, billed output/reasoning tokens, tool charges if any, rate source/date/currency, pricing tier, provider fees, and billed retry attempts. Separate token categories without double-counting reasoning already included in output usage.

Default chart basis: **estimated list-price inference cost at run time**, using the observed usage and applicable endpoint/cache/batch rates, before promotional credits or negotiated discounts. Badge estimates and cache/batch tiers. Keep actual billed cost and out-of-pocket spend as separate fields; sponsored credits must not turn an expensive endpoint into a zero-cost benchmark winner. A cold-cache or common-price-date repricing study is a separately labeled derived view, never an overwrite of historical results.

For selected tasks, compute accuracy and mean task cost using identical weights. Overall uses equal weights across diagram types, then equal weights within each type; filtered views use equal task weights unless an explicit release rule says otherwise. Include costs of wrong, refused, or truncated responses and any billed transport retries—not only successful tasks. Incomplete runs or unknown costs do not enter the accuracy–cost frontier. Genuine zero-cost points need a separate marker/linear view, not an arbitrary epsilon on the log axis.

Record evaluator/rendering infrastructure cost separately from model inference. Future agentic tracks include all inference turns and tool charges, while baseline remains one completion. Self-hosted GPU estimates need declared hardware, utilization, and amortization assumptions; show them separately from hosted API pricing by default.

Leaderboard columns: model/configuration, accuracy with count/interval, USD/task, latency mean/p50/p95, timing/register/schematic accuracy, endpoint, run date, and pricing basis. Token unit prices remain secondary metadata. Optional cost per passed task = total selected cost / passed task count for an equally weighted view; undefined with zero passes. This is an observed workload ratio, not a promise that retries yield a correct answer at that price.

Acceptance tests: chart/table totals match run artifacts; filtering uses identical cohorts; weights align across axes; dominance and ties use unrounded values; unknown/zero cost, zero passes, and incomplete runs handled explicitly. Keep dataset/scorer versions and pricing snapshots in exports. MVP needs basic plot/table and cost provenance; richer filters, frontier, intervals, and gallery linking land by public release.

### Scorer trust and safety

- Freeze task assertions before scored runs; do not tune them to favor observed model outputs.
- Test equivalent encodings, single-property mutations, boundary cases, and unsupported features.
- Cross-check reference IR against independent hand-worked expectations and rendered inspection, not only the same generator used to produce tasks.
- Bound output size, diagram complexity, clock expansion, and rendering time/memory. Run rendering in isolated workers without network or secrets; never execute model-generated code. Sanitize or isolate SVG in the gallery.
- Judge models are excluded from primary v0.1 scoring. Add subjective evaluation only with published rubrics and human calibration.

## 6. Dataset and reproducibility

### Task record

Versioned JSON/JSONL records contain:

- Stable `id`, `task_version`, `template_family`, `split`, diagram type, task family, difficulty, and tags.
- Prompt, input artifacts, answer/output contract, assumptions, supported feature profile.
- Reference response, independently checked expectations/assertions, and negative fixtures.
- Provenance, licensing, source references where needed, review status, generator version/seed where applicable.

Keep reference answers, assertions, split metadata, and reviewer notes outside provider-visible request payloads. Add tests for prompt construction to prevent answer leakage.

### v0.1 target

Expand MVP to 40 tasks for pilot, then 90 if quality gates hold:

| Diagram type | Synthesis | Repair | Comprehension | Consistency | Total |
|---|---:|---:|---:|---:|---:|
| Timing | 14 | 6 | 8 | 2 | 30 |
| Register | 14 | 6 | 8 | 2 | 30 |
| Schematic | 14 | 6 | 8 | 2 | 30 |
| **Total** | **42** | **18** | **24** | **6** | **90** |

Aim for 10 tasks per difficulty per diagram type, but do not manufacture difficulty or fill unsupported cells to satisfy a quota. Prefer simple generators for D1/D2, manually authored D3 cases, and manual review of every released task.

Split target: 30 public development tasks and 60 evaluation tasks held back until release. Allocate by template family before model-driven prompt/scorer iteration, not by random parameter instance. MVP development tasks may need replacement to achieve family separation. Publish actual split counts and coverage.

Freeze evaluation tasks before final runs. Publish them with v0.1 for offline reproduction; future uncontaminated evaluation requires new held-out families/versioned releases. Procedural variants and canaries do not prove absence of training contamination. No private-test leaderboard service is promised.

Use original/paraphrased protocol scenarios with version/section references, not redistributed specification prose or figures. Confirm dataset license separately from the repository's MIT code license. No telemetry or scraped user examples without suitable rights and privacy review.

### Run manifest and artifacts

Record dataset/scorer versions and hashes, git commit, Node and renderer versions, skin/config, provider and exact model ID, run timestamp, prompts, supported sampling parameters, output limits, seeds when supported, retry policy, and track. For local models include checkpoint, quantization, and runtime.

Store raw responses and per-task results so scorer changes can be applied without new API calls. Version re-scored results rather than overwriting published scores. Cache keys include model/config, full request, and task version; cached completions are not independent repeat samples. Exclude credentials and private provider metadata from published artifacts.

Proposed layout and CLI contracts (to implement, not existing commands):

```text
schemas/                 Task, answer, run, and supported-profile schemas
src/adapters/            Provider and renderer adapters
src/ir/                  Timing, register, schematic canonicalizers
src/scorers/             Task checks, semantic diffs, protocol assertions
src/cli/                 Dataset validation, generation runs, offline scoring
datasets/                Manifests, dev/test tasks, provenance
tests/fixtures/          Reference, equivalent, malformed, and mutated outputs
runs/                    Local raw outputs, manifests, scored results; ignored by default
reports/                 Generated report data and static gallery
docs/                    Methodology, dataset card, reproducibility, LLM reference
```

```text
npm test
npm run dataset:validate -- --manifest <manifest>
npm run bench:run -- --manifest <manifest> --model <config> --track baseline
npm run bench:score -- --run <run-directory>
npm run bench:report -- --run <run-directory>
```

### Model access and coverage strategy

**Decision: aggregator for breadth, direct APIs for selected reference runs, hosted open-weight endpoints for additional families. Treat pre-release access as a partnership opportunity, never a release dependency.**

The practical evaluation pattern is a shared harness over several access routes, with credentials, quotas, versions, and endpoint behavior managed separately. A unified SDK or OpenAI-compatible API reduces integration work; it does not grant access or guarantee identical inference behavior.

#### Access routes

| Route | Use here | Advantages | Constraints |
|---|---|---|---|
| Aggregator: OpenRouter | First broad-coverage adapter | One account/API across many model families | Pin underlying provider/endpoint; disable fallbacks. Catalog entries and provider duplicates are not distinct model families. |
| Hugging Face Inference Providers | Alternative or supplemental open-model coverage | Unified billing/client support across inference providers | Hub model availability does not imply hosted inference availability. Select provider explicitly, not automatic routing. |
| Direct model-vendor APIs | Selected frontier baselines and launch-day additions | Direct model identity, native parameter support, provider relationship | Separate accounts, spending limits, eligibility, regions, and quotas; snapshots may still be mutable. |
| Hosted open-weight providers: Together AI, Fireworks AI, DeepInfra, etc. | Fill catalog gaps; compare accessible open models | No GPU operations; broad open-model selection | Verify checkpoint, quantization, chat template, and hosting details where disclosed; unknowns remain unknown. |
| Cloud catalogs: AWS Bedrock, Google Vertex AI, Azure AI model offerings | Use existing credits/accounts or required regional deployment | Central billing and organizational controls | Model access approvals, deployment/region differences, quota setup. Not the default solo-project route. |
| Self-hosted open weights on local/rented GPUs | Later reproducibility checks or unavailable checkpoints | Pin weights, tokenizer, template, inference runtime | GPU cost and operations; model licenses/gating; do not make this an MVP requirement. |
| Provider/community-contributed runs | Expand coverage after artifact format stabilizes | Sponsor pays inference; otherwise inaccessible models possible | Require full manifest/raw outputs; independently re-score; label contributor-run versus maintainer-run. Never request shared API keys. |

Start with **one aggregator plus one direct vendor adapter**, not accounts at every provider. Add a generic configurable OpenAI-compatible adapter where adequate, plus native adapters where request/response or reasoning controls differ. Keep adapters swappable without changing task semantics.

#### Breadth without misleading comparisons

- MVP remains two models; pilot remains 3–4 configurations. After calibration, target **10–20 distinct model configurations** if budget permits, across several independent developers, price/size tiers, and both proprietary and open-weight models. Breadth is a stretch goal, not a v0.1 release gate.
- Select the release roster using declared coverage criteria, availability, and budget before final evaluation—not whichever models score well on a preliminary subset.
- Count model families, versions, inference configurations, and hosting endpoints separately. The same checkpoint on five hosts is not five independent models.
- Label each row as **model/version + host/endpoint + inference configuration + benchmark track**. Record reasoning mode/budget, quantization, template, and system-prompt transformations when known. An API result measures the served system, not isolated weights.
- Match baseline instructions and task requirements across models. Preserve required native role formats; disclose adaptations. Do not silently enable schema-constrained decoding for only some models. If tested, make constrained decoding a separate track.
- Record unsupported sampling parameters rather than dropping them silently. If a model requires its own sampling defaults, disclose the exception; there is no universal temperature setting for every model.
- Keep provider-routing controls explicit. For OpenRouter, use a specific provider endpoint with fallbacks disabled and require parameter support where applicable. Request failures are better than unreported provider/model substitution. Hugging Face likewise supports explicit provider choice. [A1–A3]
- “Latest,” automatic routing, free-tier aliases, and anonymous preview identities are unsuitable for a version-stable reference row. If evaluated, label them mutable/anonymous and record run time and returned identity; never infer a private model's identity from behavior.
- Compare a small public development subset through aggregator and direct API for selected anchor models to detect adapter/configuration differences. This is a sanity check, not proof that hosted implementations are identical.

Maintain a model registry containing requested/returned IDs, developer, access route, endpoint, availability status, context/output limits, supported parameters, pricing snapshot, quotas, data policy, and any publication restrictions. Secrets live outside the registry. Catalog refresh discovers candidates; human approval controls paid runs.

#### Budget, funding, and operating policy

Estimate each campaign before execution:

```text
requests = tasks × model configurations × tracks × repetitions
cost = sum(input_tokens × input_rate + billed_output_tokens × output_rate) / 1,000,000
       + separately priced services and GPU time, if any
```

Example workload, not a price quote: 90 tasks × 20 configurations × one track × one completion = 1,800 requests. Three documentation tracks make 5,400. Include billable reasoning tokens, transport retries, and repeats in the budget. Use a small calibration batch to estimate token distributions; cap expensive models individually. Batch APIs can reduce expense where offered, but require compatible deadlines and artifact capture.

Funding priority: small approved self-funded baseline → provider/aggregator credits or sponsorship → suitable research grants → community runs. Credit support must not determine rankings or provide sponsors editorial control. Disclose credits, donated compute, and partnerships; record list-price estimates separately from actual out-of-pocket cost.

- **OpenAI Researcher Access Program:** documented support is up to $1,000 in credits for publicly available models, for eligible responsible-AI research. It is not a general pre-release entitlement. Confirm eligibility and current application terms. [A4]
- **Anthropic External Researcher Access Program:** targeted at safety/alignment research; its documentation explicitly excludes nonpublic/experimental model access and distinguishes limited pre-deployment partnerships. Hardware diagram benchmarking is not automatically eligible. [A5]
- **Evaluation-development grants:** Anthropic has published a third-party evaluation initiative. Treat this as a lead to assess against current priorities/intake, not promised funding or early access. [A6]
- Provider startup, academic, and open-source support programs are additional leads only when this project's actual status meets eligibility. Do not rebrand capability benchmarking as safety research merely to qualify.

Check terms covering automated benchmarking, publication of scores and outputs, data use, retention, and model licenses before each route is approved. Consumer chat subscriptions do not automatically include API access; browser automation and shared accounts are not the benchmark's access strategy. Free tiers are useful for development but should not underpin release schedules or held-out-data protection.

#### Upcoming models: two distinct access levels

1. **Public preview or launch-day API:** realistic near-term goal. Watch official release notes/model catalogs and opt into relevant developer notifications. Maintain a compatibility smoke test and ready-to-run frozen benchmark. Public preview endpoints can change or disappear; record preview status and do not merge their results with a later stable release. Google's model documentation explicitly distinguishes stable, preview, latest, and experimental variants. [A7]
2. **Confidential pre-release model:** typically a selective relationship with a model lab, evaluation team, or an established external evaluator. Payment for ordinary API access does not buy this. Public safety-testing calls sometimes exist, but are purpose-specific and time-limited. OpenAI's cited safety-testing call closed January 10, 2025; it is precedent, not a current application route. [A8]

No generally open, guaranteed pre-release route for this hardware capability benchmark was established by this research. Do not wait for one.

**Partnership path:**

- After MVP, prepare a short evaluator packet: WaveDrom maintainer credentials; intended engineering audience; supported tasks; deterministic scorer and audit evidence; measured baseline failures; public examples; runtime/token budget; expected turnaround; data/publication policy.
- Position the value precisely: hardware-documentation generation and comprehension with machine-checkable timing, field-layout, and logic errors—not generic chatbot rankings or an unsupported claim to measure broad hardware design ability.
- Approach developer relations, model evaluation/research teams, and existing evaluator organizations through official contact channels or warm introductions. Request either direct API access, a provider-run artifact submission, or integration of this benchmark into their own evaluations. None guarantees acceptance or permission to share access.
- Ask first for credits or launch-day access, then pre-release access once the benchmark has demonstrated reliable, useful findings. Be ready to return a concise failure report within a mutually agreed window.
- Negotiate model/version disclosure, evaluation window, quotas, data retention/training use, permitted artifacts, named participants, and publication rights before receiving confidential material. Prefer a defined embargo end and freedom to publish unfavorable findings. Avoid agreements allowing only favorable scores to be released; disclose unavoidable publication restrictions.
- Isolate embargoed tasks, outputs, credentials, and CI artifacts from the public repository/site. Keep preview evaluation work separate from final public release results; rerun the publicly released endpoint when feasible.
- Provider-run results stay visibly labeled until independently reproduced. NDA-only results cannot carry the same public reproducibility claim as released artifacts.

Suggested outreach request:

> I maintain WaveDrom and am building an open benchmark for LLM generation and comprehension of timing, register, and schematic diagrams. Our pilot uses deterministic semantic checks and produces actionable failure reports. Could your evaluation or developer-relations team discuss API credits, launch-day access, or a pre-release evaluation partnership? We can provide the harness, public sample tasks, a fixed token budget, and a mutually agreed reporting schedule. We seek permission to publish all aggregate outcomes after an agreed embargo, with confidential artifacts handled separately.

Access does not imply permission to train on outputs, redistribute weights, share credentials, or publish confidential results. Confirm each permission explicitly.

#### Access milestones

| Existing phase | Added access work | Completion evidence |
|---|---|---|
| Phase 0 | Compare aggregator/direct coverage; approve accounts, terms, retention policy, and spending cap | Initial model registry, chosen routes, no paid calls before approval |
| Phase 2 | Implement aggregator + direct adapters; run two-model MVP | Pinned endpoints, recorded effective parameters, complete artifacts |
| Phase 3 | Prepare evaluator packet; contact suitable labs/providers; apply for eligible credits | Public pilot report and outreach/application log; acceptance not required |
| Phase 4 | Expand to budgeted roster; publish access provenance and sponsorship | Comparable complete runs; unavailable models recorded rather than silently replaced |
| After release | Monitor new public APIs; cultivate selective pre-release partnerships | Versioned launch-day runs; separate preview/embargo process if access is granted |

#### Access references

Program terms, quotas, availability, and intake status change; recheck before applying or running. References support route descriptions, not a guarantee of admission or continuous access.

- **A1:** [OpenRouter model catalog/API](https://openrouter.ai/docs/guides/overview/models).
- **A2:** [OpenRouter provider routing, fallback and data-policy controls](https://openrouter.ai/docs/guides/routing/provider-selection).
- **A3:** [Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers/index); [Lighteval provider-backed evaluation example](https://huggingface.co/docs/lighteval/use-inference-providers-as-backend).
- **A4:** [OpenAI Researcher Access Program](https://openai.com/form/researcher-access-program/).
- **A5:** [Anthropic External Researcher Access Program](https://support.claude.com/en/articles/9125743-what-is-the-external-researcher-access-program).
- **A6:** [Anthropic third-party model evaluation initiative](https://www.anthropic.com/news/a-new-initiative-for-developing-third-party-model-evaluations).
- **A7:** [Gemini model versions and preview lifecycle](https://ai.google.dev/gemini-api/docs/models).
- **A8:** [OpenAI early access for safety testing — historical call](https://openai.com/index/early-access-for-safety-testing/).

## 7. Phased execution

### Phase 0 — Charter and feasibility | weeks 1–2

Work:

- Confirm assumptions, budget, licensing approach, reviewer availability, and minimum protocols.
- Choose initial aggregator/direct access routes, approve terms and data policy, and create the model registry described in section 6.
- Pin Node.js, WaveDrom, and any separate register/schematic renderer dependencies; verify actual APIs and licenses.
- Render and independently check two tiny examples per diagram type, including clock and bus-label behavior.
- Specify supported feature profiles, task/answer schemas, scoring rules, and evaluation track defaults.
- Verify essential related-work references before using them in public positioning.

Deliverables: methodology draft, dependency lockfile, feature-support matrix, six feasibility fixtures, cost/run policy.

**Exit gate:** all three types have a demonstrated rendering and semantic-checking path; unresolved limitations explicitly excluded. If feasibility fails, revise scope before building a large corpus.

### Phase 1 — Trustworthy evaluator | weeks 3–5

Work:

- Implement parse/profile/render gates, isolated renderer adapter, and structured diagnostics.
- Implement register IR first, bounded timing IR second, small combinational schematic IR third.
- Add valid/ready and UART scenario checkers, typed comprehension checks, and offline scoring CLI.
- Grow feasibility fixtures to 12 reference tasks, four per type; add equivalence and mutation tests.

Deliverables: offline grader, schemas, 12 tasks, conformance suite, CI with no provider credentials required.

**Exit gate:** 12/12 references pass, all intentional semantic mutations fail expected checks, equivalent supported representations pass, clean-checkout tests succeed.

### Phase 2 — End-to-end MVP | weeks 6–8

Work:

- Expand to the 30-task MVP matrix.
- Implement aggregator and direct-provider adapters, mock provider, caching, rate-limit handling, bounded transport retries, and manifests; pin endpoints and record effective model parameters.
- Run two available models on identical baseline requests; preflight a small batch against budget.
- Generate local JSON/HTML report with accuracy–cost plot/table, per-task cost ledger, and safe reference-versus-output gallery.
- Review all model failures for scorer defects; keep any pre-freeze reruns labeled development results.

Deliverables: executable MVP and first measured results.

**Exit gate:** all MVP acceptance criteria in section 4 met; offline re-scoring reproduces stored metrics.

### Phase 3 — Pilot and calibration | weeks 9–11

Work:

- Expand to 40 tasks by adding 10 repair tasks across types; introduce D3 cases within validated features.
- Run 3–4 model configurations if budget permits, including an open-weight model if accessible. Use exact current IDs, not draft-era model lists.
- Repeat a stratified subset three times to estimate output variability; keep first-attempt reporting intact.
- Blind-review at least 20 task/output pairs spanning types, pass/fail outcomes, and edge cases; include all suspected scorer disagreements.
- Seek a second hardware reviewer. If unavailable, label results maintainer-audited rather than independently validated.
- Freeze error taxonomy and revise ambiguous prompts; inspect ceiling/floor effects without optimizing for a preferred ranking.

- Prepare the evaluator partnership packet and begin targeted credit/access outreach; do not make acceptance a phase gate.

Deliverables: calibration report, corrected scorer tests, per-type failure analysis, release scope decision, evaluator packet and access outreach log.

**Exit gate:** at least 90% agreement on audited deterministic pass/fail decisions, exact numerator/denominator published, no known critical false pass. No requirement that models rank in a particular order. Low discrimination triggers future task design, not selective removal of inconvenient results.

### Phase 4 — v0.1 freeze and public release | weeks 12–18

Work:

- Expand toward the 90-task matrix; add structured consistency checks and only grader-supported protocol cases.
- Establish family-separated splits, review provenance, freeze tasks/scorer/track, and publish dataset card and methodology.
- Execute final budgeted baseline runs; preserve artifacts and report uncertainty/coverage. Expand toward 10–20 model configurations only if budget and access permit; publish hosting provenance and sponsorship.
- Build accuracy–cost overview and observed Pareto frontier, with accessible leaderboard, consistent cohort filters, cost provenance, and latency view as specified in section 5.
- Link chart/table selections to the static gallery with type/task/protocol/model filters, localized diffs, invalid-output views, and comprehension answer views.
- Add per-type profiles and overall accuracy, version/date labels, run instructions, and result submission requirements.
- Prepare WaveDrom.com integration; keep static deployment independently usable if site access is delayed.

Deliverables: tagged v0.1, released tasks and result artifacts, static report/gallery, reproducibility guide.

**Exit gate:** clean checkout can reproduce published scores from released raw outputs; tests and dataset validation pass; reported numbers match report artifacts; public static site available. Confirm WaveDrom.com placement separately if deployment is outside this repository.

**Contingency:** reserve weeks 19–20 for defects and release work. If capacity is insufficient, publish a smaller, explicitly scoped v0.1 dataset with revised frozen weights/counts—not unreviewed tasks or unimplemented categories.

### Phase 5 — Documentation uplift | after v0.1, approximately weeks 21–24

Work:

- Write `docs/wavedrom-for-llms.md` from verified renderer behavior and development-set failures; include valid examples and limits.
- Compare baseline, existing docs, and optimized reference using identical tasks, model configurations, attempt limits, and declared context budgets.
- Tune on development tasks only. Freeze documentation before evaluating new held-out task families if making generalization claims.
- Report paired per-task changes, context/token cost, regressions, and unchanged outcomes. Never put uplift-track results into the baseline ranking.

Deliverables: versioned LLM reference and reproducible ablation report. Schema export, cookbook, and index are optional follow-ups.

**Exit gate:** controlled study completed and all outcomes published. Positive uplift is not a release requirement.

### Phase 6 — External adoption and expanded research | after stable release

- Choose one integration target after checking its current API, license, and contribution policy; implement one adapter rather than several speculative integrations.
- Seek an independent run and publish a technical report with verified related work.
- Add image-to-WaveJSON, source-format conversion, free-form explanations, appropriate diagram selection, agentic feedback, or richer protocols as separately versioned tracks.
- Introduce a managed private evaluation service only if demand and maintenance capacity justify it.

**Exit gate:** at least one external end-to-end run; integration accepted or distributed as a documented standalone adapter.

## 8. Risks, guardrails, and open decisions

| Risk | Guardrail |
|---|---|
| Incorrect scorer creates convincing rankings | Independent expectations, equivalence/mutation tests, manual audits, versioned score corrections |
| Full WaveDrom semantics overwhelm scope | Explicit bounded feature profiles; unsupported cases rejected, never silently approximated |
| Public tasks leak into training | Label evaluation exposure; freeze/release versions; add new held-out families for future claims |
| Protocol or schematic claims exceed representation | Check only observable, encoded properties; state assumptions and exclusions per task |
| Hosted model drift or nondeterminism | Exact IDs/dates/configs, raw artifacts, repeat subset, no promise that fresh API calls reproduce bytes |
| API spending exceeds budget | Preflight token/cost estimate, explicit spending cap, cached responses, no automatic unbounded runs |
| Maintainer bias or weak expert review | Predeclared metrics, open scorer, publish disagreements/limitations, external review when possible |
| Scope or deployment delays | Gate-driven phases; static local artifacts first; cut breadth before validation |

Decisions to confirm in Phase 0; defaults allow planning but do not authorize paid runs:

1. **API budget and provider access:** choose a hard spending cap and initial two models before querying.
2. **Minimum protocols:** default valid/ready + UART; add I2C/SPI only after initial scorers are trusted.
3. **Schematic feature support:** confirm pinned dialect/renderer and bounded combinational subset.
4. **Review capacity:** name a second reviewer or disclose maintainer-only calibration.
5. **Dataset license and visibility:** approve release-time publication; choose dataset license after provenance review.
6. **Hosting:** confirm WaveDrom.com integration path; default to portable static report.

## 9. First implementation batch

Execute Phase 0 before expanding prose or collecting model outputs:

1. Verify rendering APIs and pin dependencies for all three diagram types.
2. Commit six tiny, manually checked fixtures; record unsupported/ambiguous features.
3. Define task schema, supported profiles, typed answer contract, and pass/fail result schema.
4. Add tests and a single-task offline evaluator; expand to 12 reference/mutation pairs.
5. Confirm budget and providers, then proceed toward the 30-task MVP.
