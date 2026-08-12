# IFCOM+ Requirements Catalog

## 1. Document Control

| Field | Value |
|---|---|
| Document status | Draft baseline for portfolio development |
| Version | 0.1 |
| Author | Oleksii Kim |
| Primary source | Anonymized Business Analyst recruitment take-home brief |
| Secondary source | Original candidate case-study submission |
| Related document | [Project Origin and Scope](00-project-origin-and-scope.md) |
| Supersedes | `docs/business_requirements.md` |

This catalog establishes the requirements baseline for the IFCOM+ portfolio case. It corrects the previous interpretation of the registration-termination rule, restores requirements omitted from the short BRD, and separates source-backed requirements from independently proposed extensions.

The source rule is:

> A client registration is terminated one year after the client's last product is deactivated.

It is **not** a 30-day timeout for registrations awaiting approval.

## 2. Classification and Priority Rules

### 2.1 Evidence Classification

| Classification | Meaning |
|---|---|
| `SOURCE` | Explicitly stated in the recruitment brief |
| `DERIVED` | Necessary or logically elaborated from source information without changing its intent |
| `ASSUMPTION` | Proposed because the source does not provide sufficient detail; requires validation |
| `OPEN QUESTION` | Cannot be resolved from available evidence and requires a decision |

### 2.2 Initial Priority

| Priority | Meaning |
|---|---|
| `M` | Must be covered to satisfy the source baseline or its necessary derived behavior |
| `S` | Recommended extension with material business or risk value |
| `C` | Optional portfolio or solution extension |
| `TBD` | Priority cannot be assigned until the underlying question is resolved |

Priorities are an initial analyst recommendation, not an approved Product Owner decision.

## 3. Functional Requirements

### 3.1 Client Registration and Client Management

| ID | Requirement | Class | Priority | Source | Primary verification |
|---|---|---:|---:|---|---|
| `FR-CRM-001` | The system shall allow an authorized Agent or Manager to create a new company registration. | `SOURCE` | M | Brief §1.2 Client and product management; §2.2 Agent and Manager roles | Functional test |
| `FR-CRM-002` | The system shall allow an authorized user to verify an already registered company. | `SOURCE` | M | Brief §1.2 Client and product management | Functional test; verification method remains open |
| `FR-CRM-003` | The system shall allow an authorized user to search registered companies. | `SOURCE` | M | Brief §1.2 Client and product management | Search tests with matching and no-result cases |
| `FR-CRM-004` | The system shall display a registration detail to a user authorized under the ownership rules. | `SOURCE` | M | Brief §1.2 Owner-only detail rule | Authorization test |
| `FR-CRM-005` | The system shall allow the registration owner to edit an existing registration. | `SOURCE` | M | Brief §1.2 Client editing and owner-only edit rule | Functional and authorization tests |
| `FR-CRM-006` | Before creating a registration, the system shall determine whether the company already exists using an agreed unique business identifier or matching rule and shall prevent a confirmed duplicate. | `DERIVED` | M | Brief §1.1 duplicate-client problem; §1.2 company verification | Duplicate and concurrency tests |
| `FR-CRM-007` | Each registration shall have exactly one current owner. | `SOURCE` | M | Brief §1.2 Ownership definition | Data-integrity and functional tests |

The source does not identify IČO as the approved duplicate key. IČO-based validation is retained as a proposed assumption in Section 5.

### 3.2 Client Product Management and Registration Lifecycle

