# IFCOM+ User Stories and BDD Acceptance Criteria

## 1. Document Control

| Field | Value |
|---|---|
| Document status | Draft for stakeholder refinement |
| Version | 0.1 |
| Author | Oleksii Kim |
| Requirements baseline | [IFCOM+ Requirements Catalog](01-requirements-catalog.md) |
| Evidence boundary | [Project Origin and Scope](00-project-origin-and-scope.md) |

This document converts eight priority business capabilities into user stories and executable-style BDD acceptance criteria. It does not resolve gaps that the source brief leaves open. Where a scenario depends on an unapproved decision, the dependency is shown explicitly through an `OQ-*` or `A-*` reference.

The stories are portfolio analysis artifacts, not stakeholder-approved backlog items.

## 2. Conventions

### 2.1 Scenario Tags

| Tag | Meaning |
|---|---|
| `@baseline` | Tests a `SOURCE` or `DERIVED` requirement from the catalog |
| `@authorization` | Tests role, ownership, or organizational-scope enforcement |
| `@negative` | Tests a rejected or invalid action |
| `@boundary` | Tests a date, lifecycle, or scope boundary |
| `@concurrency` | Tests simultaneous actions that could violate a business rule |
| `@assumption` | Tests a proposed extension that is not part of the source baseline |

### 2.2 Placeholders

Angle-bracket placeholders such as `<company key>` or `<reporting month>` represent values whose exact format or business definition must be agreed during refinement. They must be replaced with concrete examples before implementation testing.

### 2.3 Readiness Rule

A story is not `Ready for Development` while a linked open question changes its actor, authorization rule, business outcome, calculation, or validation key. The scenarios below make the intended behavior reviewable; they do not conceal missing decisions.

## 3. Story Inventory

| Story ID | Title | Primary actor | Linked requirements | Readiness blockers |
|---|---|---|---|---|
| `US-CRM-001` | Create a company registration without duplicates | Authorized sales user | `FR-CRM-001`, `FR-CRM-002`, `FR-CRM-006`, `FR-CRM-007`, `FR-RBAC-008` | `OQ-001`, `OQ-002` |
| `US-CRM-002` | Find and maintain accessible registrations | Agent or Manager | `FR-CRM-003` to `FR-CRM-005`, `FR-RBAC-001` to `FR-RBAC-007` | `OQ-010` |
| `US-APR-001` | Review and approve a new registration | Manager | `FR-APR-001` | `OQ-003` to `OQ-007` |
| `US-RBAC-001` | Reassign registration ownership | Manager or authorized deputy | `FR-RBAC-007`, `FR-RBAC-009`, `FR-RBAC-010` | `OQ-008`, `OQ-009` |
| `US-PROD-001` | Record and deactivate client products without duplicates | Registration owner | `FR-PROD-001` to `FR-PROD-003` | `OQ-011` |
| `US-LIFE-001` | Terminate an inactive client registration | Distribution-network Manager | `FR-LIFE-001` | `OQ-007`, `OQ-012` |
| `US-IAM-001` | Maintain workers, permissions, and organizational units | Administrator | `FR-ORG-001` to `FR-ORG-007`, `FR-IAM-001` to `FR-IAM-011` | `OQ-013`, `OQ-014`, `OQ-017` |
| `US-RPT-001` | Review management performance reports | Manager | `FR-RPT-001` to `FR-RPT-004` | `OQ-015`, `OQ-016` |

## 4. User Stories and Acceptance Criteria

### US-CRM-001 - Create a Company Registration Without Duplicates

**User story**

> As an authorized sales user, I want the system to determine whether a company already exists before creating a registration, so that I avoid duplicate client records and the manual correction they cause.

**Business rules**

- An authorized Agent or Manager may create a registration.
- A newly created registration has exactly one owner: the creating worker.
- A confirmed duplicate must not create another registration.
- The approved company matching key and the meaning of `verify` must be resolved through `OQ-001` and `OQ-002`.
- IČO may be adopted through `A-CRM-001`, but it is not treated as source-approved in the baseline scenarios.

**Acceptance criteria**

