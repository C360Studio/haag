# HAAG: Human-Agent Accountability and Governance

An agentic governance framework. Responsibility can be delegated.
Accountability cannot.

**Version:** 0.1.0-draft
**Status:** Public specification draft
**License:** CC BY 4.0 (see [LICENSE](../LICENSE))

## 1. Purpose and core premise

HAAG (Human-Agent Accountability and Governance) is an architecture-neutral
governance and assurance framework for systems that combine deterministic
software with probabilistic model or agent steps. Its name states its premise:
a person may delegate a task to an agent, but accountability for the outcome
stays with the person. Its four observation questions are remembered by the
mnemonic **NEAR**: Necessity, Evidence, Authority, Recovery.

Most AI governance frameworks treat governance either as organizational policy
(committees, checklists, risk tiers) or as a synchronous control point at the
model perimeter. Agentic and multi-step workflows fail across broader seams:
models are invoked where deterministic code belongs, policies change while
long-running work is in flight, live model prose is mistaken for verified
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
outcome cannot. The law has a name for the analogous idea, the non-delegable
duty; HAAG takes it as the governance premise of every principle below,
without claiming it as a legal fact in any jurisdiction. The test across all
four headings is therefore **human use**: the responsible person must be able
to:

1. Understand the decision and inspect its basis.
2. Act within their authority and understand the scope and cost of what they
   permit.
3. Distinguish verified facts, model claims, and unknown states.
4. After an interruption or failure, identify what actually occurred and choose
   the next safe action.

They must be able to do all four **without reading internal source code or
reconstructing a workflow from raw logs**.

A supported surface that a responsible person demonstrably cannot use is a
finding against the principle that governs that surface, even where the
surface is structurally complete. The
[operator protocol](../protocol/h1-operator.md) defines the evidence that
demonstrates it.

### 1.2 How to read this specification

Each principle in section 2 has three parts.

- **Property.** What must be true, stated so that a reviewer can observe
  whether it holds. This is the normative content, and the only place a
  requirement is stated.
- **Observe.** What a reviewer looks at, or what bounded test a reviewer runs,
  to decide whether the property holds. Observe introduces no requirement of
  its own. Where an Observe paragraph names a finding under another
  principle, that principle's Property states the requirement.
- **Example mechanisms.** Ways teams have satisfied the property. These are
  non-normative. HAAG requires none of them, and a mechanism's presence does
  not establish the property.

#### The reviewer

A reviewer applying this specification has the system's records and surfaces,
its configuration and documentation, a disposable non-production instance of
it, and the ability to inject faults on that instance: kill a process at a
named point, take a dependency down, cut the network. The reviewer may read
source code. The responsible person of section 1.1 must never need to. Where
an observation requires an instance, a record, or a fault point the reviewer
does not have, the property is recorded as unobserved, never as satisfied.

#### Outcomes

Each property yields one of three outcomes. HAAG describes a system by its
findings, never by a pass or fail.

- **Finding.** The property does not hold. Every property is a requirement;
  there is no weaker tier, and a team's reason for not meeting one is
  recorded with the finding, not instead of it.
- **No finding.** The property was observed and holds for what was inspected.
- **Unobserved.** The property could not be observed with the reviewer's
  access, and the reason is recorded. Unobserved is never reported as
  satisfied.

There is no notion of HAAG compliance, certification, or score. Rolling
findings into a single number hides the single-boundary failure that matters.

#### Coverage

Observations are made against the step map produced by the [design walkthrough
protocol](../protocol/design-walkthrough.md). Properties that quantify over
every boundary, every surface, or every invocation are observed by sampling
that map. The reviewer records which steps, edges, and surfaces were inspected
and which were not. Findings come from what was inspected. What was not
inspected is coverage, not a pass.

## 2. The four headings

### 2.1 Necessity (N)

> Why does this step need a model, and how will its result be checked?

Model calls introduce non-determinism, latency, cost, and attack surface. Each
model duty must be justified, and what the model is trusted to produce must be
bounded and checked.

#### N1. Deterministic authority

**Property.** Every numeric or invariant-bearing value a model produces passes
through a code check that bounds or verifies it before the value is used in a
state transition, a settlement, or a user-facing figure. A user-facing figure
is a structured metric, total, score, or field presented outside a region
labelled as model claim (see E2); a number inside free-text model prose
rendered in a claim region is a claim, not a figure, and a number extracted
from that prose into a structured field becomes a figure and must pass the
check. Calculations, aggregations, schema routing, state transitions, and
formatting are performed by code. A number a model legitimately produces (a
rubric score, a triage priority) is recorded as a model claim, not as an
observed fact. Whether a duty should have been code at all is a question for
N4, where it is tested rather than asserted.

