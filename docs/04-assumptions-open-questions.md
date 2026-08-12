# IFCOM+ Assumptions and Open Questions

## 1. Document Control

| Field | Value |
|---|---|
| Document status | Draft decision log |
| Version | 0.1 |
| Author | Oleksii Kim |
| Evidence boundary | [Project Origin and Scope](00-project-origin-and-scope.md) |
| Requirements baseline | [IFCOM+ Requirements Catalog](01-requirements-catalog.md) |
| Acceptance baseline | [User Stories and BDD Acceptance Criteria](02-user-stories.md) |
| Policy baseline | [RBAC and Business Rules](03-rbac-and-business-rules.md) |

This document turns the assumptions and open questions identified during requirements analysis into a controlled decision backlog. It does not silently fill gaps in the recruitment brief and does not treat an analyst recommendation as stakeholder approval.

Current position:

- all 14 `A-*` items remain proposed and require validation;
- all 23 `OQ-*` items remain open;
- `OQ-003` to `OQ-005` are critical blockers because the approval policy cannot be implemented consistently without them;
- no stakeholder decision is recorded in this version.

## 2. Decision Governance

### 2.1 Status Model

| Item | Status | Meaning |
|---|---|---|
| Assumption | `PROPOSED` | Plausible solution or rule awaiting an authorized decision |
| Assumption | `VALIDATED` | Approved and ready to be promoted into the controlled requirements baseline |
| Assumption | `REJECTED` | Explicitly declined; dependent rules and stories must be removed or revised |
| Assumption | `SUPERSEDED` | Replaced by a later, more precise proposal or decision |
| Assumption | `DEFERRED` | Excluded from the current phase but retained for later consideration |
| Open question | `OPEN` | A decision is still required |
| Open question | `ANSWERED` | A decision record contains an approved answer |
| Open question | `DEFERRED` | The answer is not required for the current delivery scope |
| Open question | `CANCELLED` | The question no longer applies because the related scope was removed or changed |

An assumption becomes `VALIDATED`, or an open question becomes `ANSWERED`, only when a named decision owner approves a decision record. Workshop discussion, implementation preference, or inclusion in a diagram is not approval.

### 2.2 Decision Urgency

| Priority | Meaning | Required timing |
|---|---|---|
| `P0` | Current requirements or policies are contradictory or blocked | Resolve before approving the affected process, RBAC, or state model |
| `P1` | Detailed solution design, data rules, API contracts, or executable tests depend on the answer | Resolve before implementation-ready specifications are baselined |
| `P2` | The item refines reporting, UX, localization, or operational quality | Resolve before the affected feature or non-functional acceptance is signed off |

Priority expresses decision urgency, not business value.

### 2.3 Required Evidence

A decision should cite at least one relevant evidence type:

- business-owner confirmation;
- security, privacy, risk, or service-owner policy;
- data profiling or master-data analysis;
- current-process observation or stakeholder workshop output;
- architecture or integration constraint;
- prototype, usability test, performance test, or operational measurement.

## 3. Immediate Decision Queue

| Order | Decision bundle | Questions | Why it comes first | Unlocks |
|---:|---|---|---|---|
| 1 | Approval authority and access | `OQ-003` to `OQ-005` | The source requires Manager approval but also restricts detail access to the owner; self-approval is unresolved | Approval BPMN, RBAC policy, approval API, acceptance tests |
| 2 | Approval outcome and state model | `OQ-006`, `OQ-007` | Rejection, notification, resubmission, and lifecycle states are not source-defined | State diagram, workflow paths, notification scope, persistence model |
| 3 | Company identity and verification | `OQ-001`, `OQ-002` | Duplicate prevention is mandatory, but the company key and meaning of verification are unknown | Logical data model, database constraints, search behavior, duplicate tests |
| 4 | Delegation, reassignment, and list scope | `OQ-008` to `OQ-010` | Several authorization rules are provisional until relationship and target scopes are defined | Complete RBAC matrix, reassignment flow, list queries, negative tests |
| 5 | Product identity and termination | `OQ-011`, `OQ-012` | Duplicate-product enforcement and the one-year boundary need exact semantics | Product model, lifecycle automation, boundary tests |