```gherkin
@baseline
Scenario: Create a registration when no matching company exists
  Given an Agent or Manager has permission to create registrations
  And the approved company matching rule is configured
  And no existing registration matches the submitted company data
  When the user submits valid registration data
  Then the system creates exactly one registration
  And sets the creating user as the current owner
  And routes the new registration into the Manager approval process

@baseline @negative
Scenario: Prevent creation of a confirmed duplicate company
  Given an Agent or Manager has permission to create registrations
  And an existing registration matches the submitted company data under the approved matching rule
  When the user submits the new registration
  Then the system does not create another registration
  And informs the user that the company already exists
  And does not disclose another owner's identity unless the user is authorized to see it

@baseline @concurrency
Scenario: Prevent duplicate records created by simultaneous requests
  Given no registration exists for the same approved company key
  When two authorized users submit matching company registrations concurrently
  Then the system persists only one registration for that company key
  And returns a duplicate or conflict outcome for the other request

@authorization @negative
Scenario: Reject a create request from a user without permission
  Given a signed-in user does not have permission to create registrations
  When the user submits company registration data
  Then the system denies the action
  And creates no registration
```

**Not decided in this story**

- the unique company key and normalization rules;
- whether verification is internal, external, or both;
- required registration fields;
- ARES integration and IČO checksum validation.

---

### US-CRM-002 - Find and Maintain Accessible Registrations

**User story**

> As an Agent or Manager, I want registration lists and search results that reflect my organizational scope, while only registrations I own can be opened and edited, so that I can find relevant work without gaining unauthorized access to client details.

**Business rules**

- List visibility does not grant detail or edit permission.
- An Agent's organizational list is restricted by both Sales Point and Manager.
- Agents and Managers have separate `My Registrations` views.
- Managers have direct-report and organizational-unit views.
- Only the current owner may open and edit registration detail under the source baseline.
- The exact meaning of the Manager's organizational-unit view remains unresolved in `OQ-010`.

**Acceptance criteria**

```gherkin
@baseline @authorization
Scenario: Show the Agent's organizational registration list
  Given an Agent belongs to Sales Point A and reports to Manager M1
  When the Agent opens the organizational registration list
  Then the list includes registrations owned by workers who belong to Sales Point A and report to Manager M1
  And excludes registrations owned by workers in another Sales Point
  And excludes registrations owned by workers reporting to another Manager

@baseline @authorization
Scenario: Show registrations currently owned by the signed-in user
  Given an Agent or Manager owns one or more registrations
  When the user opens My Registrations
  Then the system shows the registrations currently owned by that user
  And does not include registrations the user no longer owns

@baseline @authorization
Scenario: Show a Manager's direct-report registration list
  Given a Manager has direct reports in the distribution network
  When the Manager opens the direct-report registration list
  Then the system shows registrations owned by those direct reports
  And excludes registrations owned by workers outside the Manager's reporting scope

@baseline
Scenario: Search for a registration within the user's list visibility scope
  Given an Agent or Manager has access to a registration summary under an applicable list rule
  When the user searches using supported search criteria matching that registration
  Then the registration summary is included in the results
  And the search result does not grant additional detail or edit permission

@baseline @authorization @negative
Scenario: Prevent a non-owner from opening or editing a listed registration
  Given a registration is visible in a user's list
  And the user is not the current owner
  When the user attempts to open the registration detail or submit an update
  Then the system denies the detail or update action
  And does not return protected registration data
  And leaves the registration unchanged

@baseline
Scenario: Allow the owner to open and update a registration
  Given the signed-in user is the current registration owner
  When the user opens the registration and submits valid changes
  Then the system displays the registration detail
  And saves the accepted changes
  And keeps the same current owner unless an authorized reassignment occurs

@baseline @negative
Scenario: Return no result without disclosing an inaccessible registration
  Given a registration matches the submitted search criteria
  And the registration is outside every list scope available to the signed-in user
  When the user performs the search
  Then the inaccessible registration is not returned
  And the response does not reveal whether that registration exists
```

**Pending refinement**

The Manager organizational-unit scenario must be added after `OQ-010` defines the exact record population and grouping rule.

---

### US-APR-001 - Review and Approve a New Registration

**User story**

> As a Manager, I want to review and approve a newly submitted registration, so that the registration completes the required management-control step.

**Business rules**

