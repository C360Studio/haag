# Implementations and History

This file records where ONUS responsibilities are implemented and where the
framework came from. It changes more often than the specification and is kept
separate so the specification can stay architecture-neutral.

## C360 Studio ecosystem

Within the public C360 Studio ecosystem, ONUS responsibilities are factored
across architectural boundaries rather than bundled into one governance
binary.

- **ONUS specification and protocols (this repository).** Owns the four
  observation questions, the design walkthrough rubric, and the H1 blind
  human-operator protocol.
- **[SemEngine](https://github.com/C360Studio/semengine).** Owns the generic,
  domain-neutral substrate: knowledge graph, NATS transport, revision-guarded
  mutations, rule evaluation, and durable lifecycle and attempt settlement
  ([SemEngine #24](https://github.com/C360Studio/semengine/issues/24)). These
  implement the Evidence and Recovery properties at the storage and execution
  layer.
- **[SemTeams](https://github.com/C360Studio/semteams).** Owns the agentic
  runtime, category packs (including the absorbed
  [SemDev](https://github.com/C360Studio/semdev) issue-to-PR workflows), and
  the operator surface
  ([ADR-061](https://github.com/C360Studio/semteams/blob/main/docs/adr/061-agent-runtime-ownership-and-observable-work.md)).
  These implement stable attempt identity, revision-bound approvals,
  separation of harness facts from model claims, and operator recovery from
  interrupted work.

## History

- **SemStreams (origin).** The blind-built edge-triage golden and Golden
  Supplement N1 (now Supplement H1) originated in [SemStreams](https://github.com/C360Studio/semstreams)
  ([issue #1367](https://github.com/C360Studio/semstreams/issues/1367),
  PR #1368) before the substrate was extracted into SemEngine and the agentic
  runtime consolidated into SemTeams.
- **2026-10-20.** Specification and protocols published as 0.1.0-draft in
  this repository.

## Adding an implementation

Open a pull request adding one entry naming the project, which principles it
implements, and where a reader can observe them. Entries describe what a
project implements. They do not claim conformance, because ONUS defines none.