**Observe.** For each numeric or invariant-bearing model output in the step
map, follow the data path to its first use. A value that reaches a state
transition, a settlement, or a user-facing figure without passing a code check
is a finding.

**Example mechanisms.** Range and enum validation on model outputs; code-side
recomputation of any total the model reports; invariants enforced in the state
store rather than in the prompt.

#### N2. Named, bounded duty

**Property.** Each model invocation has a named duty (for example, triage
classification, candidate summarization, rubric scoring) with declared inputs
and a declared output shape. A duty is skipped, not invoked, when no valid
input exists.

**Observe.** The list of model duties for the workflow, matched against the
record of model invocations. An invocation that cannot be matched to a named
duty, or that ran on empty or invalid input, is a finding. This observation
requires a record of every model invocation. If no such record exists, that is
an E2 finding, and N2 is recorded as unobserved.

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
code, a lookup, a human) on a fixed corpus of representative inputs, edge
cases, and failure modes for the duty (for an extraction or triage duty: known
facts, distractions, and contradictions). The comparison record names the duty
configuration it evaluated: model, prompt or instructions, and check. The
decision to rely on the duty cites that record. When the model, prompt, or
check changes materially, the comparison is rerun before the changed duty is
relied upon; reliance that cites a comparison of a different configuration is
a finding. A syntactically valid response is not evidence that the step is
useful.

**Observe.** The stated success criterion, the corpus, the comparison record
with its named configuration, and the citation from the decision to rely on
the duty. Compare the configuration in the record with the configuration in
production. A mismatch, or a comparison the model lost followed by reliance
with no stated reason, is a finding. "No model needed" is a valid outcome of
the comparison and is recorded as such.

**Example mechanisms.** A golden corpus checked into the repository; a side-
by-side run against a rule-based baseline; periodic re-comparison when the
model or prompt version changes.

### 2.2 Evidence (E)

> What happened, what supports the result, and can the responsible person
> inspect it?

Logging events to an internal telemetry backend does not satisfy the Evidence
heading if the responsible person or a downstream consumer cannot inspect the
basis of a result through supported product surfaces.

#### E1. Labelled verification state

**Property.** At every boundary in the step map that carries external,
untrusted, or model-derived data (artifacts, evidence, model and tool
outputs), the data carries labels along four independent axes. Pure control
edges that carry none of these (authentication, quota reservation, capability
lookup) are out of scope for E1:

| Axis | Labels |
| :--- | :--- |
| Verification | unchecked or live; checked against a named policy revision |
| Transformation | original; redacted or cleaned, with provenance of the transformation |
| Disposition | admitted; suspect or quarantined (blocked from downstream model or user consumption) |
| Completeness | complete; truncated; unscored |

The axes are independent. Partial output is never silently promoted to
complete. A single status field that collapses the axes loses information and
is a finding.

**Observe.** For each edge in the step map, which labels are carried and
whether the consuming component or surface reads them. Record which edges were
inspected. A surface that shows a truncated or unscored result without the
label is a finding. Truncate an output on the disposable instance and confirm
the label survives to the surface.

**Example mechanisms.** Per-axis fields on the artifact record; a wrapper type
that cannot be unwrapped without reading the labels; UI badges derived from
the labels rather than from a separate status.

#### E2. Observations distinct from claims

**Property.** Every model invocation and every effectful tool call is recorded
by the harness as an observation. Harness-recorded facts (model invocations,
tool executions, HTTP effects, state transitions, timestamps, resource
consumption) are stored in a field or record type structurally distinct from
model-generated prose, inferences, and recommendations. Every surface that
shows both renders them in separately labelled regions.

**Observe.** The record schema, and each result surface in the step map. A
model invocation with no harness record, a record with no structural
distinction, or a surface that renders model text inside a region labelled as
observation, is a finding. Whether a person can actually tell the two apart is
measured by the operator protocol. A MISMATCH there that the scorer attributes
to this surface is a finding against E2 under the human-use test (section
1.1).

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

**Observe.** The input source of each downstream duty in the step map. Then
one test on the disposable instance, in either form: run with distinguishable
checked and unchecked fixtures and confirm the downstream duty consumed the
checked revision; or alter the stored checked copy and confirm that the
downstream duty either sees the alteration or halts on an integrity or
revision mismatch. A downstream duty that consumes the live stream, or that
proceeds on the unaltered live content while the stored copy differs, is a
finding.

**Example mechanisms.** Downstream duties read from the artifact store by
revision, not from the producing step's output channel.

#### E5. Inspectable provenance

**Property.** An artifact or decision record links to its authoritative input
revision, its source receipts (including sources that were explicitly
uncaptured or unavailable), the policy and prompt versions active at the time,
and its integrity signatures. The responsible person can reach all of these
through supported product surfaces.