- Every newly submitted registration requires Manager approval.
- The approving Manager and approval-access exception are not defined in the source.
- Approval must not be modeled as implementation-ready until `OQ-003` and `OQ-004` are resolved.
- Self-approval and lifecycle states remain unresolved through `OQ-005` and `OQ-007`.
- Rejection, rejection reason, notification, and reminder behavior are assumptions, not baseline requirements.

**Provisional baseline acceptance criteria**

These scenarios become executable only after the approver-selection and approval-access rules are approved.

```gherkin
@baseline @authorization
Scenario: Approve a registration as the authorized approver
  Given a newly submitted registration requires approval
  And the Manager is the authorized approver under the approved approval rule
  And the approved access exception allows the Manager to review the registration
  When the Manager records an approval decision
  Then the system records the registration as approved according to the agreed lifecycle model
  And records the decision actor and decision time

@authorization @negative
Scenario: Prevent approval by a Manager outside the permitted scope
  Given a newly submitted registration requires approval
  And the signed-in Manager is not the authorized approver under the approved approval rule
  When the Manager attempts to approve the registration
  Then the system denies the action
  And does not change the approval outcome

@baseline @negative
Scenario: Prevent a repeated decision on a completed approval step
  Given an approval decision has already been recorded for the registration
  When an authorized Manager submits another approval decision for the same approval step
  Then the system rejects the repeated decision
  And preserves the original decision outcome
```

**Assumption-backed extension scenarios**

The following scenarios are intentionally tagged `@assumption` and require approval of `A-APR-001` to `A-APR-004`.

```gherkin
@assumption @negative
Scenario: Require a reason when the Manager rejects a registration
  Given the Manager is authorized to decide the registration
  When the Manager attempts to reject it without a reason
  Then the system does not complete the rejection
  And asks the Manager to provide a rejection reason

@assumption
Scenario: Reject a registration and notify its owner
  Given the Manager is authorized to decide the registration
  When the Manager rejects it with a reason
  Then the system records the rejection according to the agreed lifecycle model
  And stores the rejection reason
  And notifies the current registration owner of the outcome

@assumption @boundary
Scenario: Remind the Manager without interrupting the approval task
  Given the registration has awaited a Manager decision for 48 hours
  And no approval or rejection has been recorded
  When the reminder condition is evaluated
  Then the system sends a reminder to the responsible Manager
  And keeps the approval task active
  And does not change the registration outcome
```

**No acceptance decision is made here for:**

- Manager self-approval;
- which Manager is responsible;
- whether a non-owner Manager may see full registration detail;
- rejection, resubmission, cancellation, or escalation;
- final status names and transitions.

---

### US-RBAC-001 - Reassign Registration Ownership

**User story**

> As a Manager or authorized deputy, I want to reassign a registration within my permitted management scope, so that client ownership can be maintained when responsibilities change.

**Business rules**

- Reassignment may be initiated by a Manager or an authorized deputy.
- The registration must currently be owned by the Manager or by one of the Manager's direct reports.
- The new owner must be eligible under the rule to be agreed through `OQ-009`.
- After reassignment, owner-only detail and edit rights follow the new owner.
- Deputy appointment and scope must be resolved through `OQ-008`.

**Acceptance criteria**

```gherkin
@baseline @authorization
Scenario: Manager reassigns a registration owned by a direct report
  Given a registration is owned by one of the Manager's direct reports
  And the selected new owner is eligible under the approved reassignment rule
  When the Manager confirms the reassignment
  Then the system sets the selected worker as the only current owner
  And grants the new owner owner-level detail and edit access
  And removes owner-level detail and edit access from the previous owner

@baseline @authorization
Scenario: Authorized deputy reassigns a registration within delegated scope
  Given an active deputy assignment authorizes the user to act for a Manager
  And the registration is owned by that Manager or one of that Manager's direct reports
  And the selected new owner is eligible under the approved reassignment rule
  When the deputy confirms the reassignment
  Then the system sets the selected worker as the only current owner

@authorization @negative
Scenario: Prevent reassignment outside the Manager's scope
  Given the registration is not owned by the Manager or any of the Manager's direct reports
  When the Manager attempts to change its owner
  Then the system denies the reassignment
  And leaves the current owner unchanged

@authorization @negative
Scenario: Prevent reassignment by an inactive or out-of-scope deputy
  Given the user's deputy assignment is inactive or does not cover the responsible Manager
  When the user attempts to change the registration owner
  Then the system denies the reassignment
  And leaves the current owner unchanged

@baseline @negative
Scenario: Prevent assignment to an ineligible worker
  Given the Manager may reassign the registration
  And the selected worker does not satisfy the approved target-owner rule
  When the Manager confirms the reassignment
  Then the system rejects the change
  And leaves the current owner unchanged
```

