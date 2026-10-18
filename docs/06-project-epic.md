# IFCOM+ Project Epic

## 1. Epic Record

| Field | Value |
|---|---|
| Epic ID | `EPIC-IFCOM-001` |
| Title | Modernize distribution-network client and product management in IFCOM+ |
| Version | 1.0 — 18 October 2026 |
| Status | Documented portfolio epic; implementation readiness depends on the recorded decisions |
| Author | Oleksii Kim |
| Origin | Anonymized Business Analyst recruitment take-home assignment |
| Classification | Analyst synthesis of source requirements and derived rules; optional extensions remain separate |
| Business decision owner | Product Owner / distribution-network business owner — proposed role, no named stakeholder assigned |
| Primary users | Agent, Manager, Administrator; an authorized deputy acts within agreed delegation rules |
| Priority | Source-scope capabilities are the baseline priority; sequencing below is an analyst proposal |
| Delivery estimate | Not estimated: team, funding, release date, and measurable service targets are not supplied |
| Scope boundary | One project-level epic covering the source brief, with eight existing capability stories |

The assignment explicitly requested a project epic. This document supplies that artifact and connects the existing analysis. It does not assert that a delivery team has accepted the backlog, that the system has been implemented, or that business stakeholders have validated the proposals.

This is a broad parent epic rather than a single-sprint commitment. The eight capability stories are a starting breakdown; delivery-sized stories and tasks would be refined later.

## 2. Problem and Business Objective

The legacy IFCOM application has limited functionality, slow responses, and incomplete availability to field Agents. Some Agents complete paper forms and send them to a home office for manual entry. The supplied brief identifies transcription errors and late discovery that a company already exists or a client already has the product being offered or sold.

The objective is to support corporate-client and product management through IFCOM+, with controlled ownership and access, Manager approval, organizational administration, and management reporting.

**Epic statement — analyst synthesis**

> As the insurer's distribution-network business owner, I want Agents, Managers, and Administrators to manage company registrations, client products, and the supporting organizational data in IFCOM+, so that registration and sales information can be captured and maintained with appropriate checks, responsibilities, and visibility.

Expected business value:

- fewer repeated manual-entry steps and avoidable transcription errors;
- earlier detection of duplicate registrations and duplicate active products;
- explicit ownership and controlled registration decisions;
- consistent product and registration lifecycle handling;
- access to the four management reports required by the brief.

These are intended outcomes. No measured reduction, financial return, delivery saving, or elimination of all paper handling is claimed.

## 3. Scope and Story Breakdown

### 3.1 Included Capabilities

All linked requirements retain the classification and detailed wording in the [requirements catalog](01-requirements-catalog.md). The story IDs refer to [the existing user stories and BDD](02-user-stories.md).

| Capability | Existing story | Requirement coverage | Main unresolved dependencies |
|---|---|---|---|
| Create and verify company registrations, prevent confirmed duplicates, assign an initial owner | `US-CRM-001` | `FR-CRM-001`, `FR-CRM-002`, `FR-CRM-006`, `FR-CRM-007`, `FR-RBAC-008` | `OQ-001`, `OQ-002` |
| Search and maintain accessible registrations; provide scoped lists and My Registrations | `US-CRM-002` | `FR-CRM-003`–`FR-CRM-005`, `FR-RBAC-001`–`FR-RBAC-007` | `OQ-010`; review-access exception is separately blocked by `OQ-004` |
| Require Manager approval for new registrations | `US-APR-001` | `FR-APR-001` | `OQ-003`–`OQ-007` |
| Reassign ownership within the permitted management or deputy scope | `US-RBAC-001` | `FR-RBAC-007`, `FR-RBAC-009`, `FR-RBAC-010` | `OQ-008`, `OQ-009` |
| Record and deactivate client products; prevent confirmed active-product duplicates | `US-PROD-001` | `FR-PROD-001`–`FR-PROD-003` | `OQ-011`; interaction with registration states under `OQ-007` |
| Automatically terminate registration one year after last-product deactivation, with no active product remaining | `US-LIFE-001` | `FR-LIFE-001` | `OQ-007`, `OQ-012` |
| Maintain workers, permissions, and organizational units under the specified hierarchy | `US-IAM-001` | `FR-ORG-001`–`FR-ORG-007`, `FR-IAM-001`–`FR-IAM-011` | `OQ-013`, `OQ-014`, `OQ-017` |
| Produce all four required registration and monthly product-sales reports | `US-RPT-001` | `FR-RPT-001`–`FR-RPT-004` | `OQ-015`, `OQ-016` |