| ID | Requirement | Class | Priority | Source | Primary verification |
|---|---|---:|---:|---|---|
| `FR-PROD-001` | The system shall allow an authorized Agent or Manager to record a product sold to a registered client and shall maintain the relationship between the client, registration, and product. | `DERIVED` | M | Brief §1.1 product forms; §1.2 product management; §2.2 Agent and Manager roles; reporting requirements | Functional and data-model tests |
| `FR-PROD-002` | The system shall allow an authorized user to deactivate a client product and record the effective deactivation date. | `SOURCE` | M | Brief §1.2 last-product deactivation rule | Functional and data-integrity tests |
| `FR-PROD-003` | Before a product is offered or sold, the system shall detect whether the client already has the same active product and prevent a confirmed duplicate. | `DERIVED` | M | Brief §1.1 duplicate-product problem | Validation and concurrency tests |
| `FR-LIFE-001` | The system shall automatically terminate a client registration one year after the client's last product is deactivated, provided the client has no active product. | `SOURCE` | M | Brief §1.2 Client and product management | Boundary and scheduled lifecycle tests |

`Terminated` is used here as a business outcome. The final status name, calculation method, time zone, and behavior after product reactivation remain open questions.

### 3.3 Registration Lists, Visibility, and Ownership

| ID | Requirement | Class | Priority | Source | Primary verification |
|---|---|---:|---:|---|---|
| `FR-RBAC-001` | The Agent's organizational-unit registration list shall include only registrations whose owners have the same Manager as the Agent and belong to the same sales point as the Agent. | `SOURCE` | M | Brief §1.2 Agent registration lists | Authorization decision-table tests |
| `FR-RBAC-002` | The system shall provide an Agent with a `My Registrations` list containing registrations currently owned by that Agent. | `SOURCE` | M | Brief §1.2 Agent registration lists | Functional and authorization tests |
| `FR-RBAC-003` | The system shall provide a Manager with a list of registrations owned by the Manager's direct reports. | `SOURCE` | M | Brief §1.2 Manager registration lists | Authorization decision-table tests |
| `FR-RBAC-004` | The system shall provide a Manager with an organizational-unit view of registrations associated with the Manager's direct reports. | `SOURCE` | M | Brief §1.2 Manager registration lists | Authorization tests; exact list semantics remain open |
| `FR-RBAC-005` | The system shall provide a Manager with a `My Registrations` list containing registrations currently owned by that Manager. | `SOURCE` | M | Brief §1.2 Manager registration lists | Functional and authorization tests |
| `FR-RBAC-006` | A registration detail shall be viewable only by its current owner. | `SOURCE` | M | Brief §1.2 Owner-only detail rule | Positive and negative authorization tests |
| `FR-RBAC-007` | A registration shall be editable only by its current owner. | `SOURCE` | M | Brief §1.2 Owner-only edit rule | Positive and negative authorization tests |
| `FR-RBAC-008` | When a registration is created, the creating worker shall become its owner unless ownership is assigned through an authorized reassignment process. | `SOURCE` | M | Brief §1.2 Ownership definition | Functional and audit-state tests |
| `FR-RBAC-009` | A Manager or an authorized deputy shall be able to change the owner of a registration within the permitted management scope. | `SOURCE` | M | Brief §1.2 Ownership reassignment | Authorization and functional tests |
| `FR-RBAC-010` | A Manager or deputy shall be allowed to initiate reassignment only for a registration currently owned by the Manager or by one of the Manager's direct reports. | `SOURCE` | M | Brief §1.2 Ownership reassignment scope | Negative authorization tests |

List visibility and detail access are intentionally modeled separately. The source permits broader registration lists while restricting registration detail and editing to the owner.

### 3.4 Registration Approval

| ID | Requirement | Class | Priority | Source | Primary verification |
|---|---|---:|---:|---|---|
| `FR-APR-001` | Each newly submitted registration shall require Manager approval before completion of the registration process. | `SOURCE` | M | Brief §1.2 New-registration approval | Workflow and authorization tests |

The source does not explicitly define rejection, rejection reasons, reminders, notifications, status names, or the approver-selection rule. These items are not part of the source baseline and appear in Section 5 as proposed assumptions.

### 3.5 Organizational Structure

