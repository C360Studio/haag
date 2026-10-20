# Changelog

All notable changes to the specification and protocols are recorded here.
Normative changes (anything that alters what counts as a finding) are marked
**normative**.

## Unreleased

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

## 0.1.0-draft - 2026-10-20

Initial public draft, published as ONUS under CC BY 4.0.

- Specification: four headings, nineteen principles (N1-N4, E1-E6, A1-A6,
  R1-R5) in property / observe / example-mechanisms form.
- Design and workflow walkthrough protocol.
- Supplement N1 blind human-operator protocol with three templates.
- Positioning relative to perimeter gateway frameworks.
