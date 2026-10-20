# HAAG: Human-Agent Accountability and Governance

An agentic governance framework. Responsibility can be delegated.
Accountability cannot.

**Version:** 0.2.0
**Status:** Public specification
**License:** CC BY 4.0 (see [LICENSE](../LICENSE))

## 1. Purpose and core premise

HAAG (Human-Agent Accountability and Governance) is an architecture-neutral
governance and assurance framework for systems that combine deterministic
software with probabilistic model or agent steps. Its name states its premise:
a person may delegate a task to an agent, but accountability for the outcome
stays with the person. Its four observation questions are remembered by the
mnemonic **NEAR**: Necessity, Evidence, Authority, Recovery.

Most AI governance frameworks treat governance either as organizational policy
(committees, checklists, risk tiers) or as a synchronous control point at a
single model or tool call. Agentic and multi-step workflows fail across broader
seams: models are invoked where deterministic code belongs, policies change
while long-running work is in flight, live model prose is mistaken for verified
evidence, and network or process failures leave paid external work in an
uncertain state.

HAAG supplies four observation questions. It creates no runtime dependency,
configuration surface, conformance level, or composite maturity score.

| Heading | Core question |
| :--- | :--- |
| **Necessity (N)** | Why does this step need a model, and how will its result be checked? |
| **Evidence (E)** | What happened, what supports the result, and can the responsible person inspect it? |
| **Authority (A)** | Who permits the action, what may it do, and where must it stop? |
| **Recovery (R)** | What is confirmed after a failure, and what is the next safe action? |

### 1.1 The governing test: human use and non-delegable accountability

Responsibility for a task can be delegated to an agent. Accountability for the
outcome cannot. Holding a person accountable for a system they cannot inspect,
bound, or recover turns the operator into a moral crumple zone—that is a system
defect, not governance. The test across all four headings is therefore **human
use**: the responsible person must be able to:

1. Understand the decision and inspect its basis.
2. Act within their authority and understand the scope and cost of what they
   permit.
3. Distinguish verified facts, model claims, and unknown states.
4. After an interruption or failure, identify what actually occurred and choose
   the next safe action.

They must be able to do all four **without reading internal source code or
reconstructing a workflow from raw logs**.

A supported surface that looks complete on paper but that a responsible person
cannot actually use is a finding against the principle governing that surface.
The [operator protocol](../protocol/h1-operator.md) tests this directly with an
unexposed human operator.

### 1.2 How to read this specification

Each principle in section 2 has three parts:

- **Property.** What must hold in the system. This is the requirement.
- **Observe.** What a reviewer inspects, or what fault test a reviewer runs, to
  check whether the property holds.
- **Example mechanisms.** Practical patterns teams use to satisfy the property.
  These are non-normative; HAAG requires none of them, and a mechanism's
  presence does not by itself prove the property holds.

#### Reviewer setup and outcomes

A reviewer applying this specification works from the workflow's step map (see
the [design walkthrough protocol](../protocol/design-walkthrough.md)), the
system's records and surfaces, its configuration and documentation, and a
disposable non-production instance where faults can be injected (killing a
worker at a named point, taking a dependency down, cutting the network). A
reviewer may read source code; the responsible operator in section 1.1 must
never need to.

Each principle checked produces one of three outcomes:

- **Finding.** The property does not hold. Any trade-off or reason the team
  accepted is recorded alongside the finding, not instead of it.
- **No finding.** The property was inspected or tested and holds for the steps
  and surfaces checked.
- **Unobserved.** The reviewer could not test the property (for example,
  because no disposable instance or fault injection point was available), and
  records why. Unobserved is a gap in testability, never a pass.

HAAG defines no compliance badge, certification, or composite score. Rolling
findings into a single number hides the single-boundary failure that matters.

## 2. The four headings

### 2.1 Necessity (N)

> Why does this step need a model, and how will its result be checked?