| ID | Requirement | Class | Priority | Source | Primary verification |
|---|---|---:|---:|---|---|
| `FR-ORG-001` | The system shall represent `Region` and `Sales Point` as organizational-unit types. | `SOURCE` | M | Brief §2.1 Organizational structure | Data-model and functional tests |
| `FR-ORG-002` | The organizational structure shall represent four Regions, each containing one or more Sales Points. | `SOURCE` | M | Brief §2.1 Organizational structure | Cardinality and data-integrity tests |
| `FR-ORG-003` | Each distribution-network worker shall belong to exactly one Sales Point. | `SOURCE` | M | Brief §2.1 Organizational structure | Data-integrity tests |
| `FR-ORG-004` | Each distribution-network worker shall have exactly one Manager. | `SOURCE` | M | Brief §2.1 Organizational structure | Data-integrity tests |
| `FR-ORG-005` | Each Manager shall have one or more direct reports. | `SOURCE` | M | Brief §2.1 Organizational structure | Cardinality tests |
| `FR-ORG-006` | A Manager's direct reports may belong to different Sales Points but shall remain within one Region. | `SOURCE` | M | Brief §2.1 Organizational structure | Cross-unit authorization and integrity tests |
| `FR-ORG-007` | A Sales Point shall support zero or more assigned workers. | `SOURCE` | M | Brief §2.1 Organizational structure | Cardinality tests |

### 3.6 Users, Roles, Permissions, and Organizational Administration

| ID | Requirement | Class | Priority | Source | Primary verification |
|---|---|---:|---:|---|---|
| `FR-IAM-001` | The system shall support the business roles `Agent`, `Manager`, and `Administrator`. | `SOURCE` | M | Brief §2.2 Users and roles | Role-assignment tests |
| `FR-IAM-002` | During phase 1, IFCOM+ shall store organizational structure, worker, role, and authorization information in its own database without integration to an external identity and authorization repository. | `SOURCE` | M | Brief §1.2 Organizational and worker management | Architecture review and integration test |
| `FR-IAM-003` | The Administrator shall manually maintain organizational, worker, role, and permission data in IFCOM+. | `SOURCE` | M | Brief §1.2 Organizational and worker management | Functional and authorization tests |
| `FR-IAM-004` | Access to the organizational-unit and worker-management area shall be restricted to Managers and Administrators. | `SOURCE` | M | Brief §1.2 Organizational and worker management | Negative authorization tests |
| `FR-IAM-005` | The Administrator shall be able to create a worker. | `SOURCE` | M | Brief §1.2 Administrator activities | Functional and authorization tests |
| `FR-IAM-006` | The Administrator shall be able to edit a worker. | `SOURCE` | M | Brief §1.2 Administrator activities | Functional and authorization tests |
| `FR-IAM-007` | The Administrator shall be able to transfer a worker from one organizational unit to another. | `SOURCE` | M | Brief §1.2 Administrator activities | Functional, integrity, and authorization tests |
| `FR-IAM-008` | The Administrator shall be able to grant and revoke application permissions for a worker. | `SOURCE` | M | Brief §1.2 Administrator activities | Permission-effect and authorization tests |
| `FR-IAM-009` | The Administrator shall be able to create an organizational unit. | `SOURCE` | M | Brief §1.2 Administrator activities | Functional and authorization tests |
| `FR-IAM-010` | The Administrator shall be able to edit an organizational unit. | `SOURCE` | M | Brief §1.2 Administrator activities | Functional and authorization tests |
| `FR-IAM-011` | The Administrator shall be able to deactivate an organizational unit. | `SOURCE` | M | Brief §1.2 Administrator activities | Dependency, integrity, and authorization tests |

The brief allows Managers into this application area primarily to obtain reports, while the listed maintenance operations are assigned to the Administrator. Any additional Manager administration permission requires an explicit decision.

### 3.7 Management Reporting

