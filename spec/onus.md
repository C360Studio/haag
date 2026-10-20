# ONUS: Oversight, Necessity, Uncertainty, Scope

An agentic governance framework. Responsibility can be delegated.
Accountability cannot.

**Version:** 0.1.0-draft
**Status:** Public specification draft
**License:** CC BY 4.0 (see [LICENSE](../LICENSE))

## 1. Purpose and core premise

ONUS is an architecture-neutral governance and assurance framework for
systems that combine deterministic software with probabilistic model or agent
steps. Its name states its premise: a person may delegate a task to an agent,
but the onus for the outcome stays with the person. Its four observation
questions are remembered by the mnemonic **NEAR**: Necessity, Evidence,
Authority, Recovery.

Most AI governance frameworks treat governance either as organizational policy
(committees, checklists, risk tiers) or as a synchronous control point at the
model perimeter. Agentic and multi-step workflows fail across broader seams:
models are invoked where deterministic code belongs, policies change while
long-running work is in flight, live model prose is mistaken for verified
evidence, and network or process failures leave paid external work in an
uncertain state.

ONUS supplies four observation questions. It creates no runtime dependency,
configuration surface, conformance level, or composite maturity score.

| Heading | Core question |
| :--- | :--- |
| **Necessity (N)** | Why does this step need a model, and how will its result be checked? |
| **Evidence (E)** | What happened, what supports the result, and can the responsible person inspect it? |
| **Authority (A)** | Who permits the action, what may it do, and where must it stop? |
| **Recovery (R)** | What is confirmed after a failure, and what is the next safe action? |

### 1.1 The governing test: human use and non-delegable accountability

Responsibility for a task can be delegated to an agent. Accountability for the
outcome cannot. In law this is a non-delegable duty, and it is the premise of
every principle below. The test across all four headings is therefore
**human use**: the responsible person must be able to:

1. Understand the decision and inspect its basis.
2. Act within their authority and understand the scope and cost of what they
   permit.
3. Distinguish verified facts, model claims, and unknown states.
4. After an interruption or failure, identify what actually occurred and choose
   the next safe action.

They must be able to do all four **without reading internal source code or
reconstructing a workflow from raw logs**.

### 1.2 How to read this specification

Each principle in section 2 has three parts.

- **Property.** What must be true, stated so that a reviewer can observe
  whether it holds. This is the normative content.
- **Observe.** What a reviewer looks at to decide whether the property holds.
- **Example mechanisms.** Ways teams have satisfied the property. These are
  non-normative. ONUS requires none of them, and a mechanism's presence does
  not establish the property.

ONUS describes a system by its findings, never by a pass or fail.

- **Must** marks a property whose absence is a recorded finding.
- **Should** marks a property whose absence is recorded together with the
  trade-off the team accepted.
- There is no notion of ONUS compliance, certification, or score. Rolling
  findings into a single number hides the single-boundary failure that
  matters.

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
legitimately produces a number (a rubric score, a triage priority), code
bounds and validates it, and the system records it as a model claim (see E2),
not as an observed fact.

**Observe.** For each numeric or invariant-bearing output in the workflow,
identify the component that computes or verifies it. A model output that
flows into a state transition or a user-facing figure without a code check is
a finding.

**Example mechanisms.** Range and enum validation on model outputs; code-side
recomputation of any total the model reports; invariants enforced in the
state store rather than in the prompt.

#### N2. Named, bounded duty

**Property.** Each model invocation has a named duty (for example, triage
classification, candidate summarization, rubric scoring) with declared inputs
and a declared output shape. A duty is skipped, not invoked, when no valid
input exists.

**Observe.** The list of model duties for the workflow. An invocation that
cannot be matched to a named duty, or that runs on empty or invalid input, is
a finding.

**Example mechanisms.** A duty registry; an input guard that short-circuits
before the model call; per-duty output schemas.

#### N3. Declared check and failure action

**Property.** Each duty names the check applied to its output and the action
taken when the check fails: reject, bounded retry, or escalation to a person.
A duty whose output flows onward without a named check, or whose failure
action is undefined, is a finding.

**Observe.** For each duty, the check and the failure action as documented
and as implemented. A retry without a bound is a finding under both N3 and
A5.