---

### US-PROD-001 - Record and Deactivate Client Products Without Duplicates

**User story**

> As a registration owner, I want to record and deactivate the client's products while preventing duplicate active products, so that the client's product portfolio remains accurate.

**Business rules**

- An authorized Agent or Manager may record a product sold to a client.
- The user must have the required access to the client registration.
- The system must prevent a confirmed duplicate product under the approved product-identity and active-state rules.
- Deactivation must record an effective date because it drives the one-year registration-termination rule.
- Product identity and the meaning of `active`, `offered`, and `sold` require resolution through `OQ-011`.

**Acceptance criteria**

```gherkin
@baseline
Scenario: Record a product not currently active for the client
  Given the signed-in user is the current registration owner
  And the selected product does not conflict with an active client product under the approved duplicate rule
  When the user records the product as sold
  Then the system links the product to the client registration
  And records the data required by the approved product lifecycle

@baseline @negative
Scenario: Prevent a duplicate active product
  Given the client has an active product matching the selected product under the approved product-identity rule
  When the registration owner attempts to record the product again
  Then the system rejects the action
  And does not create another active client-product relationship

@baseline @concurrency
Scenario: Prevent duplicate active products submitted concurrently
  Given the client does not currently have the selected active product
  When two authorized requests attempt to record the same product concurrently
  Then the system persists only one active client-product relationship
  And returns a duplicate or conflict outcome for the other request

@baseline
Scenario: Deactivate an active client product
  Given the signed-in user is authorized to maintain the registration
  And the client product is active
  When the user deactivates the product with a valid effective date
  Then the system marks the client product as inactive according to the agreed lifecycle
  And records the effective deactivation date

@authorization @negative
Scenario: Prevent product maintenance by an unauthorized non-owner
  Given the signed-in user is not authorized to edit the registration
  When the user attempts to add or deactivate a client product
  Then the system denies the action
  And leaves the client's product portfolio unchanged
```

**Pending refinement**

Whether a previously deactivated product may be sold again is not decided until `OQ-011` defines product identity and lifecycle rules.

---

### US-LIFE-001 - Terminate an Inactive Client Registration

**User story**

> As a distribution-network Manager, I want a client registration to terminate one year after its last product is deactivated, so that the registration lifecycle reflects the client's inactive product portfolio.

**Business rules**

- The trigger is the last product's effective deactivation date, not time spent awaiting approval.
- Termination requires that no active client product remains.
- The meaning of `one year`, time zone, reactivation behavior, and status transitions require resolution through `OQ-012` and `OQ-007`.
- A scheduled background process is an implementation proposal under `A-LIFE-001`, not the source business requirement itself.

**Provisional acceptance criteria**

```gherkin
@baseline @boundary
Scenario: Terminate the registration on the one-year anniversary
  Given all client products are inactive
  And the most recently deactivated product reached its one-year anniversary under the approved date rule
  When the registration lifecycle rule is evaluated
  Then the system marks the registration as terminated according to the agreed lifecycle model

@baseline @boundary
Scenario: Keep the registration before the one-year anniversary
  Given all client products are inactive
  And the most recently deactivated product has not reached its one-year anniversary
  When the registration lifecycle rule is evaluated
  Then the system does not terminate the registration

@baseline @negative
Scenario: Keep the registration while any product remains active
  Given at least one client product is active
  When the registration lifecycle rule is evaluated
  Then the system does not terminate the registration

@baseline @boundary
Scenario: Do not terminate after a product is reactivated before the anniversary
  Given all client products were inactive
  And a client product was reactivated before the applicable one-year anniversary
  When the registration lifecycle rule is evaluated
  Then the system does not terminate the registration
```