Model calls introduce non-determinism, latency, cost, and attack surface. Each
model duty must be justified, and what the model is trusted to produce must be
bounded and checked.

#### N1. Deterministic authority

**Property.** A model is never the sole authority for a value that
deterministic code can compute or verify. Calculations, aggregations, schema
routing, state transitions, and formatting are performed by code. Where a model
legitimately produces a number or invariant-bearing value (a rubric score, a
triage priority), code bounds and validates it before it drives a state
transition, a settlement, or a structured user-facing metric, and the system
records it as a model claim (see E2), not an observed fact. Numbers inside
free-text prose rendered in a claim region remain claims unless extracted into
a structured field.

**Observe.** Trace each numeric or invariant-bearing model output in the step
map to its first use. Any value that reaches a state transition, a settlement,
or a structured metric without passing a code check is a finding.

**Example mechanisms.** Range and enum validation on model outputs; code-side
recomputation of any total the model reports; invariants enforced in the state
store rather than in the prompt.

#### N2. Named, bounded duty

**Property.** Each model invocation has a named duty (for example, triage
classification, candidate summarization, rubric scoring) with declared inputs
and a declared output shape. A duty is skipped, not invoked, when no valid
input exists.

**Observe.** Match the workflow's declared model duties against the recorded
model invocations. An invocation that cannot be matched to a named duty, or
that ran on empty or invalid input, is a finding.

**Example mechanisms.** A duty registry; an input guard that short-circuits
before the model call; per-duty output schemas.

#### N3. Declared check and failure action

**Property.** Each duty names the check applied to its output and the action
taken when the check fails: reject, bounded retry, or escalation to a person.
A duty whose output flows onward without a named check, or whose failure
action is undefined, is a finding.

**Observe.** For each duty, check the validation and failure action both in
documentation and at runtime. An unbounded retry loop is a finding.

**Example mechanisms.** Schema validation with a rejection path; a retry
budget per duty; a human review queue for outputs that fail a confidence
threshold.

#### N4. Comparative value

**Property.** Before relying on a model duty, the team states what success
means for it and compares it against a simpler baseline (deterministic code, a
lookup, a human) on a fixed corpus of representative inputs, edge cases, and
failure modes (such as known facts, distractions, and contradictions). The
comparison record names the model, prompt, and check evaluated, and is rerun
when any of those change materially. A syntactically valid response is not
evidence that the step is useful.

**Observe.** Inspect the success criterion, the evaluation corpus, and the
comparison record for the configuration running in production. Relying on a
duty that has no comparison record, whose production configuration drifted
materially from the tested one, or that lost to the simpler baseline without a
stated reason is a finding. "No model needed" is a valid outcome.

**Example mechanisms.** A golden evaluation corpus checked into the repository;
a side-by-side run against a rule-based baseline; re-running the comparison in
CI when the model or prompt changes.

### 2.2 Evidence (E)

> What happened, what supports the result, and can the responsible person
> inspect it?

Logging events to an internal telemetry backend does not satisfy the Evidence
heading if the responsible person or a downstream consumer cannot inspect the
basis of a result through supported product surfaces.

#### E1. Labelled verification state

**Property.** At every boundary in the step map that carries external,
untrusted, or model-derived data (artifacts, evidence, model and tool outputs),
the data carries labels along four independent axes (pure control calls such as
authentication or quota checks are out of scope):

| Axis | Labels |
| :--- | :--- |
| Verification | unchecked or live; checked against a named policy revision |
| Transformation | original; redacted or cleaned, with provenance of the transformation |
| Disposition | admitted; suspect or quarantined (blocked from downstream model or user consumption) |
| Completeness | complete; truncated; unscored |

The axes are independent. Partial output is never silently promoted to
complete. A single status field that collapses the axes loses information and
is a finding.

**Observe.** Check which labels travel across each data boundary in the step
map and whether consuming components and surfaces read them. Truncate an output
on the disposable instance and confirm the surface displays the truncated state
rather than presenting it as complete.

