# ONUS Design and Workflow Walkthrough Protocol

**Version:** 0.1.0-draft
**Applies to:** [ONUS specification](../spec/onus.md) 0.1.0-draft
**License:** CC BY 4.0

Before or alongside runtime testing, a team can apply ONUS as a structured
design and workflow audit across every step of an end-to-end journey. The
output is a per-principle outcome record tagged to specification principles,
never a score. The step map produced in section 1 is also the coverage basis
for every observation in the specification (see
[specification section 1.2](../spec/onus.md#12-how-to-read-this-specification)).

## 1. Step-by-step workflow mapping

List every sequential and parallel step of a representative workflow, from
caller authentication through external tool or model execution, verification,
packaging, and closure. Then list its failure and cancellation branches. Tag
each step with the NEAR headings it exercises and the principles a reviewer
should check there.

### 1.1 Worked example

The table below maps one representative workflow: a queued job with a quota
ledger, worker pickup, an external call, and durable settlement. **It is an
example of the mapping method, not a required architecture.** A workflow with
no ledger and no worker queue maps the same way: list the steps, then tag
them.

| Step | Component or boundary | Headings | Principles | Governance obligation |
| :--- | :--- | :--- | :--- | :--- |
| 01. Caller submission | Edge, UI, or API | A | A1 | Verify identity, session, and scope. Disclose external effects and limits. |
| 02. Input admission | Scanner or policy guard | A, R | A2, R1 | Validate input before paid or effectful work. Distinguish refusal from guard failure. |
| 03. Resource reservation | Ledger or quota store | A, E | A5 | Reserve the estimated budget for the whole run, delegates included. Refuse the run if insufficient. |
| 04. Capability pickup | Worker or tool adapter | A, E | A3, A4, A6, A7 | Re-read the current policy revision at pickup. Emit no external traffic on refusal. |
| 05. External or model call | Model or tool provider | N, A, R | N2, N3, N4, A5, R3, R4 | Bound retries. Persist the job identity before polling. Record uncertain creates. |
| 06. Output verification | Scanner or validator | E, A | N3, E1, E3 | Inspect output and error text before storage or downstream use. Label the copy's state. |
| 07. Durable settlement | State store | E, R | E5, E6, R2 | Commit result, provenance, and cost in one revision-guarded transaction. |
| 08. Secondary model duty | Evaluator or summarizer | N, E | N1, N2, N4, E2, E4 | Operate only on saved checked copies. Keep model prose separate from code-computed metrics. |
| 09. Closure and delivery | Coordinator | E, R | E1, R4, R5 | Wait for active work to settle. Separate user-facing readiness from background archive completion. |

Failure and cancellation branches get their own rows. At minimum, map: a
policy change between steps 01 and 04, a crash between the send and the
identity write in step 05, and a cancel while step 05 is active.

## 2. Five-stage human walkthrough rubric

To audit whether a design satisfies the human-use test, walk a single
representative scenario through five stages. At each stage, record what the
responsible person can and cannot do using only supported surfaces. Each
"cannot" is a finding against the principles listed.

| Stage | What the responsible person must be able to do | Principles |
| :--- | :--- | :--- |
| 1. Before submission | Explain why a model is involved, how its output is checked, which external services may run, who can see the result, and the resource or spending limit. | N2, N3, N4, A1, A5 |
| 2. At a scope or policy change | Identify who can authorize the change, which policy revision applies, the exact effect of approval or rejection, and whether an earlier approval still holds. | A2, A3, A4, A7, E5 |
| 3. At a result | Separate an external source claim, a model judgment or summary, a checked or cleaned copy, a cryptographic integrity signature, and a code-checked figure. | N1, E1, E2, E3 |
| 4. With missing evidence | Identify the missing item (uncaptured source, unscored output, unavailable log) and its effect on relying on the answer. | E1, E5, R1 |
| 5. After failure or replacement | Identify completed effects, uncertain or orphaned effects, retained cost facts, and the next authorized recovery action. | R2, R3, R4, R5 |

Stages 4 and 5 are the design-time form of
[H1 protocol](h1-operator.md) cases 3 and 4. A design that fails stage 4 or 5
here will produce a MISMATCH or UNRESOLVED there. Run the walkthrough first;
it is cheaper.

A "cannot" at any stage is a finding even where the surface is structurally
complete. This is the same rule the operator protocol applies to a MISMATCH
attributed to a surface: a supported surface the responsible person cannot use
is a finding under the human-use test of specification section 1.1.

## 3. Recording the result

The reviewer applying this protocol has the vantage defined in specification
section 1.2: records, surfaces, configuration, a disposable non-production
instance, and fault injection on it. Each principle observed yields one of
three outcomes, and all three are recorded. Use the
[design walkthrough record template](templates/design-walkthrough-record.md).

### 3.1 Findings

Record each finding as:

- the step or stage,
- the principle identifier,
- what the reviewer observed,
- what the property requires,
- and any reason the team gives for not meeting it.

### 3.2 No finding

Record each principle that was observed and holds as:

- the step or stage,
- the principle identifier,
- and what was inspected or which test was run.

A no-finding without a named inspection or test is recorded as unobserved.

### 3.3 Unobserved

Record each principle that could not be observed as:

- the step or stage,
- the principle identifier,
- and the reason: the missing record, instance, surface, or fault point.

Unobserved is never reported as satisfied. A spec Observe paragraph that
names an unobserved condition (for example, no invocation record for N2, no
fault injection point for R4) produces this outcome, not a pass.

### 3.4 Coverage

Alongside the outcomes, record which steps, edges, and surfaces of the step
map were inspected and which were not. Findings and no-findings come from
what was inspected. What was not inspected is coverage, not a pass.

### 3.5 No totals

Do not total the findings. Do not compute a percentage of principles
satisfied, observed, or covered. A single finding at step 05 under R4 can
matter more than every other row combined.