The remaining questions should then be resolved by domain: administration, reporting, security and privacy, service quality, localization, device support, and deployment platform.

## 4. Assumption Register

All entries in this section have status `PROPOSED`. The recommendation column is an analyst position for discussion, not an approved requirement.

### 4.1 Client Data and Integration

| ID | Proposed assumption | Catalog priority | Risk if wrong | Validation and owner | Analyst recommendation | Related questions / artifacts |
|---|---|:---:|---|---|---|---|
| `A-CRM-001` | Use IČO as the primary company duplicate-check key, with an agreed normalization and matching rule. | S | Legitimate companies may be blocked, or duplicates may still be created. | Profile company data and edge cases; confirm with Product Owner and Data Owner. | Validate IČO as the preferred key, but allow an approved composite or exception rule where IČO is absent or invalid. | `OQ-001`, `OQ-002`; logical model, SQL constraints, API validation, duplicate tests |
| `A-INT-001` | Validate and enrich company data through the Czech ARES register, including timeout and unavailable-service handling. | C | The registration flow may depend on an unavailable, unsuitable, or legally restricted external service. | Confirm business value, data rights, service contract, rate limits, latency, and fallback with Product, Data, Privacy, and Architecture owners. | Keep as an optional integration; do not make ARES availability a prerequisite for the source baseline without an explicit decision. | `OQ-001`, `OQ-002`, `OQ-019`, `OQ-022`; integration contract, BPMN exception path |

### 4.2 Approval Workflow

| ID | Proposed assumption | Catalog priority | Risk if wrong | Validation and owner | Analyst recommendation | Related questions / artifacts |
|---|---|:---:|---|---|---|---|
| `A-APR-001` | Allow the assigned Manager to reject a registration. | S | The workflow may invent an unauthorized outcome or provide no valid response to an unacceptable submission. | Product Owner defines permitted decisions and business outcomes. | Approve only together with the registration state model and resubmission rule. | `OQ-003`, `OQ-006`, `OQ-007`; BPMN, state model, `POL-APR-005` |
| `A-APR-002` | Require a rejection reason. | S | An unnecessary field may burden users, or an absent reason may make correction and audit impossible. | Product, Risk, and Compliance owners define reason type, visibility, and retention. | Require a reason if rejection is approved; prefer a controlled category plus optional comment where appropriate. | `OQ-006`, `OQ-007`, `OQ-022`; UI, API, audit model, `POL-APR-006` |
| `A-APR-003` | Notify the registration owner after approval or rejection. | S | Owners may not know that action is required, while excessive notification may expose data or create noise. | Product and Security owners confirm channel, recipients, content, retry, and privacy rules. | Approve an in-application notification as the minimum after the outcome and recipient model are defined. | `OQ-006`, `OQ-007`, `OQ-017`, `OQ-022`; notification contract, acceptance tests |
| `A-APR-004` | Send a non-interrupting reminder after Manager review has been pending for 48 hours. | C | An arbitrary timer may misrepresent the SLA, create alert fatigue, or conflict with working calendars. | Measure desired turnaround and confirm escalation policy with Product and Service owners. | Defer the fixed 48-hour value until an approval SLA and business-calendar rule exist. | `OQ-003`, `OQ-007`, `OQ-018`, `OQ-019`; BPMN timer, scheduler, monitoring |

### 4.3 Lifecycle and API

| ID | Proposed assumption | Catalog priority | Risk if wrong | Validation and owner | Analyst recommendation | Related questions / artifacts |
|---|---|:---:|---|---|---|---|
| `A-LIFE-001` | Implement the one-year termination rule through a scheduled background process. | S | Termination may occur late, twice, or against stale lifecycle data. | Architect and Product Owner confirm trigger model, frequency, idempotency, retry, time zone, and operational ownership. | Use a recoverable, idempotent background mechanism after the one-year calculation and reactivation rules are approved. | `OQ-007`, `OQ-012`, `OQ-018`, `OQ-019`; scheduler design, state model, lifecycle tests |
| `A-API-001` | Expose selected IFCOM+ capabilities through a REST API with explicit authorization and error contracts. | C | The portfolio may design an unused interface or expose protected data without a confirmed consumer and threat model. | Identify consumers and use cases; validate with Product, Architecture, and Security owners. | Retain as a portfolio extension, but specify only endpoints that support approved use cases and enforce the same object-level policy as the UI. | `OQ-017`, `OQ-019`, `OQ-022`, `OQ-023`; OpenAPI, authorization tests |