This table maps all 44 functional requirements in the current catalog. The seven cross-cutting requirements and constraints are covered below. The mapping establishes analysis coverage, not implementation or test completion.

### 3.2 Cross-Cutting Scope

| Requirement / constraint | Source expectation | Refinement still needed |
|---|---|---|
| `CON-ARCH-001` | Web application | Detailed application design |
| `CON-TECH-001` | Java according to the supplied J2EE constraint | Exact supported platform and version; the epic does not modernize the constraint silently |
| `CON-DEP-001` | Application-server environment called `Boss` in the brief | Product/version clarification under `OQ-023` |
| `NFR-AVL-001` | Availability stated as 24 hours a day, 365 days a year | Measurement, exclusions, maintenance, recovery targets, and acceptance under `OQ-018` |
| `NFR-CAP-001` | Approximately 20 processed registrations per day | Peaks, concurrent usage, data volume, and response-time targets under `OQ-019` |
| `NFR-L10N-001` | Czech and Slovak language variants | Content scope, formatting, fallback, and ownership under `OQ-020` |
| `NFR-DEV-001` | Desktop, laptop, and tablet support | Supported browsers, screens, and interactions under `OQ-021` |

The organizational baseline includes four Regions, one or more Sales Points per Region, and the specified worker/Manager relationships. Phase 1 stores organizational, worker, role, and authorization data within IFCOM+ without integration to an external identity and authorization repository. Manager access to the administration area does not automatically grant Administrator maintenance capabilities.

### 3.3 Conditional Extensions

The following are separately tracked proposals, not mandatory source scope simply because they appear in a model:

- rejection, rejection reason, outcome notifications, and the 48-hour reminder: `A-APR-001`–`A-APR-004`;
- IČO as the duplicate-check key and ARES enrichment: `A-CRM-001`, `A-INT-001`;
- a scheduled background mechanism for closure: `A-LIFE-001`;
- detailed lifecycle choices, including product activation only after approval and inactivity-interval reset: `LC-P01`–`LC-P05` in [the lifecycle document](05-registration-lifecycle.md);
- other proposed security, performance, usability, and API details in the catalog and decision log.

An extension is included in delivery only through an explicit scope decision, with dependent criteria updated accordingly.

### 3.4 Exclusions

The portfolio does not include a working production application, deployment or operational support, a production migration, insurance pricing or underwriting, claims, billing, payment processing, or formal regulatory/security certification.

The later ERD, API, and SQL artifacts are analysis and design evidence. Their presence will not establish production delivery. Detailed limits are recorded in [Project Origin and Scope](00-project-origin-and-scope.md).

## 4. Epic Acceptance Criteria

These are proposed delivery acceptance criteria derived from the baseline. They describe evidence a future implementation would need; they have **not** been executed or passed in this portfolio. Detailed scenarios remain in the linked stories rather than being duplicated here.

