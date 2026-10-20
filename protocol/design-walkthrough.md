# ONUS Design and Workflow Walkthrough Protocol

**Version:** 0.1.0-draft
**Applies to:** [ONUS specification](../spec/onus.md) 0.1.0-draft
**License:** CC BY 4.0

Before or alongside runtime testing, a team can apply ONUS as a structured
design and workflow audit across every step of an end-to-end journey. The
output is a list of findings tagged to specification principles, never a
score.

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
| 04. Capability pickup | Worker or tool adapter | A, E | A3, A4, A6 | Re-read the current policy revision at pickup. Emit no external traffic on refusal. |
| 05. External or model call | Model or tool provider | N, A, R | N2, N3, A5, R3, R4 | Bound retries. Persist the job identity before polling. Record uncertain creates. |
| 06. Output verification | Scanner or validator | E, A | N3, E1, E3 | Inspect output and error text before storage or downstream use. Label the copy's state. |
| 07. Durable settlement | State store | E, R | E5, E6, R2 | Commit result, provenance, and cost in one revision-guarded transaction. |
| 08. Secondary model duty | Evaluator or summarizer | N, E | N1, N2, E2, E4 | Operate only on saved checked copies. Keep model prose separate from code-computed metrics. |
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
| 1. Before submission | Explain why a model is involved, which external services may run, who can see the result, and the resource or spending limit. | N2, N4, A1, A5 |
| 2. At a scope or policy change | Identify who can authorize the change, which policy revision applies, and the exact effect of approval or rejection. | A2, A3, A4, E5 |
| 3. At a result | Separate an external source claim, a model judgment or summary, a checked or cleaned copy, and a cryptographic integrity signature. | E1, E2, E3 |
| 4. With missing evidence | Identify the missing item (uncaptured source, unscored output, unavailable log) and its effect on relying on the answer. | E1, E5, R1 |
| 5. After failure or replacement | Identify completed effects, uncertain or orphaned effects, retained cost facts, and the next authorized recovery action. | R2, R3, R4, R5 |

Stages 4 and 5 are the design-time form of
[N1 protocol](n1-operator.md) cases 3 and 4. A design that fails stage 4 or 5
here will produce a MISMATCH or UNRESOLVED there. Run the walkthrough first;
it is cheaper.

## 3. Recording the result

Record each finding as:

- the step or stage,
- the principle identifier,
- what the reviewer observed,
- what the property requires,
- and, for a "should", the trade-off the team accepted.

Do not total the findings. Do not compute a percentage of principles
satisfied. A single finding at step 05 under R4 can matter more than every
other row combined.