### 4.4 Security, Audit, Performance, and Usability

| ID | Proposed assumption | Catalog priority | Risk if wrong | Validation and owner | Analyst recommendation | Related questions / artifacts |
|---|---|:---:|---|---|---|---|
| `A-SEC-001` | Protect browser-to-server communication with HTTPS using TLS 1.2 or higher. | M | Sensitive data may be exposed, or a fixed version may become inconsistent with the target platform policy. | Security Owner and Architect confirm the current approved transport-security standard. | Validate the security objective; reference the maintained organizational standard instead of freezing an obsolete cipher or protocol list in business requirements. | `OQ-017`, `OQ-022`, `OQ-023`; architecture, deployment, security tests |
| `A-SEC-002` | Store passwords using an approved adaptive password-hashing algorithm and never in plaintext. | M | Compromised credentials may be recoverable; alternatively, local passwords may not exist under the chosen authentication model. | Security Owner confirms identity provider, credential ownership, hashing standard, reset, and migration rules. | Validate if IFCOM+ stores local credentials; otherwise replace with the controls required by the approved identity provider. | `OQ-017`, `OQ-022`; IAM design, data model, security tests |
| `A-SEC-003` | End authenticated sessions after a defined period of inactivity. | M | Long sessions increase unauthorized-access risk; a short timeout harms field and tablet use. | Security, Product, and UX owners perform a risk-based session review. | Validate the control and decide normal, privileged, and absolute session limits separately. | `OQ-017`, `OQ-021`, `OQ-022`; session design, UX behavior, security tests |
| `A-AUD-001` | Record security-relevant access and change events in a protected audit trail. | M | Incidents and sensitive changes may be untraceable, or excessive logs may violate privacy and retention rules. | Security and Privacy owners define events, fields, access, integrity, retention, monitoring, and deletion. | Validate the objective; start with authorization changes, approval, reassignment, sensitive denials, and automated termination outcomes. | `OQ-017`, `OQ-022`; audit schema, retention policy, test scenarios |
| `A-PERF-001` | Set a measurable response-time objective for client lists and details; the candidate proposal was two seconds. | S | An arbitrary target may be infeasible or too weak and may not reflect realistic load. | Establish workloads and percentile-based targets with Product, Architecture, and Service owners. | Validate the need for measurable targets, but treat two seconds as a test hypothesis until load and percentile are defined. | `OQ-018`, `OQ-019`; performance budget and test plan |
| `A-USAB-001` | Define measurable usability criteria for navigation, forms, errors, and onboarding. | S | The redesign may remain subjective and impossible to accept objectively. | Product and UX owners define representative tasks, users, devices, accessibility, and success measures. | Validate and express criteria through task completion, error recovery, accessibility, and usability testing. | `OQ-020`, `OQ-021`; UI specification, prototype, usability tests |

## 5. Open-Question Register

### 5.1 Company, Approval, and Lifecycle