| ID | Acceptance condition | Verification and traceability |
|---|---|---|
| `EAC-01` | An authorized Agent or Manager can create a company registration, assign its initial owner, and prevent a confirmed duplicate under the agreed matching rule. Verification of an existing company has explicitly defined behavior. | `US-CRM-001`; valid, invalid, duplicate, and concurrent-creation scenarios; `OQ-001`/`OQ-002` must be resolved. |
| `EAC-02` | The applicable registration lists and My Registrations return only the allowed population. Listing a record does not grant detail or edit access; those actions enforce the owner rule and any explicitly approved review exception. | `US-CRM-002`, `FR-RBAC-001`–`FR-RBAC-007`; positive/negative access checks and scope-isolation examples. |
| `EAC-03` | A new registration cannot complete its approval workflow without a valid Manager decision under the agreed approver, review-access, and self-approval policy. Each approval step has at most one final decision. | `US-APR-001`, `BR-APR-001`, `BR-APR-002`; unauthorized, repeat, and concurrent-decision cases; `OQ-003`–`OQ-005` block final acceptance. |
| `EAC-04` | A Manager or valid deputy can reassign ownership only from the allowed source population to an allowed target. Invalid or revoked delegation does not grant reassignment. | `US-RBAC-001`, `FR-RBAC-009`, `FR-RBAC-010`; source, target, and delegation boundary cases after `OQ-008`/`OQ-009`. |
| `EAC-05` | Authorized product operations maintain the client-product relationship, record effective deactivation, and prevent confirmed active-product duplicates using agreed product identity and activity rules. | `US-PROD-001`, `FR-PROD-001`–`FR-PROD-003`; duplicate, authorization, and effective-date cases after `OQ-011`. |
| `EAC-06` | Registration terminates automatically at the agreed one-year boundary after last-product deactivation, with no active product remaining. It does not terminate early or based on an obsolete inactivity interval. | `US-LIFE-001`, `BR-LIFE-001`–`BR-LIFE-005`; boundary and active-product scenarios, plus chosen reactivation behavior after `OQ-007`/`OQ-012`. |
| `EAC-07` | An Administrator can maintain workers, application permissions, and organizational units while preserving the agreed hierarchy and handling dependent registrations, reports, and access when workers transfer or units deactivate. | `US-IAM-001`, `FR-ORG-001`–`FR-ORG-007`, `FR-IAM-001`–`FR-IAM-011`; integrity, permission-change, and dependency scenarios after relevant IAM decisions. |
| `EAC-08` | All four reports are available to the appropriate Manager and reconcile to an agreed reference dataset: registrations by worker and unit, and monthly product sales by worker and unit. | `US-RPT-001`, `FR-RPT-001`–`FR-RPT-004`; counting, time-zone, attribution, and scope-isolation cases after `OQ-015`/`OQ-016`. |
| `EAC-09` | Agreed Czech/Slovak content and desktop/laptop/tablet support are demonstrated against the accepted language and device matrix. | `NFR-L10N-001`, `NFR-DEV-001`; localization and device checks after `OQ-020`/`OQ-021`. |
| `EAC-10` | The design and implementation conform to the clarified platform constraints and satisfy measurable availability and workload criteria derived from the source expectations. | `CON-ARCH-001`, `CON-TECH-001`, `CON-DEP-001`, `NFR-AVL-001`, `NFR-CAP-001`; architecture review and agreed operational/performance evidence after `OQ-018`, `OQ-019`, `OQ-023`. |

A rejected access attempt must not disclose another owner's protected registration data. The existing [policy model](03-rbac-and-business-rules.md) supplies the detailed rules; this epic does not override them.

## 5. Outcome Measures

The measures below are **analyst proposals for future validation**. The brief supplies no baseline measurements or improvement targets. No current results are claimed, and the four source-required management reports remain distinct from these proposed product-success measures.

| Measure | Proposed definition / evidence | Baseline and target |
|---|---|---|
| Registration completion time | Elapsed time from a consistently defined accepted submission to final approval, segmented to distinguish work from waiting | Not measured; event definitions, observation window, and target require agreement |
| Registration correction rate | Proportion of accepted submissions requiring later correction, with correction reasons categorized | Not measured; denominator, reasons, and target require agreement |
| Late duplicate discovery | Confirmed company or active-product duplicates discovered after creation/recording, using the approved identity rules | Not measured; matching definition and target require agreement |
| Direct capture adoption | Proportion of in-scope submissions entered directly by authorized sales users rather than re-entered from paper | Not measured; in-scope population and adoption target require agreement |
| Lifecycle processing timeliness | Difference between the agreed closure boundary and the actual effective closure, including failed/retried processing | Not measured; allowed delay and measurement rule require agreement |

Functional acceptance of the agreed rules is separate from proving business improvement after deployment. Availability and response-time measures belong to the service requirements and their open decisions, not invented percentage claims in this epic.

## 6. Dependencies, Risks, and Decision Ownership

The following owners are **proposed decision-making roles**, not people who were interviewed or who have approved this case. The [decision log](04-assumptions-open-questions.md) remains authoritative for question status.

| Dependency / risk | Required decision or response | Existing references / proposed owner |
|---|---|---|
| Approval conflicts with owner-only detail access; approver and self-approval are undefined | Define the review view, authorized approver, and self-approval rule before finalizing approval behavior | `OQ-003`–`OQ-005`; Product Owner, Security Owner, Risk Owner |
| Incomplete company and product identity rules permit duplicate or wrongly rejected records | Define matching, normalization, product activity, and exception behavior before final keys/constraints | `OQ-001`, `OQ-002`, `OQ-011`; Product Owner / Data Owner |
| Undefined lifecycle dates or concurrent activation cause incorrect termination | Define states, one-year calculation, time zone, and activation/reset/closure ordering | `OQ-007`, `OQ-012`; Product Owner |
| Delegation, worker transfers, unit deactivation, or reporting attribution leak access or distort counts | Agree source/target/dependency scopes and report definitions | `OQ-008`–`OQ-010`, `OQ-013`–`OQ-016`; business, data, reporting, and security owners |
| IAM, privacy, and operational controls are insufficiently specified | Define account/access controls, data retention, service targets, and the supported platform | `OQ-017`–`OQ-023`; Security/Privacy/Service owners, Architect, UX/Product owners as appropriate |
| One broad epic hides delivery size or introduces attractive optional features as mandatory | Split capabilities into delivery-sized stories, preserve extension classifications, and estimate only after scope refinement | This epic, the story inventory, and the assumption register; Product Owner / delivery team |