**Example mechanisms.** Schema validation with a rejection path; a retry
budget per duty; a human review queue for outputs that fail a confidence
threshold.

#### N4. Comparative value

**Property.** Before a model duty is relied upon, the team has stated what
success means for it and compared it against a simpler baseline (deterministic
code, a lookup, a human) on a fixed corpus containing known facts,
distractions, and contradictions. A syntactically valid response is not
evidence that the step is useful.

**Observe.** The stated success criterion and the comparison record. "No model
needed" is a valid outcome of the comparison and is recorded as such.

**Example mechanisms.** A golden corpus checked into the repository; a
side-by-side run against a rule-based baseline; periodic re-comparison when
the model or prompt version changes.

### 2.2 Evidence (E)

> What happened, what supports the result, and can the responsible person
> inspect it?

Logging events to an internal telemetry backend does not satisfy the Evidence
heading if the responsible person or a downstream consumer cannot inspect the
basis of a result through supported product surfaces.

#### E1. Labelled verification state

**Property.** Data carries labels at every boundary along four independent
axes:

| Axis | Labels |
| :--- | :--- |
| Verification | unchecked or live; checked against a named policy revision |
| Transformation | original; redacted or cleaned, with provenance of the transformation |
| Disposition | admitted; suspect or quarantined (blocked from downstream model or user consumption) |
| Completeness | complete; truncated; unscored |

The axes are independent. Partial output is never silently promoted to
complete. A single status field that collapses the axes loses information and
is a finding.

**Observe.** At each boundary, which labels are carried and whether a consumer
can read them. A surface that shows a truncated or unscored result without
the label is a finding.

**Example mechanisms.** Per-axis fields on the artifact record; a wrapper type
that cannot be unwrapped without reading the labels; UI badges derived from
the labels rather than from a separate status.

#### E2. Observations distinct from claims

**Property.** Harness-recorded facts (tool executions, HTTP effects, state
transitions, timestamps, resource consumption) are structurally distinct from
model-generated prose, inferences, and recommendations, and remain distinct in
every surface a person sees.

**Observe.** A result surface or decision record. If a reader cannot tell
which statements the harness observed and which the model produced, that is a
finding.

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

**Observe.** The wording of check results in records and surfaces. "Verified"
without naming what was verified is a finding.

