# ONUS Supplement H1: Blind Human-Operator Evaluation Protocol

**Version:** 0.1.0-draft
**Applies to:** [ONUS specification](../spec/onus.md) 0.1.0-draft
**License:** CC BY 4.0

Automated tests prove that code paths execute as written. They do not prove
that a person can govern the system under realistic information boundaries.
H1 is a bounded, repeatable empirical exercise run against a pinned release
tag. Its output is a set of findings, never a score. H is for human; the
letter avoids collision with specification principle N1.

## 1. Information boundaries and roles

To prevent knowledge leaks from invalidating the observations, the protocol
separates three participants.

1. **Builder (optional baseline phase).** Builds or configures the target
   application from public documentation and public runtime surfaces alone.
   Logs every time an undocumented internal file must be consulted
   (`violation`) and every time something fails without indication
   (`silent`). **The builder may be a human or an AI agent.**
2. **Coordinator and scorer.** Prepares the isolated test cases and freezes
   the Reference Sheet before the operator session begins. The scorer may
   inspect all code, logs, and internal state. The scorer also grades the
   session; section 5 states the control on that conflict.
3. **Human operator.** A person who did not build the application and has not
   read the internal source, the builder transcript, the audit planning
   documents, or the scorer's Reference Sheet.

**Rule.** No AI agent may substitute for the human operator. If no unexposed
human participant is available, record the exercise as `NOT RUN`.

## 2. Pre-exercise frozen Reference Sheet

Before showing any case to the operator, the scorer finalizes a private
Reference Sheet, computes the SHA-256 digest of the finished file, and records
the digest and timestamp in a detached manifest and in each case scoring
record. The digest is never written into the sheet itself, since doing so
would change the bytes it describes. Use the [reference sheet
template](templates/reference-sheet.md). It contains, per case:

- Task purpose, deterministic steps, and model duties (N).
- Named accountable role, permitted actions, and policy and evidence
  requirements (A, E).
- Exact test inputs and observable runtime facts (E).
- Justified expected human decisions and expected runtime transitions (A, R).
- Facts that genuinely remain unknown from the record (E, R).
- The specification properties the case is designed to exercise, by
  identifier (N1 through R5), and any the scorer already knows cannot be
  observed on this release, with the reason. Declaring these before the
  session is what stops the property record being written to fit the result.

**Integrity rule.** Use only the system's declared policy and actual
capabilities. Never invent a guarantee the runtime does not enforce. If an
expected result cannot be justified from the specification and the system's
own documentation, label that reference `UNRESOLVED` before observing the
operator. Never rewrite the Reference Sheet after the exercise.

## 3. Operator brief and time bounds

### 3.1 Time ceiling

| Phase | Budget |
| :--- | :--- |
| Case preparation and sealing | 30 minutes |
| Operator session | 60 minutes, at most 15 minutes per case |
| Scoring | 30 minutes |

The preparation budget is provisional. Record actual preparation time on the
first runs and revise this table from measurement, not from preference.

### 3.2 Operator packet

Provide only the neutral domain task, the declared policy and role, the
documented public entry points (UI, CLI, or documented HTTP endpoints), and the
permitted actions. Use the [operator packet template](templates/operator-packet.md).

### 3.3 Standard operator brief

> You are responsible for the supplied task within the stated policy. Inspect
> the proposed action and its evidence. Explain why model judgment is
> involved, what you may authorize, and what remains uncertain. Approve,
> reject, or leave the action undecided, stating your reason. Inspect the
> resulting state and evidence. After an interruption, identify what happened
> and the next documented action you can take. Say "unknown" where the
> available record cannot establish an answer. Ask for help if needed; we will
> record the difficulty.

## 4. The four standard cases and four questions

Run four isolated cases against the sealed application.

| Case | Scenario and target observation |
| :--- | :--- |
| 1. Supported action | A policy-permitted action with all required supporting evidence present. Observe approval and verification of the recorded outcome. |
| 2. Disallowed action | A proposed action outside declared policy or authority. Observe rejection and verification that no unauthorized effect occurred. |
| 3. Insufficient evidence | A missing input basis or unverified source state. Observe whether the operator recognizes the uncertainty rather than guessing. |
| 4. Process replacement or interruption | A pending approval or in-flight external action spanning a process restart or uplink interruption. Observe inspection of retained identity, evidence, and the next safe action. |

**Rule for case 3.** Induce the missing evidence only through supported
surfaces and supported failure modes. Do not fabricate the state by editing
storage directly; a state the system could never reach tells you nothing about
how the operator would see a state it can.

Cases 3 and 4 are the runtime form of
[walkthrough](design-walkthrough.md) stages 4 and 5.

In each case, evaluate the operator's unassisted response across the four
questions.