Exploratory ERD and API work can proceed with explicit working assumptions. These unresolved decisions prevent claiming that the affected specifications are implementation-ready.

## 7. Proposed Delivery Sequence

This is a dependency-based sequence for discussion, not an approved release plan or a schedule.

| Slice | Focus | Dependencies / reason |
|---|---|---|
| 1 | Organization, workers, permissions, and agreed access foundations | Enables meaningful ownership and scoped operations; clarify IAM and administration policy |
| 2 | Registration creation, verification, search, ownership transfer, and Manager approval | Depends on company matching, list scopes, delegation, and the approval decisions |
| 3 | Client-product operations and automatic registration termination | Depends on product identity and approved lifecycle/date semantics |
| 4 | The four reports and end-to-end acceptance | Depends on agreed business events, counted states, attribution, and representative data |

Localization, device support, authorization, data integrity, and service-quality requirements apply throughout the slices. They are not postponed until slice 4. Optional rejection/reminder/ARES features enter a slice only after a scope decision.

No budget, team capacity, story points, delivery dates, or committed MVP boundary are inferred from this sequence.

## 8. Readiness and Completion

### 8.1 Ready for Implementation Refinement

Before affected stories can be treated as ready:

- acceptance-changing questions have recorded answers, or a clearly bounded proposed alternative is identified for portfolio design;
- the business approver, access policy, company/product identity, and lifecycle semantics are internally consistent;
- necessary quality, security, privacy, platform, language, and device criteria are measurable or explicitly constrained;
- stories are small enough for the intended delivery approach and have concrete examples;
- the domain model, interfaces, data constraints, and verification approach align with the selected rules.

A proposed portfolio alternative permits analysis work; it is not a substitute for stakeholder approval in a real delivery.

### 8.2 Product Delivery Complete

A future delivery could be accepted only when:

- all in-scope epic criteria have traceable implementation and test evidence;
- applicable negative, authorization, boundary, and concurrency scenarios pass;
- all four reports reconcile to agreed data and definitions, and any adopted outcome measures have an agreed measurement method;
- the selected operational and non-functional criteria are demonstrated;
- required business/service owners accept the result and outstanding defects or exclusions have explicit dispositions.

These conditions are not met merely by adding documents or diagrams to GitHub.

### 8.3 Portfolio Artifact Complete

This epic artifact is complete when its business objective, scope, story mapping, acceptance criteria, dependencies, proposed sequence, and evidence boundaries are documented and navigable. This version supplies those elements. The wider portfolio still needs the planned system-design and consolidated traceability artifacts.

## 9. Related Artifacts and Next Step

| Artifact | Role in this epic |
|---|---|
| [00 — Project origin and scope](00-project-origin-and-scope.md) | Establishes source context and portfolio boundaries |
| [01 — Requirements catalog](01-requirements-catalog.md) | Defines the baseline and proposal classifications |
| [02 — User stories and BDD](02-user-stories.md) | Breaks the epic into eight capability stories and detailed scenarios |
| [03 — RBAC and business rules](03-rbac-and-business-rules.md) | Defines access decisions and domain rules |
| [04 — Assumptions and open questions](04-assumptions-open-questions.md) | Tracks unresolved decisions and proposed extensions |
| [05 — Registration lifecycle](05-registration-lifecycle.md) | Proposes state transitions and closure semantics |
| [Registration/approval BPMN](../diagrams/client-registration-approval.bpmn) | Models the Agent-initiated registration workflow |
| [Registration lifecycle UML](../diagrams/client-registration-lifecycle.puml) | Models the registration object's states and transitions |

The next portfolio artifact is the **domain model / logical ERD and data dictionary**. It should derive entities, relationships, cardinalities, keys, and ownership/lifecycle attributes from the existing baseline, while keeping unresolved identity and lifecycle choices visible.

After the data model, the planned sequence is API contracts, SQL constraints/queries, and consolidated test scenarios and traceability.