| ID | Requirement | Class | Priority | Source | Primary verification |
|---|---|---:|---:|---|---|
| `FR-RPT-001` | The system shall provide a Manager with the number of registered companies for each direct report. | `SOURCE` | M | Brief §1.2 Report 1 | Aggregation and authorization tests |
| `FR-RPT-002` | The system shall provide a Manager with the number of registered companies for each organizational unit containing the Manager's direct reports. | `SOURCE` | M | Brief §1.2 Report 2 | Aggregation and authorization tests |
| `FR-RPT-003` | The system shall provide a Manager with the number of products sold by each direct report per month. | `SOURCE` | M | Brief §1.2 Report 3 | Aggregation, date-boundary, and authorization tests |
| `FR-RPT-004` | The system shall provide a Manager with the number of products sold by each relevant organizational unit per month. | `SOURCE` | M | Brief §1.2 Report 4 | Aggregation, date-boundary, and authorization tests |

## 4. Non-Functional Requirements and Technical Constraints

Technical choices explicitly mandated by the source are recorded as constraints rather than being misclassified as quality requirements.

| ID | Requirement or constraint | Class | Priority | Source | Primary verification |
|---|---|---:|---:|---|---|
| `CON-ARCH-001` | IFCOM+ shall be implemented as a web application. | `SOURCE` | M | Brief §1.3 Technical requirements | Architecture review |
| `CON-TECH-001` | IFCOM+ shall be implemented in Java according to J2EE standards. | `SOURCE` | M | Brief §1.3 Technical requirements | Architecture and build review |
| `CON-DEP-001` | IFCOM+ shall run in the application-server environment identified as `Boss` in the supplied brief. | `SOURCE` | M | Brief §1.3 Technical requirements | Deployment review; exact product remains open |
| `NFR-AVL-001` | IFCOM+ shall be available 24 hours per day, 365 days per year. | `SOURCE` | M | Brief §1.3 Technical requirements | Availability measurement after SLA clarification |
| `NFR-CAP-001` | The solution shall be sized for an expected workload of approximately 20 processed registrations per day. | `SOURCE` | M | Brief §1.3 Technical requirements | Capacity and workload test after peak profile clarification |
| `NFR-L10N-001` | IFCOM+ shall provide at least Czech and Slovak language variants. | `SOURCE` | M | Brief §1.3 Technical requirements | Localization and content tests |
| `NFR-DEV-001` | The user interface shall support desktop computers, laptops, and tablets. | `SOURCE` | M | Brief §3 Design requirements | Cross-device functional and visual tests |

`NFR-AVL-001` does not yet define an availability percentage, maintenance windows, measurement point, recovery targets, or exclusions. `NFR-CAP-001` supplies a business-volume input but not a response-time or concurrency target.

## 5. Proposed Requirements Requiring Validation

The following items originated in the candidate analysis or later portfolio design. They are valuable proposals, but they are not source-approved requirements.

