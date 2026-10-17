# InsuCorp (IFCOM+) — Insurance CRM Modernization

**Business and Systems Analysis portfolio case based on a real recruitment take-home assignment.**

The case explores a replacement for a legacy B2B insurance CRM used by a Czech distribution network. It covers requirements, business rules, access control, acceptance criteria, and the registration approval process.

**Current stage:** the BA documents and revised BPMN are available. The next artifact is a separate registration lifecycle model; data, API, and SQL design are still planned.

> This is an anonymized portfolio extension of a written recruitment brief. No stakeholder interviews or production implementation are claimed. Assumptions and unresolved decisions are identified throughout the documentation.

## Business Context and Scope

Some Agents could not access the legacy IFCOM application and sent paper forms to a home office for manual entry. The brief identifies delays, transcription errors, duplicate company registrations, and duplicate product sales or offers.

IFCOM+ covers company registrations, Manager approval, ownership and access rules, client products, organizational administration, and four management reports. The source termination rule is **one year after the client's last product is deactivated**.

See [Project Origin and Scope](docs/00-project-origin-and-scope.md) for the source context, constraints, and portfolio boundaries.

## Artifact Guide

| Artifact | What it contains |
|---|---|
| [00 — Project origin and scope](docs/00-project-origin-and-scope.md) | Source context, evidence boundaries, limitations, and extension scope |
| [01 — Requirements catalog](docs/01-requirements-catalog.md) | Functional requirements, non-functional requirements, constraints, and stable requirement IDs |
| [02 — User stories and BDD](docs/02-user-stories.md) | Eight stories with positive, negative, authorization, boundary, and concurrency scenarios |
| [03 — RBAC and business rules](docs/03-rbac-and-business-rules.md) | Role capabilities, record scope, access decisions, and business rules |
| [04 — Assumptions and open questions](docs/04-assumptions-open-questions.md) | Proposed extensions, unresolved decisions, priorities, and dependencies |
| [BPMN source](diagrams/client-registration-approval.bpmn) | Editable BPMN 2.0 XML for Camunda Modeler |
| [BPMN preview](diagrams/client-registration-approval.png) | Image of the registration and approval process |

The `docs/` folder contains analysis documents. The `diagrams/` folder contains the current BPMN source and its PNG preview. Edit the `.bpmn` file in Camunda Modeler and export the PNG after changes so both remain aligned.

## Client Registration and Approval

![Client registration and approval BPMN](diagrams/client-registration-approval.png)

This model covers an Agent submitting a company registration, duplicate checking, initial ownership assignment, routing to a Manager, review, and the proposed outcome notifications. It has one pool and three lanes: **Agent**, **IFCOM+ System**, and **Manager**.

Read the diagram together with the [decision log](docs/04-assumptions-open-questions.md):

- **Company matching:** the duplicate-check key remains open under `OQ-001`; IČO is a proposal under `A-CRM-001`.
- **Approval policy:** `OQ-003`–`OQ-005` cover approver selection, the Manager's review access, and self-approval. The affected steps remain marked `BLOCKED`.
- **Proposed behavior:** rejection, a mandatory rejection reason, outcome notifications, and a 48-hour reminder are labeled `PROPOSED`. The reminder uses a non-interrupting boundary timer, so review continues.
- **Lifecycle boundary:** the one-year registration-termination rule belongs in a separate lifecycle model. Process outcomes do not establish approved state names; `OQ-007` remains open.

The BPMN file is an analysis model marked `isExecutable="false"`. It documents behavior and decision gaps; workflow-engine implementation is outside the current scope.

## Evidence Boundaries

Artifacts distinguish `SOURCE` requirements, `DERIVED` rules, `ASSUMPTION` proposals, and unresolved `OPEN QUESTION` items. List visibility and detail access are modeled separately; Manager approval does not silently override owner-only access.

ARES integration, IČO checksum validation, REST endpoints, SQL design, and audit events are portfolio proposals. Inclusion in a document or diagram does not establish stakeholder approval.

## Project Progress

“Documented” means an artifact exists; it does not mean stakeholder approval.

| Stage | Current position |
|---|---|
| 1 — Origin and evidence boundaries | Documented in the origin file and README |
| 2 — Requirements catalog | Draft catalog available |
| 3 — Stories, access rules, and decision log | Draft documents available; project epic still to add |
| 4 — Process and lifecycle models | Revised BPMN available; separate lifecycle model still missing |
| 5 — Systems design | Domain model / ERD, data dictionary, OpenAPI, and SQL not yet added |
| 6 — Verification and traceability | BDD scenarios exist; consolidated test scenarios and traceability matrix still planned |
| 7 — Repository presentation | Artifact navigation updated; final consistency review remains |

## Next Work

1. **Complete stage 4:** draft the registration lifecycle and transition table, including authorized actors, guards, product deactivation, and termination. Track dependencies on `OQ-006`, `OQ-007`, and `OQ-012` without presenting proposed answers as stakeholder decisions.
2. **Build the systems-design artifacts:** domain model / ERD and data dictionary, followed by the proposed API contract and SQL schema and queries.
3. **Complete coverage and review:** add the project epic requested in the original assignment, consolidate test scenarios, and connect requirements, rules, models, and tests in a traceability matrix.

The target evidence chain is:

`source brief → requirement → business rule/process → API/data constraint → test`

## Author and Contribution

**Oleksii Kim** — analysis of the written brief, requirements organization, user stories and acceptance criteria, access-policy analysis, process modeling, and documentation of assumptions and decision gaps.
