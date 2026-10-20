# Example Design Walkthrough: SemTeams Research & Program Pulse

<!-- CC BY 4.0 licensed worked example of the HAAG 0.2.0 design walkthrough
     protocol against C360Studio/semteams. -->

This file is a real worked example of the
[HAAG Design and Workflow Walkthrough Protocol](../design-walkthrough.md) run
against the `research` category pack (`/research` and `/pulse`) in
[SemTeams](https://github.com/C360Studio/semteams). The five findings below are
tracked in [SemTeams #352](https://github.com/C360Studio/semteams/issues/352).

| Field | Value |
| :--- | :--- |
| Workflow walked | SemTeams `research` category pack (`plan -> gather [1..4] -> synthesize -> reviewer-research -> coordinator`) + `approval_required` tool gate |
| Target release tag or commit | `semteams@30ce863a` (on frozen `semstreams@8b99efe9`) |
| Target configuration | `configs/flow-bootstrap.json`, `configs/rules/research/*.json`, `SEMTEAMS_PORTFOLIO_PATH` allowlist |
| HAAG version applied | 0.2.0 |
| Reviewer | Static and surface walkthrough (`semteams` rules, tool executors, Svelte UI, ADRs 030, 060, 061) |
| Date (UTC) | 2026-10-20 |

## 1. Step map

| Step | Component or boundary | Headings | Principles | Governance obligation |
| :--- | :--- | :--- | :--- | :--- |
| 01. Operator submission | UI `/research` or `/pulse` action chip -> `agentic-dispatch` | A, N | A1, N2 | Identify caller (`xUserIDIdentityMiddleware`), disclose external channels (`github_read`, `web_search`), and route to coordinator. |
| 02. Coordinator and Plan spawn | Rule `01-coordinator-research-spawn.json` -> `researcher-research-plan` (`max_iterations=12`) | N, A | N2, N3, A5 | Decompose request into `1..4` subtopics via `researchdecide` and `emit_plan`. Reject `<1` or `>4` epics before publishing triples. |
| 03. Parallel Gather fan-out | Rule `02-plan-to-gather.json` (`for_each` over `1..4` subtopics) -> `researcher-research-gather` (`max_iterations=20`) | N, A, E | N2, A2, A4, A6, E2 | Execute `github_read` (portfolio-allowlisted, server-injected token), `web_search`, or `bash` (with `*_TOKEN` stripped). Record tool trajectories in `AGENT_TRAJECTORIES` and `AGENT_CONTENT`. |
| 04. Tool approval gate (when triggered) | `ApprovalFilter` -> `LoopEntity.PendingApproval` -> `PendingApprovalSection.svelte` (`POST /teams-dispatch/loops/{id}/approval`) | A, R | A1, A2, A7, R3 | Pause loop on `approval_required` tools; display tool name, reason, and JSON args; resume only on matching `execution_id` approval. |
| 05. Gather join barrier | Rules `03a` and `03b` (`research.gather.completed-subtopic` counter `length_eq` `subtopics.length`) | E, R | R2, R3 | Deduplicate completed gather loop IDs on the PLAN entity; spawn `researcher-research-synthesize` (`max_iterations=24`) once all `N` gatherers finish. |
| 06. Mechanical derivation and artifact emit | `emit_research_artifact` (`VerifyDerivation` + `Executor.Execute`) | N, E, R | N1, N3, E1, E2, E3, E4, E5, R1 | Verify every `EvidenceRecord`'s `tool_result_ref` belongs to the current run anchor, check URL/token boundaries and verbatim `quote` (>= 12 chars) against raw tool output without an LLM, stamp server-side `retrieved_at` and `kind`, compute count triples in Go, and distinguish `ToolErrorInvalidArgs` from `ToolErrorNetwork`. |
| 07. Conformance review | Rule `04-synthesize-to-reviewer.json` -> `reviewer-research` (`max_iterations=8`) | N, E | N2, N3, E4 | Grade plan conformance against the saved artifact (`decide(action=approved \| insufficient)`). |
| 08. Closure and delivery | Rule `07-reviewer-approved-to-coordinator.json` -> coordinator `respond_direct` | E, R | E2, E5, R5 | Wake coordinator to deliver the final response and link the rendered research artifact. |
| B1. Branch: Reviewer rejection or clarification | Rules `05-reviewer-rejected-retry.json` and `06-needs-clarification-replan.json` (`max_iterations=3`) | N, A | N3, A5 | Route `insufficient` back to PLAN or `needs_clarification` to coordinator. |
| B2. Branch: Gatherer or loop failure | Rules `08-loop-failed-pause.json` and `09-loop-failed-run-outcome.json` | R | R1, R4, R5 | Stamp `chain.paused.marker` on loop failure; halt join rather than synthesizing partial unchecked output. |
| B3. Branch: Cancel during pending approval or active gather | UI Cancel button -> `sendSignal("cancel")` (`ADR-030`) | R | R4, R5 | Terminate loop and discard pending approval. |

## 2. Five-stage human walkthrough

| Stage | Can do | Cannot do (each is a finding) | Principles |
| :--- | :--- | :--- | :--- |
| 1. Before submission | Trigger `/pulse` or `/research` from the UI and inspect the static portfolio repository list in `/work`. | Cannot see the run's spending/token ceiling, external egress summary, or full list of permitted effectful tools on the submission surface before starting the run. | N2, N3, N4, A1, A5 |
| 2. At a scope or policy change | Inspect the paused tool name, reason, and JSON arguments in `PendingApprovalSection.svelte`; approve, reject, or edit arguments; receive a `409` alert if another operator already resolved the execution. | Cannot verify the authorizing identity (uses unauthenticated `X-User-Id` from `localStorage`), cannot see which policy revision governs the approval, and approvals do not bind to input or policy revisions. | A2, A3, A4, A7, E5 |
| 3. At a result | Rely on `VerifyDerivation` having mechanically verified every cited `tool_result_ref` and verbatim quote against the run's tool trajectories, with code-computed count triples (`research.artifact.*-count`). | Cannot click through from every claim on the browser UI directly to the underlying trajectory evidence body without dropping to disk (`.artifacts/research/<slug>.md`) or raw storage (`#261`). | N1, E1, E2, E3, E5 |
| 4. With missing evidence | See explicit `open_gaps` and `addressed_gaps` counts stamped by `emit_research_artifact`, and rely on `VerifyDerivation` rejecting fabricated citations (`ToolErrorInvalidArgs`). | Cannot tell from the artifact surface alone when one of `N` parallel gatherers failed prior to synthesis, because the join barrier stalls at `chain.paused.marker` with no partial-evidence receipt. | E1, E5, R1 |
| 5. After failure or replacement | See `chain.paused.marker` when a loop fails (`08-loop-failed-pause.json`). | Cannot resume or reconcile a wedged multi-gatherer run from the operator UI when 1 of `N` gatherers fails, and cancelling mid-run discards the loop without recording separate local vs. external terminal cost/effect state. | R2, R3, R4, R5 |

## 3. Findings

| Step or stage | Principle | What was observed | What the property requires | Trade-off or reason accepted |
| :--- | :--- | :--- | :--- | :--- |
| Step 01, Step 04 / Stage 1, 2 | A1 | `PendingApprovalSection.svelte` shows `tool_name`, `reason`, and JSON `arguments`, but neither submission nor approval surfaces display external data egress destinations, result visibility, or the run's resource/spending ceiling. | Submission and approval surfaces must display data egress, visibility, permitted effectful tools, and resource/spending bounds before the operator commits. | Recorded in `ADR-030` as an initial product-shell wiring slice; tracked in [SemTeams #352](https://github.com/C360Studio/semteams/issues/352). |
| Step 04 / Stage 2 | A7, A2 | Approval submission binds only to `(loopId, execution_id, decision, modified_arguments)` with a forgeable client-side `X-User-Id` (`localStorage`, default `"ui-anonymous"`). It does not record or verify the input revision or active policy revision. | Approvals must bind to action, target, input revision, policy revision, and an authenticated principal, and invalidate if any change before execution. | Documented in `ADR-030` (`Identity is forgeable on the wire`); real auth and server-side tokens deferred ([SemTeams #352](https://github.com/C360Studio/semteams/issues/352)). |
| Step 02, Branch B1 | A5, N3 | Rule `max_iterations=3` on `05-reviewer-rejected-retry.json` and `06-needs-clarification-replan.json` is scoped per triggering entity. Each re-spawned planner or reviewer loop gets a new entity ID, resetting the counter and leaving total chain-wide retries and spend unbounded. | End-to-end resource and retry bounds must cap the worst permitted path across the whole run, including retries and sub-loops. | Documented in `configs/rules/research/README.md` as partial containment for `#272` pending `semengine#24` ([SemTeams #352](https://github.com/C360Studio/semteams/issues/352)). |
| Step 08 / Stage 3 | E5 | While `emit_research_artifact` verifies and stores full provenance (`tool_result_ref`, `origin_id`, `retrieved_at`, `quote`), the browser UI does not yet render clickable provenance drill-down to the trajectory bodies. | The responsible person must be able to reach input revisions, source receipts, and active policy/prompt versions through supported product surfaces. | Known UI gap tracked in `#261`, `ADR-061`, and [SemTeams #352](https://github.com/C360Studio/semteams/issues/352). |
| Step 05, Branch B2, B3 / Stage 5 | R4, R5 | If 1 of `N` gatherers fails, rule `03b` never fires and the run parks on `chain.paused.marker` with no documented operator reconcile/resume surface (`R5`). Cancelling a task via `sendSignal("cancel")` terminates the loop without recording separate local vs. external terminal state or provider cost facts (`R4`). | The system must preserve uncertainty and provider cost facts on cancel/crash (`R4`) and provide a documented operator surface to inspect and recover stuck work (`R5`). | Tracked in `configs/rules/research/README.md` (`Partial failure caveat`), `SemEngine #24`, and [SemTeams #352](https://github.com/C360Studio/semteams/issues/352). |

## 4. Checked principles and unobserved gaps

| Step or stage | Principle | Outcome (`NO FINDING` / `UNOBSERVED`) | Inspection run or reason unobserved |
| :--- | :--- | :--- | :--- |
| Step 06 | N1 | `NO FINDING` | Inspected `cmd/semteams/tools/emitartifact/executor.go` and `derivation.go`: artifact counts (`actors-count`, `tasks-count`, `open-gaps-count`), timestamps (`retrieved_at`, `produced-at`), and citation verification are computed in deterministic Go code, never trusted to the model. |
| Step 02, 03, 07 | N2 | `NO FINDING` | Inspected `configs/rules/research/README.md` and `cmd/semteams/tools/researchdecide`: every model call maps to one of four named duties (`plan`, `gather`, `synthesize`, `reviewer-research`) with schema-validated inputs/outputs and an input guard rejecting empty or `>4` subtopics. |
| Step 06 | E2, E3, E4 | `NO FINDING` | Inspected `derivation.go` and `derivation_test.go`: `VerifyDerivation` checks `tool_result_ref` against harness-recorded trajectories in `AGENT_TRAJECTORIES`/`AGENT_CONTENT`, stamps server-authoritative `retrieved_at` and `kind` (`snippet` vs. `body` vs. `issue`/`pull_request`), and blocks `reviewer-research` from running until the checked artifact is persisted. |
| Step 03 | A6 | `NO FINDING` | Inspected `ADR-060` Addendum (2026-10-14): `bash` strips `GITHUB_TOKEN` and `*_TOKEN` env vars in the executor (`bash.go`), and `github_read` enforces the closed `SEMTEAMS_PORTFOLIO_PATH` repository allowlist in Go rather than relying on prompt rules. |
| Step 06 | R1 | `NO FINDING` | Inspected `derivation.go` and `derivation_test.go`: `DerivationError` separates model citation rejections (`ToolErrorInvalidArgs`) from NATS KV/ObjectStore outages (`ToolErrorNetwork`) and corrupt state (`ToolErrorInternal`), failing closed when trajectory storage is unreachable. |
| Step 02 | N4 | `UNOBSERVED` | `ADR-060` documents why `/work` uses zero-LLM code while `/pulse` uses multi-agent synthesis, but a pinned baseline comparison record tied to the active prompt/model configuration was not inspected in this static pass. |
| Step 03, 05 | A3, R2, R3 | `UNOBSERVED` | Static walkthrough only; mid-run policy revocation (`A3`), stale-attempt write races (`R2`), and worker kill across restarts (`R3`) require fault injection against a live disposable SemTeams + NATS instance. |
