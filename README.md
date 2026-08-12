# InsuCorp (IFCOM+) — Insurance CRM Modernization

**Business and Systems Analysis portfolio case based on a real recruitment take-home assignment.**

This repository documents the analysis of a proposed replacement for a legacy B2B insurance CRM used by a Czech distribution network. The case focuses on requirements engineering, process modeling, role-based access rules, client and product lifecycle management, and selected system-design extensions.

> [!IMPORTANT]
> This is an anonymized portfolio case, not a production implementation or client engagement. The source was a written recruitment brief; no stakeholder interviews were conducted. Independently proposed details are labeled as assumptions and require validation before implementation.

## Case at a Glance

| Item | Description |
|---|---|
| Domain | B2B insurance and distribution-network CRM |
| Scenario | Modernization of the legacy IFCOM application into IFCOM+ |
| Primary users | Agent, Manager, Administrator |
| Core analysis areas | Requirements, business rules, RBAC, BPMN, BDD, data and integration proposals |
| Source | Anonymized Business Analyst recruitment take-home brief |
| Author | Oleksii Kim |
| Status | Portfolio case in active development |

## Project Origin

The source brief described an insurer whose legacy IFCOM application was slow, functionally limited, and unavailable to some field agents. Agents without system access had to complete paper forms and send them to a home office for manual entry. This created delays, transcription errors, duplicate client registrations, and cases where an existing product was offered or sold again.

The original assignment asked the candidate to:

- identify and organize functional and non-functional requirements;
- divide the requirements into business modules;
- convert selected requirements into user stories;
- define acceptance criteria; and
- define a project epic.

I extended the assignment independently into a broader BA/SA portfolio case. For the complete evidence boundary and classification rules, see [Project Origin and Scope](docs/00-project-origin-and-scope.md).

## Business Objective and Scope

IFCOM+ is intended to replace manual hand-offs with a web-based tool for managing corporate clients, their products, the distribution structure, and its sales personnel. The source-defined scope includes:

- creating, verifying, searching, viewing, and editing company registrations;
- recording and deactivating client products while addressing duplicate-product risk;
- routing new registrations for Manager approval;
- enforcing owner-, role-, and hierarchy-based visibility and actions;
- allowing authorized ownership reassignment by a Manager or deputy;
- administering workers, permissions, and organizational units;
- producing four reports on registrations and monthly product sales;
- terminating a registration one year after the client's last product is deactivated; and
- supporting Czech and Slovak users on desktop computers, laptops, and tablets.

The brief also specifies a Java/J2EE web application, continuous availability, and approximately 20 registrations per day. Where it provides no measurable acceptance threshold, the constraint remains subject to refinement rather than being presented as a validated SLA.

## Analysis Approach

To preserve traceability, repository artifacts use the following classifications:

| Classification | Meaning |
|---|---|
| `SOURCE` | Explicitly stated in the recruitment brief |
| `DERIVED` | Logically elaborated from source information without changing its intent |
| `ASSUMPTION` | Proposed because the source does not provide sufficient detail |
| `OPEN QUESTION` | Requires a stakeholder or Product Owner decision |

This prevents an independently proposed system design from being presented as an approved business requirement.

## Current Artifacts

| Artifact | Purpose | Link |
|---|---|---|
| Project origin and scope | Defines the source, evidence boundary, classifications, limitations, and out-of-scope areas | [Open document](docs/00-project-origin-and-scope.md) |
| BPMN process model | Proposed new-client registration and approval workflow | [View diagram](diagrams/client_registration_flow.png) |
| Editable BPMN source | Draw.io source for the process model | [Open source](docs/InsuCorp.drawio) |

## BPMN: Client Registration and Approval

The current process model covers registration entry, duplicate checking, Manager review, decision handling, status updates, and Agent notification.

![Client Registration and Approval BPMN](diagrams/client_registration_flow.png)

The rejection path and 48-hour Manager reminder shown in the current model are analyst-proposed extensions. They are not stated in the source brief and must therefore be treated as assumptions. The reminder should not interrupt the Manager's review task.

## Proposed System Extensions

The following ideas are included or planned as portfolio-level system proposals rather than source requirements:

- IČO checksum validation;
- integration with the Czech ARES register for company-data enrichment;
- REST API design and HTTP error handling;
- duplicate-prevention constraints at the database level;
- a 48-hour Manager reminder;
- a rejection path with a mandatory rejection reason;
- explicit registration lifecycle states;
- audit events and negative authorization scenarios.

Each proposal must be evaluated against business value, privacy, security, architecture, and operational constraints before implementation.

## Key Analytical Decisions

1. **Separate evidence from proposals.** Source requirements, derived rules, assumptions, and unresolved questions are classified explicitly.
2. **Keep list visibility separate from record access.** Being able to see a registration in a list does not automatically grant permission to open or edit its details.
3. **Model client and registration lifecycles separately.** The source rule is not a 30-day pending-approval timeout: a registration is terminated one year after the last client product is deactivated.
4. **Apply least-privilege thinking without claiming compliance.** RBAC supports controlled access, but it does not by itself demonstrate GDPR or security compliance.
5. **Protect sensitive ownership information.** A duplicate-registration response should not expose another Agent's identity unless the requesting user is authorized to see it.

## My Contribution

I independently:

- analyzed the written AS-IS and TO-BE brief;
- extracted and organized business and system requirements;
- identified missing information, assumptions, and open questions;
- drafted role- and hierarchy-based access rules;
- modeled the proposed registration and approval process in BPMN 2.0;
- drafted BDD-style acceptance criteria;
- explored data-validation and external-integration options; and
- defined the next artifacts required for end-to-end traceability.

## Planned Next Steps

- complete the modular functional and non-functional requirements catalog;
- add user stories with positive, negative, boundary, and authorization scenarios;
- create a full RBAC and business-rules decision table;
- refine the BPMN model and add a registration lifecycle state model;
- add a domain model, proposed OpenAPI 3.1 specification, and SQL schema;
- add test scenarios and a source-to-test traceability matrix.

The target evidence chain is:

`source brief → requirement → business rule/process → API/data constraint → test`
