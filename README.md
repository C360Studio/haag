# HAAG: Human-Agent Accountability and Governance

**An agentic governance framework. Responsibility can be delegated.
Accountability cannot.**

HAAG (Human-Agent Accountability and Governance) governs systems that mix
deterministic software with model or agent steps. It is built on four
questions remembered as **NEAR**: Necessity, Evidence, Authority, Recovery.

Gateway frameworks govern the pipe. HAAG governs the workflow and the human
decision.

## The four questions

| Heading | Question |
| :--- | :--- |
| **Necessity** | Why does this step need a model, and how will its result be checked? |
| **Evidence** | What happened, what supports the result, and can the responsible person inspect it? |
| **Authority** | Who permits the action, what may it do, and where must it stop? |
| **Recovery** | What is confirmed after a failure, and what is the next safe action? |

The test across all four is **human use**: can the responsible person
understand, act, distinguish fact from claim, and recover, without reading
source code or reconstructing a workflow from raw logs?

HAAG produces findings, not scores. It defines no conformance level, badge, or
composite number.

## What is here

| Path | Contents |
| :--- | :--- |
| [spec/haag.md](spec/haag.md) | The specification: twenty-two principles in property / observe / example-mechanisms form, and positioning relative to gateway frameworks. |
| [protocol/design-walkthrough.md](protocol/design-walkthrough.md) | Design-time audit: map a workflow step by step, then walk one scenario through five stages. |
| [protocol/h1-operator.md](protocol/h1-operator.md) | Supplement H1: a blind human-operator exercise against a pinned release. |
| [protocol/templates/](protocol/templates/) | Design walkthrough record, and the H1 reference sheet, operator packet, and case scoring record. |
| [IMPLEMENTATIONS.md](IMPLEMENTATIONS.md) | Where the principles are implemented, and where the framework came from. |
| [CHANGELOG.md](CHANGELOG.md) | Normative and editorial changes by version. |

## How to use it

1. Read the specification once. Each principle states a property, what to
   observe, and example mechanisms. Only the property is normative.
2. Run the design walkthrough on one representative workflow. Record findings
   against principle identifiers.
3. Before a release, run H1 with a person who did not build the system.
   Record per-case outcomes and friction counts. Do not average them.

## Status

0.1.0-draft. Principle identifiers freeze at the 0.1.0 release, when the
draft suffix drops; until then they may change. Open an issue to propose a
change; normative changes are tracked in the changelog.

## Credits

- GATES™ AI Governance Framework — coined by Druhin Dhavala, October 10, 2026.
  <https://www.druhindhavala.com/ai-governance/gates-framework/>. Licensed
  under CC BY 4.0. Cited in the specification as a published example of a
  perimeter gateway framework.
- Supplement H1 originated as Golden Supplement N1 in the C360 Studio
  SemStreams project. See
  [IMPLEMENTATIONS.md](IMPLEMENTATIONS.md).

## License

Everything in this repository is licensed under the Creative Commons
Attribution 4.0 International License ([CC BY 4.0](LICENSE)). You may share
and adapt it for any purpose, including commercially, with attribution.

Suggested attribution:

> HAAG (Human-Agent Accountability and Governance): an agentic governance
> framework, by Complete 360 Studio, licensed under CC BY 4.0.
> <https://github.com/C360Studio/haag>