**Observe.** Starting from a result, follow the links. A link that requires
internal tooling or raw log access is a finding under the human-use test. The
uncaptured-source clause cannot be checked from the record alone: a record
that silently omits a source looks identical to one with fewer sources. It is
observed only by inducing a known-unavailable source, which is H1 protocol
case 3. Without that induced case, record the clause as unobserved.

**Example mechanisms.** A provenance panel on the result surface; receipts
that record "not captured" as a first-class value; prompt and policy versions
pinned on the record.

#### E6. Retention for dispute resolution

**Property.** For each decision class the responsible organization has
declared consequential, the system retains the input revision, the request,
the observed result, the authority decision, and sufficient context for an
independent reviewer to audit the outcome: to confirm that the recorded output
is the one that was acted upon. Reproduction is not required; a step with a
model in it is not reproducible. The organization sets and states both the
list of consequential decision classes and the retention period. HAAG
prescribes neither, only that both are declared.

**Observe.** The declared decision classes and retention period, then an audit
of a sampled past decision within the period. The reviewer either confirms the
recorded output was the one acted upon or names the missing item. An
undeclared class list leaves the property's scope undefined and is itself a
finding.

**Example mechanisms.** A retention policy document referenced from the
product; immutable decision records with a declared expiry.

### 2.3 Authority (A)

> Who permits the action, what may it do, and where must it stop?

Authority governs identity, scope of permission, resource and financial
boundaries, and how policies are evaluated and enforced across distributed
steps.

#### A1. Understandable scope of permission

**Property.** The submission or approval surface displays, before the person
commits, four things: what data leaves the system and to where, who can see
the result, which effectful tools may run, and the resource or spending bound.
Authentication is not a substitute for this display.

**Observe.** The approval surface as the responsible person sees it. Any of
the four items that must be inferred from configuration or documentation
elsewhere is a finding. Whether the person understood what was displayed is
measured by the operator protocol. A MISMATCH there that the scorer attributes
to this surface is a finding against A1 under the human-use test (section
1.1).

**Example mechanisms.** A scope summary on the approval prompt; a declared
spending ceiling shown before submission.

#### A2. Enforcement points distinct from policy evaluators

**Property.** Enforcement points (workflow guards, tool dispatchers, storage
boundaries, provider adapters) are distinct from policy evaluators (local rule
libraries, external governance gateways, human approval queues). Where several
evaluators are combined, the contract states check ordering, timeout handling,
conflict resolution, and which baseline safety controls no evaluator can
override.

**Observe.** The contract between enforcement points and evaluators, then one
behavioural test on the disposable instance: take an evaluator down and
confirm the enforcement point fails closed or enters the R1 operator-wait
state. An enforcement point that passes traffic while its evaluator is
unreachable was never distinct from it, and that is a finding. An evaluator
that can disable a baseline control, or undefined behaviour when two
evaluators disagree, is a finding.

**Example mechanisms.** A policy decision point separate from the dispatcher;
a documented precedence order; a hard-coded deny list that evaluators cannot
relax.

#### A3. Per-boundary authority and policy revision

**Property.** In a multi-step or long-running workflow there is no assumed
whole-run policy snapshot. Each boundary records its own admission decision and
the exact policy or registry revision read at execution time. Work is refused
at pickup when a capability was disabled after the workflow was scheduled, and
refusal emits no external traffic.

**Observe.** On the disposable instance, schedule a multi-step run, then
revoke a capability or change the policy revision between scheduling and the
next pickup. Read the revision recorded on that step and capture outbound
traffic. A step that ran under the older revision, a pickup that did not
refuse, or any external traffic emitted on refusal is a finding. If the
instance offers no way to change policy mid-run, record A3 as unobserved.

**Example mechanisms.** Revision read and recorded at pickup; a capability
registry consulted per step rather than per run.

#### A4. Authority does not amplify through delegation

**Property.** When a step delegates to a sub-step, sub-agent, or external job,
the delegate's permitted scope is a subset of the delegator's scope at
delegation time, and is re-evaluated at the delegate's own pickup under A3. A
delegate cannot acquire a capability, budget, visibility, or egress path that
its delegator lacked. Scopes are recorded in a form that permits comparison; a
scope expressed only in prose cannot be audited for subset.

**Observe.** For a delegation chain, compare the recorded scopes at each
level. Any widening is a finding. A scope whose form does not permit
comparison is a finding. A delegate that executed under a policy revision
older than the one in force at its pickup is a finding under A3.

**Example mechanisms.** Scope tokens attenuated at each hop; the delegator's
budget reservation shared, not duplicated, by delegates.

#### A5. End-to-end resource and spending boundaries