| ID | Priority | Decision required | Minimum decision output | Owner | Blocks |
|---|:---:|---|---|---|---|
| `OQ-001` | P1 | Which attribute or combination uniquely identifies a company: IČO, another identifier, or a matching rule? | Canonical key, normalization, missing/invalid-value behavior, exception handling, and concurrency rule | Product Owner / Data Owner | `FR-CRM-002`, `FR-CRM-006`, logical model, database constraints, duplicate tests |
| `OQ-002` | P1 | What does `verify a registered company` mean, and how does it differ from search and duplicate prevention? | Actor, trigger, inputs, result states, visible data, and downstream action | Product Owner | `FR-CRM-002`, `FR-CRM-003`, `FR-CRM-006`, BPMN, API behavior |
| `OQ-003` | P0 | Which Manager approves: the creator's Manager, the owner's Manager, or a Manager selected by organizational unit? | Deterministic approver-selection rule, reassignment behavior, and unavailable-approver fallback | Product Owner | `FR-APR-001`, `POL-APR-001`, approval BPMN, API, tests |
| `OQ-004` | P0 | How may a Manager review a registration when only the owner may view registration detail? | Approved review view, permitted fields, purpose limitation, action scope, and audit rule | Product Owner / Security Owner | `FR-APR-001`, `FR-RBAC-006`, `POL-OBJ-002`, `POL-APR-001` |
| `OQ-005` | P0 | May a Manager approve their own registration, or is segregation of duties required? | Allow/deny rule, alternate approver, escalation, and audit behavior | Risk Owner / Product Owner | `POL-APR-003`, approval completion and negative tests |
| `OQ-006` | P1 | Is rejection required and, if so, what are the reason, notification, resubmission, and edit rules? | Decision options, required data, next state, responsible actor, edit scope, and notification | Product Owner | `A-APR-001` to `A-APR-003`, approval BPMN, state model |
| `OQ-007` | P1 | What are the approved registration states and permitted transitions? | Named states, transition matrix, actors, guards, terminal states, correction path, and lifecycle interaction | Product Owner | `FR-APR-001`, `FR-LIFE-001`, data model, API, test suite |
| `OQ-011` | P1 | What uniquely identifies a product, what counts as active, and where does duplicate prevention apply? | Product key, offered/contracted/sold states, active rule, resale/reactivation, and concurrency behavior | Product Owner / Product Data Owner | `FR-PROD-001` to `FR-PROD-003`, product model, constraints, tests |
| `OQ-012` | P1 | How is one year calculated, which time zone applies, and what happens after product reactivation? | Calendar rule, anchor timestamp, time zone, evaluation frequency, reactivation/cancellation, and idempotency | Product Owner | `FR-LIFE-001`, `BR-LIFE-002` to `BR-LIFE-005`, scheduler and boundary tests |

### 5.2 Authorization and Administration

| ID | Priority | Decision required | Minimum decision output | Owner | Blocks |
|---|:---:|---|---|---|---|
| `OQ-008` | P1 | How is a Manager's deputy appointed, scoped, activated, and revoked? | Delegator, eligible deputy, capabilities, record scope, start/end, revocation, conflict, and audit rules | Business Owner / Security Owner | `VALID_DEPUTY`, `POL-OWN-002`, delegation data model and tests |
| `OQ-009` | P1 | To which workers may a Manager or deputy reassign a registration? | Target role, account state, Manager/Sales Point/Region constraints, and invalid-target behavior | Product Owner | `POL-OWN-001`, `POL-OWN-005`, reassignment UI/API/tests |
| `OQ-010` | P1 | What is the exact population of the Manager's organizational-unit list, and is it filtered, grouped, or aggregated by Sales Point? | Included owners/units, summary fields, grouping, access boundary, and empty-state behavior | Product Owner | `FR-RBAC-004`, `POL-LIST-007`, queries and authorization tests |
| `OQ-013` | P1 | Which dependencies must be resolved before worker transfer or organizational-unit deactivation? | Blocking dependency matrix, reassignment/migration actions, transaction boundary, history, and rollback | Product Owner / Data Owner | `FR-IAM-007`, `FR-IAM-011`, `BR-ORG-010`, admin workflows |
| `OQ-014` | P1 | Are Managers read-only in the administration area, or may they perform maintenance actions? | Capability matrix by role, object scope, approval needs, and audit controls | Product Owner / Security Owner | `FR-IAM-004` to `FR-IAM-011`, `POL-ADM-004`, RBAC tests |
| `OQ-017` | P1 | Which authentication, account lifecycle, password, lockout, session, and privileged-access controls apply in phase 1? | Identity source, account states, authentication flows, recovery, lockout, session limits, and privileged controls | Security Owner | `FR-IAM-002`, `A-SEC-001` to `A-SEC-003`, IAM architecture |
| `OQ-022` | P1 | Which personal data, retention rules, audit events, access reviews, and privacy controls are required? | Data inventory and classification, purposes, retention, access/review rules, audit events, and privacy obligations | Privacy Owner / Security Owner | Data model, API fields, logging, audit, retention, test evidence |

### 5.3 Reporting, Service Quality, UX, and Platform