**Example mechanisms.** Per-axis fields on the artifact record; a wrapper type
that cannot be unwrapped without reading the labels; UI badges driven directly
by the four labels rather than a collapsed status string.

#### E2. Observations distinct from claims

**Property.** The harness records every model invocation and every effectful
tool call as an observation. Harness-recorded facts (model invocations, tool
executions, HTTP effects, state transitions, timestamps, resource consumption)
are stored in a field or record type structurally distinct from model-generated
prose, inferences, and recommendations. Every surface that displays both
renders them in separately labelled regions.

**Observe.** Inspect the record schema and each result surface in the step map.
A model invocation with no harness record, a schema that mixes observations and
model claims, or a surface that renders model prose inside an observation
region is a finding.

**Example mechanisms.** Separate record types for observations and claims;
rendering model text in a visibly distinct region; refusing to interpolate
model output into observation fields.

#### E3. What a check proves

**Property.** Each check's result is recorded as what it proves and no more.

- A **schema check** proves structural shape, not accuracy.
- A **safety or content scan** proves the absence of matched patterns, not
  factual truth.
- A **cryptographic signature** proves byte integrity and provenance, not the
  truth of the signed claims.

A surface that presents one kind of check as if it established another is a
finding.

**Observe.** Inspect the wording of check results in records and UI surfaces.
A generic "Verified" badge without naming what was verified is a finding.