| ID | Proposed requirement | Class | Recommended priority | Rationale / source of proposal |
|---|---|---:|---:|---|
| `A-CRM-001` | Use IČO as the primary company duplicate-check key, with an agreed normalization and matching rule. | `ASSUMPTION` | S | Candidate analysis; exact business key absent from brief |
| `A-INT-001` | Validate and enrich company data through the Czech ARES register, with timeout and unavailable-service handling. | `ASSUMPTION` | C | Portfolio integration proposal |
| `A-APR-001` | Allow an assigned Manager to reject a registration. | `ASSUMPTION` | S | Candidate analysis and BPMN extension |
| `A-APR-002` | Require a rejection reason when a registration is rejected. | `ASSUMPTION` | S | Candidate analysis and BPMN extension |
| `A-APR-003` | Notify the registration owner after approval or rejection. | `ASSUMPTION` | S | Candidate analysis and BPMN extension |
| `A-APR-004` | Send a non-interrupting reminder when Manager review has been pending for 48 hours. | `ASSUMPTION` | C | BPMN extension; no source SLA |
| `A-LIFE-001` | Implement the one-year termination rule through a scheduled background process. | `ASSUMPTION` | S | Candidate implementation proposal; business outcome is source-backed |
| `A-API-001` | Expose selected IFCOM+ capabilities through a REST API with explicit authorization and error contracts. | `ASSUMPTION` | C | Portfolio systems-analysis proposal |
| `A-SEC-001` | Protect browser-to-server communication with HTTPS using TLS 1.2 or higher. | `ASSUMPTION` | M | Candidate security proposal; minimum version requires architecture validation |
| `A-SEC-002` | Store passwords using an approved adaptive password-hashing algorithm and never in plaintext. | `ASSUMPTION` | M | Candidate security proposal; authentication design absent from brief |
| `A-SEC-003` | End authenticated sessions after a defined period of inactivity. | `ASSUMPTION` | M | Candidate tablet-risk proposal; timeout value is undecided |
| `A-AUD-001` | Record security-relevant access and change events in a protected audit trail. | `ASSUMPTION` | M | Candidate audit proposal; event scope and retention are undecided |
| `A-PERF-001` | Set a measurable response-time objective for client lists and details; the original candidate proposal was two seconds. | `ASSUMPTION` | S | Candidate analysis; no response target in source |
| `A-USAB-001` | Define measurable usability criteria for navigation, forms, errors, and onboarding rather than relying on `modern` or `intuitive`. | `ASSUMPTION` | S | Derived from the legacy design problem |

Recommended priority does not convert an assumption into an approved requirement. It indicates what should be discussed first in a real discovery or refinement workshop.

## 6. Open Questions

| ID | Question | Affected requirements | Decision owner |
|---|---|---|---|
| `OQ-001` | Which attribute or combination of attributes uniquely identifies a company: IČO, another identifier, or a matching rule? | `FR-CRM-002`, `FR-CRM-006`, `A-CRM-001` | Product Owner / Data Owner |
| `OQ-002` | What does `verify a registered company` mean operationally, and how is it different from search and duplicate prevention? | `FR-CRM-002`, `FR-CRM-003`, `FR-CRM-006` | Product Owner |
| `OQ-003` | Which Manager approves a new registration: the creator's Manager, the owner's Manager, or a Manager selected by organizational unit? | `FR-APR-001` | Product Owner |
| `OQ-004` | How can a Manager review a registration for approval when the source also states that only the owner may view registration detail? | `FR-APR-001`, `FR-RBAC-006` | Product Owner / Security Owner |
| `OQ-005` | May a Manager approve their own registration, or is segregation of duties required? | `FR-APR-001`, `FR-RBAC-005` | Risk Owner / Product Owner |
| `OQ-006` | Is rejection required? If yes, are a reason, notification, resubmission, and edit rules required? | `A-APR-001` to `A-APR-003` | Product Owner |
| `OQ-007` | What are the approved registration states and permitted transitions? | `FR-APR-001`, `FR-LIFE-001` | Product Owner |
| `OQ-008` | How is a Manager's deputy appointed, scoped, activated, and revoked? | `FR-RBAC-009`, `FR-RBAC-010` | Business Owner / Security Owner |
| `OQ-009` | To which workers may a Manager or deputy reassign a registration? | `FR-RBAC-009`, `FR-RBAC-010` | Product Owner |
| `OQ-010` | What is the exact population of the Manager's organizational-unit registration list, and is it filtered, grouped, or aggregated by Sales Point? | `FR-RBAC-004` | Product Owner |
| `OQ-011` | What uniquely identifies a product, what counts as `active`, and does the duplicate rule apply to offered, contracted, or sold products? | `FR-PROD-001` to `FR-PROD-003` | Product Owner / Product Data Owner |
| `OQ-012` | How is the one-year period calculated, which time zone applies, and what happens if a product is reactivated before termination? | `FR-LIFE-001` | Product Owner |
| `OQ-013` | What dependencies must be resolved before a worker transfer or organizational-unit deactivation is allowed? | `FR-IAM-007`, `FR-IAM-011` | Product Owner / Data Owner |
| `OQ-014` | Do Managers receive read-only access to the administration area, or may they perform any maintenance actions? | `FR-IAM-004` to `FR-IAM-011` | Product Owner / Security Owner |
| `OQ-015` | Which registration statuses count as `registered companies`, and which event and date define a `sold product` in each report? | `FR-RPT-001` to `FR-RPT-004` | Reporting Owner |
| `OQ-016` | What filters, export formats, refresh frequency, historical retention, and time zone are required for reports? | `FR-RPT-001` to `FR-RPT-004` | Reporting Owner |
| `OQ-017` | What authentication, account lifecycle, password, lockout, session, and privileged-access controls are required in phase 1? | `FR-IAM-002`, `A-SEC-001` to `A-SEC-003` | Security Owner |
| `OQ-018` | What availability percentage, maintenance windows, monitoring point, RTO, and RPO apply? | `NFR-AVL-001` | Service Owner |
| `OQ-019` | What response-time, concurrent-user, peak-volume, and batch-processing targets apply? | `NFR-CAP-001`, `A-PERF-001` | Product Owner / Architect |
| `OQ-020` | Which content must be localized, what is the fallback language, and who maintains translations? | `NFR-L10N-001` | Product Owner |
| `OQ-021` | Which browser versions, screen sizes, orientations, accessibility level, and touch interactions define device support? | `NFR-DEV-001`, `A-USAB-001` | UX Owner / Product Owner |
| `OQ-022` | Which personal data, retention rules, audit events, access reviews, and privacy controls are required? | `A-AUD-001`, `A-SEC-001` to `A-SEC-003` | Privacy and Security Owners |
| `OQ-023` | Does `Boss` identify a specific supported application-server product or a typographical/anonymization artifact? | `CON-DEP-001` | Solution Architect |