| ID | Priority | Decision required | Minimum decision output | Owner | Blocks |
|---|:---:|---|---|---|---|
| `OQ-015` | P1 | Which registration statuses count as registered companies, and which event/date defines a sold product? | Metric dictionary, included states/events, organizational attribution, corrections, and historical rule | Reporting Owner | `FR-RPT-001` to `FR-RPT-004`, report queries and acceptance tests |
| `OQ-016` | P2 | Which filters, exports, refresh frequency, history, and time zone are required for reports? | Filter set, export formats, freshness, retention window, time zone, pagination, and empty-state behavior | Reporting Owner | Report UX, API/query contract, operational acceptance |
| `OQ-018` | P1 | What availability percentage, maintenance windows, monitoring point, RTO, and RPO apply? | Service-level target, exclusions, measurement window/point, support hours, RTO, RPO, and reporting | Service Owner | `NFR-AVL-001`, deployment topology, monitoring, recovery tests |
| `OQ-019` | P1 | What response-time, concurrent-user, peak-volume, and batch-processing targets apply? | Critical transactions, workload model, percentiles, concurrency, batch windows, dataset size, and test environment | Product Owner / Architect | `NFR-CAP-001`, `A-PERF-001`, capacity design and performance tests |
| `OQ-020` | P2 | Which content must be localized, what is the fallback language, and who maintains translations? | Localized content inventory, Czech/Slovak locale rules, fallback, formatting, ownership, and release process | Product Owner | `NFR-L10N-001`, content model, localization tests |
| `OQ-021` | P2 | Which browsers, screen sizes, orientations, accessibility level, and touch interactions define device support? | Support matrix, responsive breakpoints, orientation/touch behavior, accessibility target, and test devices | UX Owner / Product Owner | `NFR-DEV-001`, `A-USAB-001`, UI design and compatibility tests |
| `OQ-023` | P1 | Does `Boss` identify a supported application-server product or a typographical/anonymization artifact? | Exact product and version, or explicit replacement of the ambiguous source term with an approved deployment constraint | Solution Architect | `CON-DEP-001`, architecture, build and deployment documentation |

## 6. Assumption-to-Question Dependencies

| Assumption | Must be resolved with | Dependency rule |
|---|---|---|
| `A-CRM-001` | `OQ-001`, `OQ-002` | Do not implement an IČO constraint until identity and verification semantics are approved. |
| `A-INT-001` | `OQ-001`, `OQ-002`, `OQ-019`, `OQ-022` | Do not make ARES mandatory until data purpose, performance, privacy, and fallback behavior are approved. |
| `A-APR-001` | `OQ-003`, `OQ-006`, `OQ-007` | Rejection requires an authorized approver and an approved outcome/state transition. |
| `A-APR-002` | `OQ-006`, `OQ-007`, `OQ-022` | Reason capture depends on rejection, state, visibility, and retention decisions. |
| `A-APR-003` | `OQ-006`, `OQ-007`, `OQ-017`, `OQ-022` | Notification depends on outcome, recipient, channel, authentication, and privacy rules. |
| `A-APR-004` | `OQ-003`, `OQ-007`, `OQ-018`, `OQ-019` | A reminder requires an assigned approver, pending state, SLA, scheduler, and workload rule. |
| `A-LIFE-001` | `OQ-007`, `OQ-012`, `OQ-018`, `OQ-019` | Automation requires approved states, time calculation, recovery, and processing targets. |
| `A-API-001` | `OQ-017`, `OQ-019`, `OQ-022`, `OQ-023` | API scope and contracts require identity, performance, data-protection, and platform decisions. |
| `A-SEC-001` | `OQ-017`, `OQ-022`, `OQ-023` | Transport controls must align with the identity model, data risk, and deployment platform. |
| `A-SEC-002` | `OQ-017`, `OQ-022` | Password storage applies only after credential ownership and retention are known. |
| `A-SEC-003` | `OQ-017`, `OQ-021`, `OQ-022` | Timeout values require identity, device-use, UX, and risk decisions. |
| `A-AUD-001` | `OQ-017`, `OQ-022` | Audit events, access, integrity, and retention depend on security and privacy policy. |
| `A-PERF-001` | `OQ-018`, `OQ-019` | The response objective requires a workload, percentile, environment, and service context. |
| `A-USAB-001` | `OQ-020`, `OQ-021` | Measurable usability criteria require locale, device, accessibility, and representative-task scope. |