**Property.** Passing an initial budget or quota check does not establish a
total run ceiling unless reservations account for retries, downstream
evaluation calls, auxiliary fetches, delegates (A4), and external jobs that
may keep running after cancellation. Permission to start a workflow is never
implicit authorization to exceed a declared limit.

**Observe.** From the step map, identify the worst permitted path: maximum
retries, every delegate, every downstream evaluation, every external job that
survives cancellation. Then test rather than sum: on the disposable instance,
set a small declared limit, drive that path, and compare total spend against
the limit. Spend above the limit is a finding. An unbounded component on the
path is a finding under N3 or A4 before the test is run.

**Example mechanisms.** Atomic reservation against a ledger before each
effectful call; a per-run ceiling that delegates draw from.

#### A6. Enforced versus advisory boundaries

**Property.** Documentation states the true runtime enforcement boundary. If a
role or agent is described as read-only or sandboxed, the runtime enforces
that restriction. Omitting a named helper tool is not enforcement.

**Observe.** For each restriction the documentation describes, attempt the
restricted action from the restricted role on the disposable instance. Success
is a finding. Reading the enforcement mechanism is not a substitute for the
attempt.

**Example mechanisms.** Filesystem and network policy applied at the process
boundary; a read-only credential rather than an instruction not to write.

#### A7. Approval bound to a revision

**Property.** An approval binds to the specific action, its target, the input
revision, the policy revision in force, and the authorizing principal, all
recorded at approval time. If any of the five changes before execution, the
approval is invalid and execution refuses until the changed action is approved
again. An approval is never reused for a changed action.

**Observe.** On the disposable instance, approve an action, then alter its
input or target before execution. Execution that proceeds on the prior
approval is a finding. An approval record missing any of the five bindings is
a finding.

**Example mechanisms.** Approval keyed by a content hash of action, target,
input revision, and policy revision; revision-guarded execution that compares
the approval's bindings to the attempt's before acting (see R2).

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

**Observe.** On the disposable instance, take a required safety check down
(the same fault as the A2 test) and run a step that depends on it. Read the
recorded outcome. "Clean", "passed", or any outcome indistinguishable from a
policy denial is a finding; the expected outcomes are fail closed, bounded
retry, or operator-wait, each recorded as an operational failure. If the
instance offers no way to take the check down, record R1 as unobserved. This
principle is shared with Authority: a refusal is an authority decision, a
failure is not.

**Example mechanisms.** Distinct outcome types for denied and errored; a
scanner timeout that yields "unscanned" rather than "no match".

#### R2. No duplicate or stale effects

**Property.** Worker retries, replays, and stale attempts cannot duplicate a
logical effect or overwrite newer state. Consequential state transitions,
settlements, and artifact writes are guarded.

**Observe.** Two tests on the disposable instance. First, replay a completed
step and confirm no second effect. Second, start two attempts of one step, let
the later attempt commit, then let the earlier (now stale) attempt finish and
attempt its write. A second effect from the replay, or the stale attempt's
write landing over the newer state, is a finding.

**Example mechanisms.** Compare-and-swap; revision guards; idempotency keys
derived from the identity of the logical effect or step, never from the
execution attempt, since each retry mints a new attempt and an attempt-derived
key would not deduplicate across retries.

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
  separately, and records the resource and cost facts the external provider
  reports at termination time.

**Observe.** The recorded state after two induced scenarios on the disposable
instance: a crash between the send and the identifier write, and a cancel
while external work is active. A zero or "no effect" where the system could
not have known the value is a finding. Compare the recorded cost facts against
the provider's own record for the same job; a provider fact missing from the
record is a finding. Both scenarios require a fault injection point. Without
one, record R4 as unobserved.

**Example mechanisms.** A tri-state effect record (confirmed, confirmed-none,
uncertain); cost fields that distinguish "0" from "unknown".

#### R5. Authorized operator path

**Property.** Flagging an uncertain or degraded state in a database is
incomplete Recovery under HAAG. An authorized person has a documented surface
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

**Gateway frameworks govern the pipe. HAAG governs the workflow and the human
decision.** The two are complementary layers.

A gateway can apply every one of its controls to every model call and still
produce HAAG findings when:

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
- [H1 blind human-operator protocol](../protocol/h1-operator.md): a bounded
  empirical exercise against a pinned release.
- [Implementations and history](../IMPLEMENTATIONS.md): where HAAG
  responsibilities are implemented and where the framework originated.

## 5. Change policy

Any change to a Property or Observe paragraph that alters what counts as a
finding is a normative change and is recorded in
[CHANGELOG.md](../CHANGELOG.md) under the specification. Adding or revising
example mechanisms or other prose is not a normative change. Principle
identifiers (N1 through R5) freeze at the 0.1.0 release, when the draft
suffix drops, and are stable from then on. A withdrawn principle keeps its
identifier and is marked withdrawn.