**Example mechanisms.** Check results stored as typed assertions ("schema:
valid", "scan: no match", "signature: valid for key K") rather than a single
boolean.

#### E4. Downstream lineage

**Property.** Downstream model steps (evaluators, judges, summarizers) and
final user-facing artifacts consume only checked copies, never raw unchecked
live streams.

**Observe.** The input source of each downstream duty. A judge that reads the
live stream rather than the stored checked copy is a finding.

**Example mechanisms.** Downstream duties read from the artifact store by
revision, not from the producing step's output channel.

#### E5. Inspectable provenance

**Property.** An artifact or decision record links to its authoritative input
revision, its source receipts (including sources that were explicitly
uncaptured or unavailable), the policy and prompt versions active at the time,
and its integrity signatures. The responsible person can reach all of these
through supported product surfaces.

**Observe.** Starting from a result, follow the links. A link that requires
internal tooling or raw log access is a finding under the human-use test.

**Example mechanisms.** A provenance panel on the result surface; receipts
that record "not captured" as a first-class value; prompt and policy
versions pinned on the record.

#### E6. Retention for dispute resolution

**Property.** For a consequential decision the system retains the input
revision, the request, the observed result, the authority decision, and
sufficient context for an independent reviewer to audit or reproduce the
outcome. The responsible organization sets the retention period and states it.
ONUS does not prescribe a duration, only that one is declared.

**Observe.** The stated retention period and whether a sampled past decision
can still be reconstructed within it.

**Example mechanisms.** A retention policy document referenced from the
product; immutable decision records with a declared expiry.

### 2.3 Authority (A)

> Who permits the action, what may it do, and where must it stop?

Authority governs identity, scope of permission, resource and financial
boundaries, and how policies are evaluated and enforced across distributed
steps.

#### A1. Understandable scope of permission

**Property.** Authentication alone does not prove that a caller understood the
consequences of an action. The submission or approval surface makes the
permitted scope clear: external data egress, who can see the result, which
effectful tools may run, and the resource or spending bound.

**Observe.** The approval surface as the responsible person sees it. If the
scope must be inferred from configuration or documentation elsewhere, that is
a finding.

**Example mechanisms.** A scope summary on the approval prompt; a declared
spending ceiling shown before submission.

#### A2. Enforcement points distinct from policy evaluators

**Property.** Enforcement points (workflow guards, tool dispatchers, storage
boundaries, provider adapters) are distinct from policy evaluators (local rule
libraries, external governance gateways, human approval queues). Where several
evaluators are combined, the contract states check ordering, timeout handling,
conflict resolution, and which baseline safety controls no evaluator can
override.

**Observe.** The contract between enforcement points and evaluators. An
evaluator that can disable a baseline control, or an undefined behaviour when
two evaluators disagree, is a finding.

**Example mechanisms.** A policy decision point separate from the dispatcher;
a documented precedence order; a hard-coded deny list that evaluators cannot
relax.

#### A3. Per-boundary authority and policy revision

**Property.** In a multi-step or long-running workflow there is no assumed
whole-run policy snapshot. Each boundary records its own admission decision and
the exact policy or registry revision read at execution time. Work is refused
at pickup when a capability was disabled after the workflow was scheduled, and
refusal emits no external traffic.

**Observe.** For a step executed after a policy change, the revision recorded
on the step. A step that ran under a revision older than the one in force at
its execution time is a finding.

**Example mechanisms.** Revision read and recorded at pickup; a capability
registry consulted per step rather than per run.

#### A4. Authority does not amplify through delegation

**Property.** When a step delegates to a sub-step, sub-agent, or external job,
the delegate's permitted scope is a subset of the delegator's scope at
delegation time, and is re-evaluated at the delegate's own pickup under A3. A
delegate cannot acquire a capability, budget, visibility, or egress path that
its delegator lacked.

**Observe.** For a delegation chain, compare the scopes at each level. Any
widening is a finding. A delegate that reads policy from a different source
than its delegator is a finding under A2.

**Example mechanisms.** Scope tokens attenuated at each hop; the delegator's
budget reservation shared, not duplicated, by delegates.

#### A5. End-to-end resource and spending boundaries

**Property.** Passing an initial budget or quota check does not establish a
total run ceiling unless reservations account for retries, downstream
evaluation calls, auxiliary fetches, delegates (A4), and external jobs that
may keep running after cancellation. Permission to start a workflow is never
implicit authorization to exceed a declared limit.

**Observe.** The declared limit and the sum of what the run could consume on
its worst permitted path. A gap is a finding.

**Example mechanisms.** Atomic reservation against a ledger before each
effectful call; a per-run ceiling that delegates draw from.

#### A6. Enforced versus advisory boundaries

**Property.** Documentation states the true runtime enforcement boundary. If a
role or agent is described as read-only or sandboxed, the runtime enforces
that restriction. Omitting a named helper tool is not enforcement.

**Observe.** For each described restriction, the mechanism that enforces it.
A restriction enforced only by prompt text or tool omission is a finding.

**Example mechanisms.** Filesystem and network policy applied at the process
boundary; a read-only credential rather than an instruction not to write.

### 2.4 Recovery (R)

> What is confirmed after a failure, and what is the next safe action?

Agentic workflows interact with external APIs, long-running jobs, and stateful
stores. ONUS treats interruption, partial completion, process replacement, and
ambiguous external effects as first-class governance concerns.

#### R1. Policy refusal distinct from operational failure

**Property.** A policy denial (disallowed input, insufficient authority,
exhausted budget) is never conflated with an operational failure (unreachable
policy store, timed-out scanner, network drop). Operational failure of a
required safety check fails closed or enters a bounded retry or operator-wait
state. It never passes as "clean".

**Observe.** The recorded outcome of a step whose safety check could not run.
"Clean" or "passed" in that case is a finding. This principle is shared with
Authority: a refusal is an authority decision, a failure is not.

**Example mechanisms.** Distinct outcome types for denied and errored; a
scanner timeout that yields "unscanned" rather than "no match".

#### R2. No duplicate or stale effects

**Property.** Worker retries, replays, and stale attempts cannot duplicate a
logical effect or overwrite newer state. Consequential state transitions,
settlements, and artifact writes are guarded.

**Observe.** Replay a completed step. A second effect, or an older result
overwriting a newer one, is a finding.

**Example mechanisms.** Compare-and-swap; revision guards; idempotency keys
derived from the attempt identity.

#### R3. Durable resumption across process replacement

**Property.** When a worker or coordinator restarts after durably recording an
external job identity or a pending approval, the replacement process resumes
tracking the existing work. It does not issue a duplicate creation and it does
not lose the approval context.

**Observe.** Kill the process after the identity is recorded and before
completion. A second external job, or an approval that must be re-requested
from scratch, is a finding.

**Example mechanisms.** Job identity persisted before the first poll; approval
state keyed by attempt identity rather than by process.

#### R4. Preserved uncertainty (no invented zeros)

**Property.** The system never claims an effect did not happen when it cannot
know.

- If a request to an effectful external service is accepted, or times out
  after send, before a durable identifier is saved, the system records an
  **uncertain or possible-orphan** state. It never claims "no effect" or
  "zero cost".
- If a workflow is cancelled while external work is active, the system
  records **local termination** and **confirmed external terminal state**
  separately, retaining all known resource and cost facts.

**Observe.** The recorded state after a send-then-crash or cancel-while-active
scenario. A zero where the system could not have known the value is a
finding.

**Example mechanisms.** A tri-state effect record (confirmed, confirmed-none,
uncertain); cost fields that distinguish "0" from "unknown".

#### R5. Authorized operator path

**Property.** Flagging an uncertain or degraded state in a database is
incomplete Recovery under ONUS. An authorized person has a documented surface
to inspect the stuck or orphaned work, verify the external outcome, reconcile
resource records, and choose the next safe action.

**Observe.** From the uncertain record, follow the documented path to
resolution. If the path requires internal tooling or source access, that is a
finding under the human-use test.

**Example mechanisms.** An operator queue of uncertain effects with a
reconcile action; a documented runbook reachable from the record.

## 3. Positioning relative to perimeter gateway frameworks

A perimeter gateway framework prescribes controls at the point where prompts
leave an organization and responses return. The usual set is: redact sensitive
data before the prompt leaves, validate templates and routing, authenticate
and authorize the caller, log the exchange immutably, and filter the response
before delivery. One published example is GATES[^gates], which names these
Secure, Govern, Authenticate, Trace, and Enforce, and applies them in the
order S, G, A, T, E. The acronym itself reads G, A, T, E, S.

**Gateway frameworks govern the pipe. ONUS governs the workflow and the human
decision.** The two are complementary layers.

A gateway can apply every one of its controls to every model call and still
produce ONUS findings when:

- The model call should have been twenty lines of deterministic code (N1,
  N4).
- The result surface blends live unverified model tokens with verified
  citations so the reader cannot tell what was checked (E1, E2).
- A policy or capability is revoked while an asynchronous workflow is queued,
  or downstream calls exceed the initial budget reservation (A3, A5).
- A worker crashes after starting a paid external job, or a user cancels
  mid-flight, leaving orphaned work with no operator path (R3, R4, R5).

[^gates]: GATES™ AI Governance Framework — coined by Druhin Dhavala,
    October 10, 2026. <https://www.druhindhavala.com/ai-governance/gates-framework/>.
    Licensed under CC BY 4.0. The summary above is a paraphrase, not a
    quotation.

## 4. Related documents

- [Design and workflow walkthrough protocol](../protocol/design-walkthrough.md):
  applying the four questions to a design before or alongside runtime testing.
- [N1 blind human-operator protocol](../protocol/n1-operator.md): a bounded
  empirical exercise against a pinned release.
- [Implementations and history](../IMPLEMENTATIONS.md): where ONUS
  responsibilities are implemented and where the framework originated.

## 5. Change policy

A change to a property's wording that alters what counts as a finding is a
normative change and is recorded in [CHANGELOG.md](../CHANGELOG.md) under the
specification. Adding or revising example mechanisms, observations, or prose is
not a normative change. Principle identifiers (N1 through R5) are stable once
published. A withdrawn principle keeps its identifier and is marked withdrawn.
