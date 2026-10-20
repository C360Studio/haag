# HAAG: Human-Agent Accountability and Governance

**An agentic governance framework. Responsibility can be delegated.
Accountability cannot.**

HAAG (Human-Agent Accountability and Governance) governs systems that mix
deterministic software with model or agent steps. It is built on four
questions remembered as **NEAR**: Necessity, Evidence, Authority, Recovery.

Gateways govern the pipe. Loop middleware hooks the call. HAAG governs the
workflow and the human decision.

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

## Where to start (if you have an hour)

You do not need to adopt all twenty-two principles at once or run a formal
three-person operator study to get value out of this repository:

1. **Steal the workflow map and five-stage walkthrough** in
   [protocol/design-walkthrough.md](protocol/design-walkthrough.md). Map one
   workflow including its policy-revocation, crash-after-send, and cancellation
   branches.
2. **Check the five highest-leverage seams on your next PR:**
   - **N1 (Deterministic authority):** Never let a model be the sole authority
     for a number, calculation, or state transition that code can compute or
     verify.
   - **E2 (Observations distinct from claims):** Keep harness-recorded facts
     (tool calls, HTTP status, cost, state transitions) structurally and
     visually separate from model prose.
   - **A7 (Approval bound to a revision):** Bind human approvals to the exact
     action, target, input revision, and policy revision so a changed target or
     input invalidates the approval.
   - **R2 (No duplicate or stale effects):** Derive idempotency keys from the
     logical workflow step—never the retry attempt ID—and guard state writes
     against stale attempts.
   - **R4 (No invented zeros):** When a worker crashes after sending an
     external request or a user cancels mid-flight, record `uncertain`—never
     `0` cost or `no effect`.
3. **Run H1 before a major release** ([protocol/h1-operator.md](protocol/h1-operator.md)):
   sit one engineer who did not build the feature down with four sealed cases
   and watch whether they can govern it from the product surface alone.

## What is here

| Path | Contents |
| :--- | :--- |
| [spec/haag.md](spec/haag.md) | The specification: twenty-two principles in property / observe / example-mechanisms form, and positioning across governance layers. |
| [protocol/design-walkthrough.md](protocol/design-walkthrough.md) | Fast architecture and PR audit: map a workflow step by step, then walk one scenario through five stages. |
| [protocol/h1-operator.md](protocol/h1-operator.md) | Supplement H1: a blind human-operator exercise against a pinned release. |
| [protocol/templates/](protocol/templates/) | Copy-paste templates for the design walkthrough record and the H1 reference sheet, operator packet, and case scoring record. |
| [IMPLEMENTATIONS.md](IMPLEMENTATIONS.md) | Where the principles are implemented, and where the framework came from. |
| [CHANGELOG.md](CHANGELOG.md) | Normative and editorial changes by version. |

## Prior art and related work

HAAG builds on ideas from agentic governance research, human-factors studies,
capability security, and distributed systems:

### Research papers

- **Agentic governance and visibility:** Yonadav Shavit et al.,
  [*Practices for Governing Agentic AI Systems*](https://cdn.openai.com/papers/practices-for-governing-agentic-ai-systems.pdf)
  (OpenAI, 2023); Alan Chan et al.,
  [*Visibility into AI Agents*](https://arxiv.org/abs/2401.13138)
  (ACM FAccT 2024); Noam Kolt,
  [*Governing AI Agents*](https://arxiv.org/abs/2501.07913)
  (Notre Dame Law Review, 2025).
- **Human oversight and moral crumple zones:** Madeleine Clare Elish,
  [*Moral Crumple Zones: Cautionary Tales in Human-Robot Interaction*](https://doi.org/10.17351/ests2019.260)
  (2019); Ben Green and Yiling Chen,
  [*The Principles and Limits of Algorithm-in-the-Loop Decision Making*](https://www.benzevgreen.com/wp-content/uploads/2019/09/19-cscw.pdf)
  (PACMHCI / CSCW 2019).
- **Capability security and prompt-injection isolation:** Edoardo Debenedetti
  et al.,
  [*Defeating Prompt Injections by Design (CaMeL)*](https://arxiv.org/abs/2503.18813)
  (2025).
- **Durable execution and uncertain external state:** Pat Helland,
  [*Life beyond Distributed Transactions: an Apostate's Opinion*](https://www.cidrdb.org/cidr2007/papers/cidr07p15.pdf)
  (CIDR 2007 / ACM Queue 2016).

### Open-source specifications, toolkits, and taxonomies

- **[OWASP Top 10 for Agentic Applications](https://genai.owasp.org/):**
  threat taxonomy covering goal hijack, tool misuse, privilege escalation,
  memory poisoning, and runaway loops.
- **[Microsoft Agent Governance Toolkit & Agent Control Specification (ACS)](https://github.com/microsoft/agent-governance-toolkit):**
  open-source runtime middleware intercepting model and tool calls inside the
  agent loop.
- **[CaMeL (`google-research/camel-prompt-injection`)](https://github.com/google-research/camel-prompt-injection):**
  reference implementation of capability-tagged data flows for tool-using
  agents.
- **[Awesome AI Governance (`agentrust-io/awesome-ai-governance`)](https://github.com/agentrust-io/awesome-ai-governance):**
  community index of agent governance specifications, cryptographic policy
  receipts, and coding-agent audit harnesses.
- **[GATES™ AI Governance Framework](https://www.druhindhavala.com/ai-governance/gates-framework/):**
  coined by Druhin Dhavala, October 10, 2026 (CC BY 4.0). Cited in the
  specification as a published example of a perimeter gateway framework.

## Status

0.2.0. Principle identifiers (N1 through R5) are frozen as of the 0.2.0
release. Open an issue to propose a change; normative changes are tracked in
the [changelog](CHANGELOG.md).

## Credits

- GATES™ AI Governance Framework — coined by Druhin Dhavala, October 10, 2026.
  <https://www.druhindhavala.com/ai-governance/gates-framework/>. Licensed
  under CC BY 4.0.
- Supplement H1 originated as Golden Supplement N1 in the C360 Studio
  SemStreams project. See [IMPLEMENTATIONS.md](IMPLEMENTATIONS.md).

## License

Everything in this repository is licensed under the Creative Commons
Attribution 4.0 International License ([CC BY 4.0](LICENSE)). You may share
and adapt it for any purpose, including commercially, with attribution.

Suggested attribution:

> HAAG (Human-Agent Accountability and Governance): an agentic governance
> framework, by Complete 360 Studio, licensed under CC BY 4.0.
> <https://github.com/C360Studio/haag>
