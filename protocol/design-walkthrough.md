# HAAG Design and Workflow Walkthrough Protocol

**Version:** 0.2.0
**Applies to:** [HAAG specification](../spec/haag.md) 0.2.0
**License:** CC BY 4.0

Before or alongside runtime testing, a team can apply HAAG as a fast
architecture and workflow audit across an end-to-end journey. The output is a
list of findings and untested gaps tagged to specification principles—never a
score.

## 1. Step-by-step workflow mapping

List every sequential and parallel step of a representative workflow, from
caller authentication through external tool or model execution, verification,
packaging, and closure. Then add rows for its failure and cancellation
branches. Tag each step with the NEAR headings and principles that apply there.

### 1.1 Worked example

The table below maps one representative workflow: a queued job with a quota
ledger, worker pickup, an external call, and durable settlement. **It is an
example of the mapping method, not a required architecture.** A synchronous
workflow with no queue and no ledger maps the same way: list the steps, then
tag them.

| Step | Component or boundary | Headings | Principles | Governance obligation |
| :--- | :--- | :--- | :--- | :--- |
| 01. Caller submission | Edge, UI, or API | A | A1 | Verify identity, session, and scope. Disclose external effects and limits. |
| 02. Input admission | Scanner or policy guard | A, R | A2, R1 | Validate input before paid or effectful work. Distinguish refusal from guard failure. |
| 03. Resource reservation | Ledger or quota store | A, E | A5 | Reserve the estimated budget for the whole run, delegates included. Refuse the run if insufficient. |
| 04. Capability pickup | Worker or tool adapter | A, E | A3, A4, A6, A7 | Re-read the current policy revision at pickup. Emit no external traffic on refusal. |
| 05. External or model call | Model or tool provider | N, A, R | N2, N3, N4, A5, R3, R4 | Bound retries. Persist the job identity before polling. Record uncertain creates. |
| 06. Output verification | Scanner or validator | E, A | N3, E1, E3 | Inspect output and error text before storage or downstream use. Label the copy's state. |
| 07. Durable settlement | State store | E, R | E5, E6, R2 | Commit result, provenance, and cost in one revision-guarded transaction. |
| 08. Secondary model duty | Evaluator, summarizer, or memory write | N, E | N1, N2, N4, E2, E4 | Operate only on saved checked copies. Keep model prose separate from code-computed metrics. |
| 09. Closure and delivery | Coordinator | E, R | E1, R4, R5 | Wait for active work to settle. Separate user-facing readiness from background archive completion. |

Failure and cancellation branches get their own rows. At minimum, map:

- a policy or capability revocation between steps 01 and 04,
- a worker crash between sending the request and saving the job ID in step 05,
- and a user cancellation while step 05 is active.

## 2. Five-stage human walkthrough rubric

To check whether a design satisfies the human-use test, walk one representative
scenario through five stages. At each stage, record what the responsible person
can and cannot do using only supported product surfaces. Every "cannot" is a
finding against the principles listed.

| Stage | What the responsible person must be able to do | Principles |
| :--- | :--- | :--- |
| 1. Before submission | Explain why a model is involved, how its output is checked, which external services may run, who can see the result, and the resource or spending limit. | N2, N3, N4, A1, A5 |
| 2. At a scope or policy change | Identify who can authorize the change, which policy revision applies, the exact effect of approval or rejection, and whether an earlier approval still holds. | A2, A3, A4, A7, E5 |
| 3. At a result | Separate an external source claim, a model judgment or summary, a checked or cleaned copy, a cryptographic integrity signature, and a code-checked figure. | N1, E1, E2, E3 |
| 4. With missing evidence | Identify the missing item (uncaptured source, unscored output, unavailable log) and its effect on relying on the answer. | E1, E5, R1 |
| 5. After failure or replacement | Identify completed effects, uncertain or orphaned effects, retained cost facts, and the next authorized recovery action. | R2, R3, R4, R5 |

Stages 4 and 5 are the design-time version of [H1 protocol](h1-operator.md)
cases 3 and 4. A design that fails stage 4 or 5 on paper will fail with a live
operator. Run this walkthrough first; it takes under an hour.

## 3. Recording the result

Use the [design walkthrough record template](templates/design-walkthrough-record.md)
to log what you checked:

1. **Findings.** Each place a principle did not hold: the step or stage, the
   principle ID, what you observed, what the property requires, and any
   trade-off the team accepted.
2. **Checked (no finding) and unobserved gaps.** Which principles held (and how
   you checked them), and which ones could not be tested yet (for example, no
   fault injection point for R4 or no invocation log for N2).

Do not compute a percentage or composite score. A single finding at step 05
under R4 or A7 can matter more than every other row combined.