| Question | What to observe |
| :--- | :--- |
| Necessity | Can the operator identify the deterministic steps, the model's specific contribution, and how its output is checked? "No model needed" is a valid finding where applicable. |
| Evidence | Can the operator locate the input basis, decision, and result, and distinguish reported claims, observed effects, and unknowns? |
| Authority | Can the operator identify the accountable role, permitted action, and approval scope, and recognize where enforcement is missing? |
| Recovery | Can the operator identify the retained outcome after interruption or replacement and the next documented action, recognizing when human intervention is required? |

## 5. Scoring

Record two separate classifications per case, six friction counts, and a
property record. Use the [case scoring template](templates/case-scoring.md).
Never combine any of them into a composite.

### 5.1 Human comprehension outcome

Graded against the sealed Reference Sheet, per question:

- `MATCH`
- `MISMATCH`
- `UNRESOLVED`
- `NOT RUN`

An operator answer of "unknown" is a `MATCH` when the system's retained
evidence genuinely cannot establish the fact.

**Control on scorer bias.** A `MISMATCH` must quote the sealed Reference
Sheet line it contradicts, verbatim, next to the operator's unassisted answer.
A `MISMATCH` without a quoted line is recorded as `UNRESOLVED`.

### 5.2 Runtime behavior outcome

Observed independently of human comprehension:

- `OBSERVED AS EXPECTED`
- `OBSERVED DIFFERENCE`
- `UNKNOWN`
- `NOT RUN`

### 5.3 Friction measures

Per case:

1. Documentation lookups, including repeats.
2. Requests for coordinator assistance. Retain the operator's unassisted
   answer before giving any hint.
3. Approval prompts encountered.
4. Evidence retrieval cost: distinct requests needed to inspect the outcome.
5. Evidence retrieval cost: distinct services or ports needed to inspect the
   outcome.
6. Elapsed wall-clock time to the unassisted answer.

### 5.4 Property record

For each specification property the sealed Reference Sheet declared the case
exercises, record one of the three outcomes defined in
[specification section 1.2](../spec/onus.md#12-how-to-read-this-specification):

- `FINDING`: the property does not hold. Name what was observed and what the
  property requires.
- `NO FINDING`: the property was observed and holds. Name the inspection, the
  test, or the operator action that showed it.
- `UNOBSERVED`: the property could not be observed in this case. Name the
  missing record, surface, instance, or fault point.

Rules:

- Every declared property gets a row. A declared property with no row is
  recorded as `UNOBSERVED` with the reason "not recorded".
- A property the case exercised but the Reference Sheet did not declare may
  be added, marked as undeclared. Undeclared rows are reported separately so
  that a run cannot be made to look thorough after the fact.
- A `NO FINDING` with nothing named in the basis column is recorded as
  `UNOBSERVED`.
- A human comprehension `MISMATCH` is not by itself a property `FINDING`. It
  points at a property to inspect, and the property outcome is decided from
  the system's records and surfaces, with one exception. Where the scorer
  attributes the `MISMATCH` to a supported surface that is structurally
  complete but ambiguous, misleading, or illegible, record a `FINDING`
  against the principle that governs that surface (A1, E1, E2, E3, or R5)
  under the human-use test of specification section 1.1. The row's basis
  must carry three things: the sealed expected understanding, the operator's
  verbatim unassisted answer, and the surface. Without all three the row is
  `UNRESOLVED`, not a `FINDING`.
- `UNOBSERVED` is never reported as satisfied and is never dropped from the
  run record.

The table below is a starting point for which properties each standard case
usually exercises. The sealed Reference Sheet is authoritative; this table is
not.

| Case | Properties usually exercised |
| :--- | :--- |
| 1. Supported action | N1, N2, N3, N4, E1, E2, E3, E4, E5, A1, A3, A7 |
| 2. Disallowed action | A1, A2, A3, A4, A5, A6, A7, R1 |
| 3. Insufficient evidence | E1, E3, E5, R1 |
| 4. Process replacement or interruption | A3, R2, R3, R4, R5 |

Case 4 depends on an interruption the scorer can induce. If the sealed
release offers no way to stop a process or cut an uplink at the required
point, R3 and R4 are `UNOBSERVED` for that reason, and the case may have to be
`NOT RUN`.

## 6. What one run means

One operator and four cases produce findings, not rates. The result of an H1
run is the per-case record, the property record with its unobserved rows
intact, and the list of findings it supports. Results are never aggregated
across runs into a percentage, a pass rate, a coverage rate, or a composite
score. Two runs against different releases may be compared finding by finding
and property by property; they may not be averaged.

## 7. History

H1 originated as the Golden Supplement N1 blind-operator exercise in the
SemStreams project, and was renamed H1 in this repository so that N1 names
only the specification principle. See
[IMPLEMENTATIONS.md](../IMPLEMENTATIONS.md) for the lineage and current
implementing projects.