## 7. Scope Exclusions

The following items are not established by the source brief and remain outside the baseline unless approved as extensions:

- a working application implementation;
- production deployment and operational support;
- cloud architecture;
- legacy data migration;
- claims, pricing, underwriting, billing, and payment processing;
- ARES or other external business-data integrations;
- SSO or external identity-repository integration during phase 1;
- proof of GDPR, security, availability, performance, or audit compliance;
- final UI design and usability validation with real users.

## 8. Legacy ID Migration

This mapping preserves references from the old short BRD while moving to stable module-based IDs.

| Legacy ID | Replacement | Notes |
|---|---|---|
| `FR_1.01` | `FR-CRM-001` | Creation applies to authorized sales users; role detail is validated separately |
| `FR_1.02` | `FR-CRM-002`, `FR-CRM-006`, `A-CRM-001` | Duplicate prevention is derived; IČO is an assumption, not a source fact |
| `FR_1.03` | `FR-PROD-001`, `FR-PROD-002` | Product recording and deactivation separated |
| `FR_1.04` | `FR-PROD-003` | Derived from the AS-IS duplicate-product problem |
| `FR_1.05` | `FR-RBAC-001` | Agent list rule retained |
| `FR_1.06` | `FR-RBAC-003`, `FR-RBAC-004` | Manager views separated |
| `FR_1.07` | `FR-APR-001`, `A-APR-001`, `A-APR-002` | Approval is source-backed; rejection and reason are assumptions |
| `FR_1.08` | `FR-LIFE-001` | Corrected to one year after last-product deactivation |

## 9. Baseline Exit Criteria

This draft can be promoted from `Draft` to `Reviewed` when:

- every source statement is mapped to at least one catalog item or documented as context only;
- conflicts between owner-only access and Manager approval are resolved;
- duplicate keys, product identity, reporting definitions, and lifecycle states are agreed;
- assumptions selected for scope are promoted into requirements with decision records;
- rejected assumptions remain clearly out of scope;
- every approved `M` requirement has acceptance criteria or a linked test scenario;
- the BPMN, RBAC matrix, domain model, API, database constraints, and tests use the same IDs and business rules.
