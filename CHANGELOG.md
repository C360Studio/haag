# Changelog

All notable changes to the specification and protocols are recorded here.
Normative changes (anything that alters what counts as a finding) are marked
**normative**.

## Unreleased

- **normative** Property is the only place a requirement is stated; Observe
  tests it and introduces none. The change policy now counts any
  finding-altering edit to Property or Observe as normative. (Issue #5, 1)
- **normative** E2 requires every model invocation and effectful tool call to
  be recorded by the harness, which is what N2's Observe relied on. A4's
  Observe no longer cites A2 for a delegate's policy source. (Issue #5, 1)
- **normative** A supported surface a responsible person demonstrably cannot
  use is a finding even when structurally complete. Section 1.1, E2, A1, the
  walkthrough rubric, and operator protocol section 5.4 state the rule and
  the evidence required. (Issue #5, 2)
- **normative** New A7, approval bound to a revision: action, target, input
  revision, policy revision, and principal, invalidated on change. (Issue
  #5, 7)
- **normative** Must/Should distinction removed; every property is a
  requirement. (Issue #5, 6)
- Principle count corrected to twenty-two with A7 added. (Issue #5, 4)

- **normative** Section 1.2 defines the reviewer's vantage (records,
  surfaces, configuration, a disposable instance, fault injection) and adds a
  third outcome, unobserved, alongside finding and no finding. Observations
  quantify over the step map and record coverage.
- **normative** N1 restated as a data-path property: every numeric or
  invariant-bearing model value passes a code check before use. The
  "could code have computed it" question moves to N4.
- **normative** N4 requires the decision to rely on a duty to cite the
  comparison result.
- **normative** E2 and A1 restated as structural and display properties;
  whether a person can tell or understood is measured by the N1 protocol.
- **normative** E6 scopes retention to declared consequential decision
  classes and replaces "reproduce" with "audit".
- **normative** A4 requires scopes to be recorded in a comparable form.
- **normative** R4 replaces "all known cost facts" with the facts the provider
  reports at termination time.
- Observe paragraphs for N2, E1, E4, E5, A2, A5, A6 and R4 now name a bounded
  test or an unobserved condition instead of a reading.
- Design walkthrough protocol records all three outcomes (finding, no
  finding, unobserved) and a coverage record, matching specification
  section 1.2.
- N1 protocol adds a per-case property record: the Reference Sheet declares
  which properties each case exercises before the session, and the scorer
  records finding, no finding, or unobserved for each. Templates updated.

## 0.1.0-draft - 2026-10-20

Initial public draft, published as ONUS under CC BY 4.0.

- Specification: four headings, twenty-one principles (N1-N4, E1-E6, A1-A6,
  R1-R5) in property / observe / example-mechanisms form.
- Design and workflow walkthrough protocol.
- Supplement N1 blind human-operator protocol with three templates.
- Positioning relative to perimeter gateway frameworks.