## 7. Decision Workshop Plan

| Workshop | Required participants | Inputs | Expected outputs |
|---|---|---|---|
| Approval and registration lifecycle | Product Owner, operational Manager, Agent representative, Risk Owner, Security Owner | Source brief, approval user story, owner-only policy, proposed BPMN | Decisions for `OQ-003` to `OQ-007`; approved state and access model |
| Master data and product lifecycle | Product Owner, Data Owner, Product Data Owner, Architect | Sample company/product data, duplicate cases, lifecycle scenarios | Decisions for `OQ-001`, `OQ-002`, `OQ-011`, `OQ-012` |
| Authorization and administration | Product Owner, Security Owner, Business Owner, Administrator representative | RBAC matrix, organization examples, reassignment cases | Decisions for `OQ-008` to `OQ-010`, `OQ-013`, `OQ-014`, `OQ-017` |
| Reporting and information governance | Reporting Owner, Product Owner, Privacy Owner, Security Owner, Data Owner | Report examples, metric definitions, data inventory | Decisions for `OQ-015`, `OQ-016`, `OQ-022` |
| Non-functional and platform scope | Service Owner, Architect, UX Owner, Product Owner, Security Owner | Workload estimates, supported devices, localization needs, platform constraints | Decisions for `OQ-018` to `OQ-021`, `OQ-023` |

Each workshop should produce decision records, not only meeting notes.

## 8. Decision Record Template

Use one record per independently reversible decision.

```markdown
### DEC-001 — <Decision title>

| Field | Value |
|---|---|
| Status | Proposed / Approved / Rejected / Superseded |
| Decision date | YYYY-MM-DD |
| Decision owner | <Role and name> |
| Resolves | OQ-___ and/or A-___ |
| Effective scope | Phase / process / system boundary |
| Evidence | Workshop, policy, data analysis, prototype, test, or architecture constraint |

**Decision**

<One unambiguous rule or selected option.>

**Rationale**

<Why this option was selected and which alternatives were rejected.>

**Consequences**

- <Required requirement, process, data, authorization, API, or test change>
- <Known risk or follow-up action>

**Affected artifacts**

- `docs/01-requirements-catalog.md`
- `docs/02-user-stories.md`
- `docs/03-rbac-and-business-rules.md`
- <other affected artifact>
```

No `DEC-*` record is included in this version because no authorized stakeholder decision has been supplied.

## 9. Change-Control Rules

When a decision is approved:

1. Change the related `A-*` or `OQ-*` status in this document and add the `DEC-*` reference.
2. Promote an accepted assumption into a new or revised requirement in `01-requirements-catalog.md`; do not leave it classified as `ASSUMPTION`.
3. Update acceptance criteria in `02-user-stories.md` where observable behavior changes.
4. Replace `PROVISIONAL`, `ASSUMPTION`, or `BLOCKED` rules in `03-rbac-and-business-rules.md` with the appropriate controlled status.
5. Update BPMN, state, data, API, reporting, and architecture artifacts affected by the decision.
6. Add positive, negative, boundary, concurrency, and authorization coverage to the future test catalog.
7. Update the future traceability matrix so the decision, requirement, story, rule, design element, and test remain connected.

If an assumption is rejected, remove or revise every dependent rule and story. Do not retain rejected behavior merely because it already appears in a draft diagram or portfolio extension.

## 10. Exit Criteria

This decision log may move from `Draft` to `Reviewed` when:

- all `P0` questions have approved decision records;
- every accepted `A-*` proposal is promoted into the requirements baseline and every rejected proposal is removed from active behavior;
- each `OQ-*` has an owner, status, and either an approved answer or an explicit deferral rationale;
- approval, ownership, deputy, list, reporting, product, lifecycle, administration, and security rules are internally consistent;
- every decision identifies affected artifacts and has been propagated to them;
- no BPMN, RBAC, API, data, or test artifact presents an unresolved assumption as a source-backed fact.
