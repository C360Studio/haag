# Changelog

All notable changes to the specification and protocols are recorded here.
Normative changes (anything that alters what counts as a finding) are marked
**normative**.

## 0.2.0 - 2026-10-20

- Renamed the framework to **HAAG (Human-Agent Accountability and Governance)**
  (`spec/haag.md`) and renamed the blind human-operator exercise to
  **Supplement H1** (`protocol/h1-operator.md`) so `N1` refers only to the
  specification principle.
- Dropped the `-draft` suffix and froze principle identifiers `N1` through
  `R5`.
- Added a practitioner quick-start guide and prior art references (papers,
  threat taxonomies, and open-source governance toolkits) to `README.md`, and
  expanded Section 3 of `spec/haag.md` to position HAAG alongside threat
  taxonomies (OWASP Agentic Top 10), perimeter gateways (GATES), and in-process
  loop middleware (Microsoft ACS).
- Rewrote the specification and protocols in direct practitioner language,
  restoring N1's core deterministic-authority rule and simplifying the design
  walkthrough (`protocol/design-walkthrough.md`) and H1 scoring templates
  (`protocol/templates/`).
- Fixed Reference Sheet sealing in H1 so the SHA-256 digest is recorded in a
  detached manifest and case scoring records rather than inside the hashed file
  itself.
- **normative** Added **A7 (Approval bound to a revision)**: human approvals
  bind to the action, target, input revision, policy revision, and authorizing
  principal, and are invalidated if any of the five changes before execution.
- **normative** **A2** requires that untrusted or model-derived data cannot
  supply the target or parameters of an effectful tool call without a
  deterministic policy check at the enforcement point or a revision-bound
  approval (A7).
- **normative** **E4** extends downstream lineage to persistent agent memory
  and knowledge stores alongside downstream model duties and user-facing
  artifacts.
- **normative** **Section 1.1** makes any supported surface that looks complete
  on paper but is unusable by the responsible operator a finding against the
  principle governing that surface (preventing "moral crumple zone" designs).
- **normative** **Section 1.2** defines the three review outcomes (`Finding`,
  `No finding`, `Unobserved`) and requires fault-injection or runtime checks
  for **E1, E4, E5, A2, A3, A5, A6, A7, R1, R2, R3,** and **R4** (including
  stale-attempt write races in R2 and logical-effect idempotency keys).
- **normative** **N4** binds the baseline comparison record to the active model,
  prompt, and check configuration and requires re-comparison when they change
  materially.
- **normative** **E1** scopes the four-axis verification labels to boundaries
  carrying external, untrusted, or model-derived data; pure control calls are
  out of scope.
- **normative** **E2** requires the harness to record every model invocation
  and effectful tool call as an observation distinct from model claims.
- **normative** **E6** scopes retention to declared consequential decision
  classes and requires auditability rather than deterministic reproduction of
  model steps.

## 0.1.0-draft - 2026-10-20

Initial public draft, published as ONUS under CC BY 4.0.

- Specification: four headings, twenty-one principles (N1-N4, E1-E6, A1-A6,
  R1-R5) in property / observe / example-mechanisms form.
- Design and workflow walkthrough protocol.
- Supplement N1 blind human-operator protocol with three templates.
- Positioning relative to perimeter gateway frameworks.
