# Design Walkthrough Record

<!-- CC BY 4.0 licensed template. One copy per workflow walked. -->

| Field | Value |
| :--- | :--- |
| Workflow walked | |
| Target release tag or commit | |
| Target configuration | |
| HAAG version applied | 0.2.0 |
| Reviewer | |
| Date (UTC) | |

## 1. Step map

One row per sequential or parallel step, plus rows for failure and cancellation
branches (at minimum: policy change before pickup, crash between send and ID
write, and cancel while external work is active).

| Step | Component or boundary | Headings | Principles | Governance obligation |
| :--- | :--- | :--- | :--- | :--- |
| | | | | |

## 2. Five-stage human walkthrough

Walk one scenario through the five stages using only supported surfaces.

| Stage | Can do | Cannot do (each is a finding) | Principles |
| :--- | :--- | :--- | :--- |
| 1. Before submission | | | N2, N3, N4, A1, A5 |
| 2. At a scope or policy change | | | A2, A3, A4, A7, E5 |
| 3. At a result | | | N1, E1, E2, E3 |
| 4. With missing evidence | | | E1, E5, R1 |
| 5. After failure or replacement | | | R2, R3, R4, R5 |

## 3. Findings

| Step or stage | Principle | What was observed | What the property requires | Trade-off or reason accepted |
| :--- | :--- | :--- | :--- | :--- |
| | | | | |

## 4. Checked principles and unobserved gaps

Record principles that held (`NO FINDING`, naming what you inspected or tested)
and principles you could not test (`UNOBSERVED`, naming the missing log,
surface, or fault hook).

| Step or stage | Principle | Outcome (`NO FINDING` / `UNOBSERVED`) | Inspection run or reason unobserved |
| :--- | :--- | :--- | :--- |
| | | | |