**Example mechanisms.** Check results stored as typed assertions ("schema:
valid", "scan: no match", "signature: valid for key K") rather than a single
boolean.

#### E4. Downstream lineage

**Property.** Downstream model steps (evaluators, judges, summarizers),
persistent agent memory or knowledge stores, and final user-facing artifacts
consume only checked copies, never raw unchecked live streams.

**Observe.** Check the input source of each downstream duty and memory write in
the step map, then verify on the disposable instance: run with distinguishable
checked and unchecked fixtures and confirm the downstream consumer reads the
checked revision, or alter the stored checked copy and confirm the consumer
either sees the change or halts on an integrity mismatch. Consuming the live
stream directly is a finding.

**Example mechanisms.** Downstream duties and memory writers read from the
artifact store by checked revision rather than subscribing to the producing
step's live output channel.

#### E5. Inspectable provenance

**Property.** An artifact or decision record links to its authoritative input
revision, its source receipts (including sources that were explicitly
uncaptured or unavailable), the policy and prompt versions active at the time,
and its integrity signatures. The responsible person can reach all of these
through supported product surfaces.

**Observe.** Starting from a result surface, follow the provenance links to
each item. Requiring internal tooling or raw log access is a finding. Because a
record that silently drops a failed source looks identical to a run with fewer
sources, test the uncaptured-source requirement by inducing a known-unavailable
source on the disposable instance (or during H1 case 3) and confirming the
missing source is explicitly recorded.

**Example mechanisms.** A provenance panel on the result surface; receipts
that record "not captured" as a first-class value; prompt and policy versions
pinned on the record.

#### E6. Retention for dispute resolution

**Property.** For each decision class the organization declares consequential,
the system retains the input revision, the request, the observed result, the
authority decision, and enough context for an independent reviewer to confirm
that the recorded output is the one that was acted upon. Exact model
reproduction is not required because model steps are non-deterministic. The
organization declares both the list of consequential decision classes and the
retention period.

**Observe.** Check the declared consequential decision classes and retention
period, then sample a past decision within that window and verify that the
retained record shows what was acted upon. Leaving consequential decision
classes undeclared is a finding.

**Example mechanisms.** A retention policy referenced from the product;
immutable decision records with a declared expiry.

### 2.3 Authority (A)

> Who permits the action, what may it do, and where must it stop?

Authority governs identity, scope of permission, resource and financial
boundaries, and how policies are evaluated and enforced across distributed
steps.

#### A1. Understandable scope of permission

**Property.** Before a person submits or approves work, the surface displays
four things: what data leaves the system and to where, who can see the result,
which effectful tools may run, and the resource or spending bound.
Authentication is not a substitute for this display.

**Observe.** Inspect the submission or approval surface as the responsible
person sees it. Requiring the person to infer any of the four items from config
files or external documentation is a finding.

**Example mechanisms.** A scope summary card on the approval prompt; a declared
spending ceiling shown before submission.

#### A2. Enforcement points distinct from policy evaluators

**Property.** Enforcement points (workflow guards, tool dispatchers, storage
boundaries, provider adapters) are distinct from policy evaluators (local rule
libraries, external governance gateways, human approval queues). Untrusted or
model-derived data cannot supply the target or parameters of an effectful tool
call without passing a deterministic policy check at the enforcement point or a
revision-bound approval (A7). Where multiple evaluators are combined, the
contract states check ordering, timeout handling, conflict resolution, and
which baseline safety controls no evaluator can override.

**Observe.** Inspect the contract between enforcement points and evaluators,
then run two checks on the disposable instance: take an evaluator down and
confirm the enforcement point fails closed or enters an operator-wait state
(R1); and attempt an effectful tool call whose target is supplied by untrusted
input outside policy. Passing traffic while the evaluator is unreachable, or
allowing untrusted text to redirect an effectful tool call, is a finding.

**Example mechanisms.** A policy decision point separate from the tool
dispatcher; capability-tagged data values that block untrusted strings from
populating tool targets; a hard-coded deny list that evaluators cannot relax.

#### A3. Per-boundary authority and policy revision

**Property.** In a multi-step or long-running workflow there is no assumed
whole-run policy snapshot. Each boundary records its own admission decision and
the exact policy or capability revision read at execution time. Work is refused
at pickup when a capability was disabled after the workflow was scheduled, and
refusal emits no external traffic.

**Observe.** On the disposable instance, schedule a multi-step run, then revoke
a capability or change the policy revision before the next step picks up.
Confirm the step records the new revision, refuses execution, and emits no
outbound network traffic.

**Example mechanisms.** Revision read and recorded at worker pickup; a
capability registry consulted per step rather than once at workflow submission.

#### A4. Authority does not amplify through delegation

**Property.** When a step delegates to a sub-step, sub-agent, or external job,
the delegate's permitted scope is a subset of the delegator's scope at
delegation time, and is re-evaluated at the delegate's own pickup under A3. A
delegate cannot acquire a capability, budget, visibility, or egress path that
its delegator lacked. Scopes are recorded in a structured form that can be
compared across hops, not only in free-text instructions.

**Observe.** Across a delegation chain, compare the recorded scopes at each
hop. Any widening of permission, or a scope recorded only as prose that cannot
be compared for subset, is a finding.

**Example mechanisms.** Scope tokens attenuated at each hop; the delegator's
budget reservation shared, not duplicated, by delegates.

#### A5. End-to-end resource and spending boundaries

**Property.** Passing an initial budget or quota check does not establish a
total run ceiling unless reservations account for retries, downstream
evaluation calls, auxiliary fetches, delegates (A4), and external jobs that
may keep running after cancellation. Permission to start a workflow is never
implicit authorization to exceed a declared limit.

**Observe.** From the step map, trace the worst permitted path (maximum
retries, all delegates, downstream evaluations, and external jobs that survive
cancellation). On the disposable instance, set a small budget limit, drive that
path, and compare total spend against the limit. Exceeding the limit, or
allowing an unbounded loop on the path, is a finding.

**Example mechanisms.** Atomic reservation against a ledger before each
effectful call; a shared per-run ceiling that retries and delegates draw from.

#### A6. Enforced versus advisory boundaries

**Property.** Documentation states the true runtime enforcement boundary. If a
role or agent is described as read-only or sandboxed, the runtime enforces
that restriction. Telling a model not to use a capability in a prompt, or
merely omitting a helper wrapper while leaving the underlying credential or
network path open, is not enforcement.

**Observe.** For each restriction described in documentation, attempt the
restricted action from that role on the disposable instance. If the attempt
succeeds, that is a finding. Reading the code is not a substitute for running
the attempt.

**Example mechanisms.** Filesystem and network policy applied at the OS or
container boundary; a read-only database or API credential rather than a prompt
instruction not to write.

#### A7. Approval bound to a revision

**Property.** A human approval binds to five things recorded at approval time:
the action, its target, the input revision, the policy revision in force, and
the authorizing principal. If any of the five changes before execution, the
approval is invalid and execution refuses until the changed action is approved
again.

**Observe.** On the disposable instance, approve an action, then alter its
input, target, or active policy before execution runs. If execution proceeds on
the stale approval, or if the approval record omits any of the five bindings,
that is a finding.

**Example mechanisms.** Approvals keyed by a content digest of action, target,
input revision, and policy revision; compare-and-swap check at execution time
verifying the approval's bindings still match the attempt.

### 2.4 Recovery (R)

> What is confirmed after a failure, and what is the next safe action?

Agentic workflows interact with external APIs, long-running jobs, and stateful
stores. HAAG treats interruption, partial completion, process replacement, and
ambiguous external effects as first-class governance concerns.

#### R1. Policy refusal distinct from operational failure

**Property.** A policy denial (disallowed input, insufficient authority,
exhausted budget) is never conflated with an operational failure (unreachable
policy store, timed-out scanner, network drop). Operational failure of a
required safety check fails closed or enters a bounded retry or operator-wait
state. It never passes as "clean".

**Observe.** On the disposable instance, take a required safety check or policy
store down and run a step that depends on it. Inspect the recorded outcome.
Recording the result as "clean", "passed", or as a policy denial is a finding;
it must be recorded as an operational failure that failed closed, queued a
bounded retry, or entered operator-wait.

**Example mechanisms.** Distinct outcome types for policy denial versus check
error; a scanner timeout that yields "unscanned" rather than "no match".

#### R2. No duplicate or stale effects

**Property.** Worker retries, replays, and stale attempts cannot duplicate a
logical effect or overwrite newer state. Consequential state transitions,
settlements, and artifact writes are guarded.

**Observe.** Run two tests on the disposable instance. First, replay a
completed step and confirm no second effect occurs. Second, start two attempts
of the same step, let the later attempt commit, and then let the earlier (now
stale) attempt finish and try to write. A duplicate effect or a stale write
overwriting newer state is a finding.

**Example mechanisms.** Compare-and-swap writes; revision guards; idempotency
keys derived from the logical effect or workflow step—never from the attempt
ID, since each retry mints a new attempt and an attempt-keyed request will not
deduplicate across retries.

#### R3. Durable resumption across process replacement

**Property.** When a worker or coordinator restarts after durably recording an
external job identity or a pending approval, the replacement process resumes
tracking the existing work. It does not issue a duplicate creation and it does
not lose the approval context.

**Observe.** Kill the worker process after the external job identity or pending
approval is recorded and before completion. If the replacement process spawns a
second external job or forces the operator to re-approve from scratch, that is
a finding.

**Example mechanisms.** Persisting the external job ID before the first status
poll; keying approval state by workflow step identity rather than worker
process memory.

#### R4. Preserved uncertainty (no invented zeros)

**Property.** The system never claims an effect did not happen when it cannot
know.

- If a request to an effectful external service is accepted, or times out
  after send, before a durable identifier is saved, the system records an
  **uncertain or possible-orphan** state. It never claims "no effect" or
  "zero cost".
- If a workflow is cancelled while external work is active, the system
  records **local termination** and **confirmed external terminal state**
  separately, and retains the resource and cost facts reported by the external
  provider at termination time.

**Observe.** Induce two faults on the disposable instance: kill the worker
between sending an external request and saving the returned ID, and cancel a
run while an external job is still executing. Recording "zero cost" or "no
effect" when the outcome was unknown, or dropping cost facts the provider
reported, is a finding.

**Example mechanisms.** A tri-state effect record (`confirmed`,
`confirmed-none`, `uncertain`); nullable or tagged cost fields that distinguish
`0` from `unknown`.

#### R5. Authorized operator path

**Property.** Flagging an uncertain or degraded state in a database is
incomplete Recovery under HAAG. An authorized person has a documented surface
to inspect the stuck or orphaned work, verify the external outcome, reconcile
resource records, and choose the next safe action.

**Observe.** Starting from an uncertain or orphaned record, follow the
documented operator path to resolution. Requiring raw database edits, internal
engineering tools, or source code access is a finding under the human-use test.

**Example mechanisms.** An operator queue of uncertain effects with a
reconcile action; a documented runbook linked directly from the degraded
record.

## 3. Positioning relative to other governance layers

Practitioners governing agentic systems encounter four distinct layers. They
solve different problems and are complementary:

| Layer | Examples | What it governs |
| :--- | :--- | :--- |
| **Threat taxonomies** | OWASP Top 10 for Agentic Applications | Names failure modes (goal hijack, tool misuse, memory poisoning, runaway loops) so teams know what to threat-model. |
| **Perimeter gateways** | GATES[^gates] | Governs the prompt pipe as traffic leaves and returns: redaction, template routing, caller auth, exchange logging, and output filtering. |
| **Agent-loop middleware** | Microsoft Agent Control Specification (ACS) | Hooks single model and tool calls inside the runtime loop (`pre_model_call`, `pre_tool_call`, `post_tool_call`). |
| **Workflow & human accountability** | **HAAG** | Governs the multi-step workflow, durable state across failures, and whether the accountable person can inspect, bound, and recover the system. |

**Gateways govern the pipe. Loop middleware hooks the call. HAAG governs the
workflow and the human decision.**

A system can address a threat checklist, pass every prompt through a perimeter
gateway, and wrap every tool call in middleware—and still produce HAAG findings
when:

- The model call should have been twenty lines of deterministic code (N1,
  N4).
- The result surface blends live unverified model tokens with verified
  citations so the reader cannot tell what was checked (E1, E2).
- A policy or capability is revoked while an asynchronous workflow is queued,
  an approval is reused after its target changed, or downstream calls exceed
  the initial budget reservation (A3, A5, A7).
- A worker crashes after starting a paid external job, or a user cancels
  mid-flight, leaving orphaned work with invented zeros and no operator
  recovery path (R3, R4, R5).

[^gates]: GATES™ AI Governance Framework — coined by Druhin Dhavala,
    October 10, 2026. <https://www.druhindhavala.com/ai-governance/gates-framework/>.
    Licensed under CC BY 4.0. GATES names five perimeter controls—Secure,
    Govern, Authenticate, Trace, and Enforce—applied in the order S, G, A, T,
    E. The summary above is a paraphrase, not a quotation.

## 4. Related documents

- [Design and workflow walkthrough protocol](../protocol/design-walkthrough.md):
  applying the four questions to a workflow design and runtime step map.
- [H1 blind human-operator protocol](../protocol/h1-operator.md): a bounded
  empirical exercise against a pinned release.
- [Implementations and history](../IMPLEMENTATIONS.md): where HAAG
  responsibilities are implemented and where the framework originated.

## 5. Change policy

Any change to a Property or Observe paragraph that alters what counts as a
finding is a normative change and is recorded in
[CHANGELOG.md](../CHANGELOG.md). Adding or revising example mechanisms or
explanatory prose is not a normative change. Principle identifiers (N1 through
R5) are frozen as of the 0.2.0 release and remain stable from then on. A
withdrawn principle keeps its identifier and is marked withdrawn.