**Not decided in this story**

- whether `one year` means a calendar-year anniversary or 365 days;
- evaluation time and time zone;
- handling of a client that has never had a product;
- whether and how a terminated registration may be restored.

---

### US-IAM-001 - Maintain Workers, Permissions, and Organizational Units

**User story**

> As an Administrator, I want to maintain workers, permissions, and organizational units inside IFCOM+, so that access and reporting rules use an accurate distribution-network structure during phase 1.

**Business rules**

- IFCOM+ stores organizational, worker, role, and permission data internally during phase 1.
- Each worker belongs to exactly one Sales Point and has exactly one Manager.
- A Manager's direct reports may belong to different Sales Points but remain within one Region.
- Administrator maintenance actions include worker creation and editing, worker transfer, permission assignment, and organizational-unit creation, editing, and deactivation.
- Managers may access this application area, but the source assigns listed maintenance actions to Administrators. Any Manager write permission depends on `OQ-014`.

**Acceptance criteria**

```gherkin
@baseline
Scenario: Create a worker with a valid organizational assignment
  Given the Administrator has permission to maintain workers
  And the selected Sales Point and Manager form a valid relationship within one Region
  When the Administrator creates the worker with an approved role assignment, one Sales Point, and one Manager
  Then the system creates the worker
  And stores exactly one current Sales Point assignment
  And stores exactly one current Manager assignment

@baseline @negative
Scenario: Reject a worker without exactly one Sales Point and one Manager
  Given the Administrator is creating or editing a distribution-network worker
  When the submitted assignment has no Sales Point, multiple Sales Points, no Manager, or multiple Managers
  Then the system rejects the change
  And does not persist an invalid worker structure

@baseline
Scenario: Transfer a worker while preserving regional hierarchy rules
  Given the Administrator is editing an existing worker
  And the target Sales Point is compatible with the worker's Manager under the approved hierarchy rule
  When the Administrator confirms the transfer
  Then the system replaces the worker's current Sales Point assignment with the target Sales Point
  And preserves exactly one Sales Point assignment

@baseline @negative
Scenario: Reject a transfer that violates the Manager's regional scope
  Given the target Sales Point belongs to a Region outside the worker's Manager scope
  When the Administrator attempts to transfer the worker
  Then the system rejects the transfer
  And preserves the current organizational assignment

@baseline @authorization
Scenario: Grant a worker permission
  Given the Administrator is authorized to maintain worker permissions
  When the Administrator grants an application permission to the worker
  Then the permission becomes effective according to the approved access-control lifecycle

@baseline @authorization
Scenario: Revoke a worker permission
  Given the Administrator is authorized to maintain worker permissions
  And the worker currently has the application permission
  When the Administrator revokes that permission
  Then the permission is no longer effective according to the approved access-control lifecycle

@baseline
Scenario: Create and edit a Sales Point
  Given the Administrator has permission to maintain organizational units
  And the parent Region exists
  When the Administrator creates a valid Sales Point and later changes its editable data
  Then the system stores the Sales Point and the accepted changes
  And preserves the Region-to-Sales-Point hierarchy constraints

@baseline @negative
Scenario: Prevent unsafe deactivation of an organizational unit
  Given an organizational unit still has unresolved worker, Manager, reporting, or registration dependencies
  When the Administrator attempts to deactivate it
  Then the system does not deactivate the unit
  And identifies that dependencies must be resolved under the approved deactivation rule

@authorization @negative
Scenario: Prevent maintenance by a user without Administrator authority
  Given a signed-in user does not have Administrator maintenance authority
  When the user attempts to create, edit, transfer, grant, revoke, or deactivate administrative data
  Then the system denies the maintenance action
  And leaves the administrative data unchanged
```

**Pending refinement**

The dependency behavior for worker transfer and unit deactivation requires `OQ-013`. Authentication and account-control behavior requires `OQ-017`.

---

### US-RPT-001 - Review Management Performance Reports

**User story**

> As a Manager, I want registration and product-sales reports for my direct reports and their organizational units, so that I can evaluate activity within my management scope.

**Business rules**

- Four separate source reports must be supported.
- Reports must remain restricted to the Manager's direct-report and organizational scope.
- Registration status inclusion, sold-product event/date, time zone, filters, export, and refresh rules require resolution through `OQ-015` and `OQ-016`.

**Provisional acceptance criteria**

```gherkin
@baseline @authorization
Scenario: Report registered-company counts by direct report
  Given the reporting definitions and period are configured
  And the Manager has direct reports with registrations in scope
  When the Manager opens the registered-companies-by-worker report
  Then the system shows the registration count for each applicable direct report
  And calculates each count using the approved registration-status rule

@baseline @authorization
Scenario: Report registered-company counts by organizational unit
  Given the reporting definitions and period are configured
  And the Manager's direct reports belong to one or more Sales Points in scope
  When the Manager opens the registered-companies-by-unit report
  Then the system shows the registration count for each applicable organizational unit
  And calculates each count using the approved grouping rule

@baseline @authorization
Scenario: Report monthly product-sales counts by direct report
  Given the sold-product event and reporting month are defined
  When the Manager opens the monthly-products-by-worker report
  Then the system shows the product-sales count for each applicable direct report
  And includes only events inside the selected reporting month

@baseline @authorization
Scenario: Report monthly product-sales counts by organizational unit
  Given the sold-product event and reporting month are defined
  When the Manager opens the monthly-products-by-unit report
  Then the system shows the product-sales count for each applicable organizational unit
  And includes only events inside the selected reporting month

@authorization @negative
Scenario: Exclude activity outside the Manager's scope
  Given workers or organizational units outside the Manager's scope have reportable activity
  When the Manager opens any management report
  Then the system excludes the out-of-scope activity
  And does not expose out-of-scope worker or client details
```

**Not decided in this story**

- which registration statuses are counted;
- which product event and date constitute a sale;
- treatment of reassignment and historical organization changes;
- time zone, filters, export formats, refresh frequency, and retention.

## 5. Assumptions Not Promoted to Baseline Stories

The following catalog assumptions remain proposals and must not be presented as approved story scope:

| Assumption | Treatment in this document |
|---|---|
| `A-CRM-001` - IČO duplicate key | Mentioned as an option; baseline scenarios use the approved company key placeholder |
| `A-INT-001` - ARES integration | Excluded from baseline acceptance criteria |
| `A-APR-001` to `A-APR-004` - rejection, reason, notification, reminder | Isolated in explicitly tagged `@assumption` scenarios |
| `A-LIFE-001` - scheduled background process | Business outcome tested; scheduling mechanism not mandated |
| `A-API-001` - REST API | Deferred to the OpenAPI artifact |
| `A-SEC-001` to `A-SEC-003` - transport, password, and session controls | Deferred until security decisions are approved |
| `A-AUD-001` - audit trail | Decision metadata is useful, but a full audit requirement remains unapproved |
| `A-PERF-001` - response-time target | Deferred until a measurable target is approved |
| `A-USAB-001` - measurable usability criteria | Deferred to UX and non-functional refinement |

## 6. Story-Level Definition of Ready

A story may be marked `Ready for Development` only when:

- its primary actor and authorization scope are agreed;
- every blocking `OQ-*` has a recorded decision;
- any accepted `A-*` proposal has been promoted to a requirement;
- business terms and validation keys are defined;
- lifecycle states and transitions used by the story are agreed;
- the happy path, authorization failures, business-rule failures, and applicable boundary or concurrency cases are concrete;
- required data fields and error outcomes are defined;
- linked BPMN, RBAC, domain, API, database, and test artifacts use the same requirement and story IDs.

## 7. Coverage Summary

| Capability | Covered by |
|---|---|
| Company creation and duplicate prevention | `US-CRM-001` |
| Search, lists, owner-only detail and edit | `US-CRM-002` |
| Manager approval | `US-APR-001` |
| Manager/deputy ownership transfer | `US-RBAC-001` |
| Product recording, duplicate prevention, and deactivation | `US-PROD-001` |
| One-year termination rule | `US-LIFE-001` |
| Worker, permission, and organizational-unit administration | `US-IAM-001` |
| Four management reports | `US-RPT-001` |

Detailed cross-artifact traceability will be maintained in `docs/05-traceability-matrix.md` after the BPMN, RBAC, domain, API, database, and test artifacts are stabilized.
