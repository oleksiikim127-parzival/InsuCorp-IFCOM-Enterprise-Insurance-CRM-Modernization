# IFCOM+ RBAC and Business Rules

## 1. Document Control

| Field | Value |
|---|---|
| Document status | Draft policy baseline |
| Version | 0.1 |
| Author | Oleksii Kim |
| Requirements baseline | [IFCOM+ Requirements Catalog](01-requirements-catalog.md) |
| User stories | [User Stories and BDD Acceptance Criteria](02-user-stories.md) |
| Evidence boundary | [Project Origin and Scope](00-project-origin-and-scope.md) |

This document defines the authorization model and atomic business rules for IFCOM+. It separates:

- **capability authorization** - which role may request an action;
- **record scope** - which records may be listed or included in reports;
- **object authorization** - whether the user may open, edit, approve, or reassign one registration;
- **business-rule validation** - whether the requested state change is valid.

The source brief contains unresolved authorization conflicts. This document exposes them rather than inventing a final policy.

## 2. Evidence and Decision Status

| Status | Meaning |
|---|---|
| `BASELINE` | Directly implements a `SOURCE` requirement from the catalog |
| `DERIVED` | Adds a necessary enforcement or integrity rule without changing source intent |
| `PROVISIONAL` | Expresses baseline intent but depends on an unresolved `OQ-*` decision |
| `ASSUMPTION` | Implements an unapproved `A-*` proposal |
| `BLOCKED` | Cannot be implemented consistently until a named conflict is resolved |

`ALLOW` in a table means only that the stated rule would allow the action. Authentication, account state, explicit permission, object scope, lifecycle rules, and other business validations must still pass.

## 3. Authorization Model

### 3.1 Why This Is Not Pure RBAC

IFCOM+ uses a hybrid authorization model:

| Control dimension | Examples |
|---|---|
| Role | Agent, Manager, Administrator |
| Explicit permission | Create registration, manage workers, run reports |
| Ownership | Only the current registration owner may open or edit detail under the source baseline |
| Reporting relationship | Manager access is restricted to direct reports and their organizational context |
| Organizational membership | Agent list scope depends on both Sales Point and Manager |
| Region | A Manager's direct reports may span Sales Points but remain within one Region |
| Delegation | A deputy may reassign ownership only under a valid delegated scope |
| Lifecycle state | Approval, product deactivation, and termination actions depend on current state |

The model is therefore best described as **RBAC with record-level attribute and relationship rules**. A role alone never establishes access to a particular registration.

### 3.2 Default-Deny Decision Order

Every protected request should be evaluated in this order:

1. Confirm that the user is authenticated and the account is active.
2. Confirm that the user's effective role and explicit permission allow the capability.
3. Determine the applicable relationship and record scope using current organizational data.
4. Check object-specific authorization such as ownership, approval assignment, or deputy delegation.
5. Check business preconditions and lifecycle rules.
6. Execute the action atomically or deny it without exposing protected information.
7. If `A-AUD-001` is approved, record the security-relevant decision and outcome.

If any mandatory check fails, access is denied. No broader role may silently bypass an object rule unless an approved exception is documented.

### 3.3 Scope Predicates

The following predicates are used throughout the decision tables:

| Predicate | Definition |
|---|---|
| `IS_OWNER(U, R)` | User `U` is the one current owner of registration `R` |
| `SAME_SALES_POINT(U, O)` | User `U` and owner `O` belong to the same Sales Point |
| `SAME_MANAGER(U, O)` | User `U` and owner `O` have the same Manager |
| `IS_DIRECT_REPORT(O, M)` | Registration owner `O` reports directly to Manager `M` |
| `IS_MANAGER_OWNED(R, M)` | Manager `M` is the current owner of registration `R` |
| `SAME_REGION(U, M)` | User `U` and Manager `M` operate within the same Region under the approved hierarchy rule |
| `VALID_DEPUTY(U, M)` | User `U` has an active delegation to act for Manager `M` within the requested capability and time scope |
| `IN_REPORT_SCOPE(X, M)` | Worker, unit, or activity `X` belongs to Manager `M` under the approved report rule |
| `IS_AUTHORIZED_APPROVER(U, R)` | User `U` is the Manager selected by the approved approver-assignment rule for registration `R` |
| `HAS_PERMISSION(U, P)` | User `U` has effective permission `P` under the phase-1 internal authorization model |

`VALID_DEPUTY`, `IN_REPORT_SCOPE`, and `IS_AUTHORIZED_APPROVER` remain provisional until `OQ-008`, `OQ-015`/`OQ-016`, and `OQ-003` are resolved.

## 4. Role-to-Capability Matrix

Legend: `A` = allowed by baseline capability rule, `C` = conditional on record scope or unresolved policy, `D` = denied by baseline, `?` = unresolved.

| Capability | Agent | Manager | Administrator | Status | Traceability |
|---|:---:|:---:|:---:|---|---|
| Create company registration | A | A | D | `BASELINE` | `FR-CRM-001`, `US-CRM-001` |
| Search registration summaries | C | C | D | `BASELINE` | `FR-CRM-003`, `US-CRM-002` |
| View Agent organizational list | A | D | D | `BASELINE` | `FR-RBAC-001` |
| View My Registrations | A | A | D | `BASELINE` | `FR-RBAC-002`, `FR-RBAC-005` |
| View Manager direct-report list | D | A | D | `BASELINE` | `FR-RBAC-003` |
| View Manager organizational-unit list | D | C | D | `PROVISIONAL` | `FR-RBAC-004`, `OQ-010` |
| Open registration detail | C | C | D | `BASELINE` | `FR-RBAC-006`, `US-CRM-002` |
| Edit registration | C | C | D | `BASELINE` | `FR-RBAC-007`, `US-CRM-002` |
| Record or deactivate client product | C | C | D | `BASELINE` | `FR-PROD-001`, `FR-PROD-002`, `US-PROD-001` |
| Approve new registration | D | ? | D | `BLOCKED` | `FR-APR-001`, `OQ-003` to `OQ-005`, `US-APR-001` |
| Reject registration | D | ? | D | `ASSUMPTION` | `A-APR-001`, `A-APR-002` |
| Reassign registration ownership | D | C | D | `PROVISIONAL` | `FR-RBAC-009`, `FR-RBAC-010`, `US-RBAC-001` |
| View management reports | D | C | D | `PROVISIONAL` | `FR-RPT-001` to `FR-RPT-004`, `OQ-015`, `OQ-016` |
| Access administration area | D | A | A | `BASELINE` | `FR-IAM-004` |
| Maintain workers, permissions, and units | D | ? | A | `PROVISIONAL` | `FR-IAM-005` to `FR-IAM-011`, `OQ-014` |

The Administrator is not granted blanket access to client commercial data. The source assigns the Administrator responsibility for organizational structure and worker administration, not universal CRM visibility.

## 5. Record-Scope Decision Tables

### 5.1 Registration List Visibility

This table determines whether a registration **summary** may appear in a named list. It does not grant detail or edit access.

| Rule | Requested view | User role | Owner relation | Organizational condition | Result | Status | Traceability |
|---|---|---|---|---|---|---|---|
| `POL-LIST-001` | Agent organizational list | Agent | Any owner | `SAME_SALES_POINT(User, Owner)` and `SAME_MANAGER(User, Owner)` | ALLOW summary | `BASELINE` | `FR-RBAC-001` |
| `POL-LIST-002` | Agent organizational list | Agent | Any owner | Either Sales Point or Manager differs | DENY | `BASELINE` | `FR-RBAC-001` |
| `POL-LIST-003` | My Registrations | Agent or Manager | `IS_OWNER(User, Registration)` | None | ALLOW summary | `BASELINE` | `FR-RBAC-002`, `FR-RBAC-005` |
| `POL-LIST-004` | My Registrations | Agent or Manager | User is not owner | None | DENY | `BASELINE` | `FR-RBAC-002`, `FR-RBAC-005` |
| `POL-LIST-005` | Direct-report registrations | Manager | `IS_DIRECT_REPORT(Owner, User)` | None | ALLOW summary | `BASELINE` | `FR-RBAC-003` |
| `POL-LIST-006` | Direct-report registrations | Manager | Owner is outside direct reports | None | DENY | `BASELINE` | `FR-RBAC-003` |
| `POL-LIST-007` | Organizational-unit registrations | Manager | Owner is associated with a direct report's unit | Exact population unresolved | PENDING DECISION | `PROVISIONAL` | `FR-RBAC-004`, `OQ-010` |

Search results must be filtered by a list scope available to the user. A search must not reveal whether an inaccessible registration exists.

### 5.2 Registration Detail and Edit Access

| Rule | User condition | Requested action | Result | Status | Traceability |
|---|---|---|---|---|---|
| `POL-OBJ-001` | `IS_OWNER(User, Registration)` | Open detail | ALLOW | `BASELINE` | `FR-RBAC-006`, `US-CRM-002` |
| `POL-OBJ-002` | User is not current owner | Open detail | DENY | `BASELINE` | `FR-RBAC-006`, `US-CRM-002` |
| `POL-OBJ-003` | `IS_OWNER(User, Registration)` and user has edit capability | Edit registration | ALLOW | `BASELINE` | `FR-RBAC-007`, `US-CRM-002` |
| `POL-OBJ-004` | User is not current owner | Edit registration | DENY | `BASELINE` | `FR-RBAC-007`, `US-CRM-002` |
| `POL-OBJ-005` | User sees registration summary through a list but is not owner | Open or edit detail | DENY | `BASELINE` | `FR-RBAC-001` to `FR-RBAC-007` |
| `POL-OBJ-006` | Administrator without a separately approved exception | Open or edit client registration | DENY | `DERIVED` | `FR-IAM-003` to `FR-IAM-011` |

This owner-only policy conflicts with Manager approval, because an approver normally requires enough information to decide. The exception must be explicitly resolved; it must not be implied from the Manager role.

### 5.3 Approval Authorization

| Rule | Condition | Action | Result | Status | Traceability |
|---|---|---|---|---|---|
| `POL-APR-001` | Registration requires approval and `IS_AUTHORIZED_APPROVER(User, Registration)` | Record approval | PROVISIONAL ALLOW | `BLOCKED` | `FR-APR-001`, `OQ-003`, `OQ-004` |
| `POL-APR-002` | User is not the authorized approver | Record approval | DENY | `PROVISIONAL` | `FR-APR-001`, `OQ-003` |
| `POL-APR-003` | Manager is also registration owner | Self-approve | PENDING DECISION | `BLOCKED` | `OQ-005` |
| `POL-APR-004` | Approval step already has a final decision | Submit another decision | DENY | `DERIVED` | `US-APR-001` |
| `POL-APR-005` | Authorized Manager and rejection extension approved | Reject registration | PROVISIONAL ALLOW | `ASSUMPTION` | `A-APR-001`, `OQ-006` |
| `POL-APR-006` | Rejection selected without reason and reason extension approved | Reject registration | DENY | `ASSUMPTION` | `A-APR-002`, `OQ-006` |

No implementation should mark approval rules as complete until the owner-only conflict and self-approval decision are closed.

### 5.4 Ownership Reassignment

| Rule | Initiator | Current owner | Target owner | Result | Status | Traceability |
|---|---|---|---|---|---|---|
| `POL-OWN-001` | Manager | Manager themself or direct report | Eligible under approved target rule | ALLOW | `PROVISIONAL` | `FR-RBAC-009`, `FR-RBAC-010`, `OQ-009` |
| `POL-OWN-002` | `VALID_DEPUTY(User, Manager)` | Manager or Manager's direct report | Eligible under approved target rule | ALLOW | `PROVISIONAL` | `FR-RBAC-009`, `FR-RBAC-010`, `OQ-008`, `OQ-009` |
| `POL-OWN-003` | Manager | Outside Manager and direct reports | Any | DENY | `BASELINE` | `FR-RBAC-010` |
| `POL-OWN-004` | User without valid Manager or deputy authority | Any | Any | DENY | `BASELINE` | `FR-RBAC-009`, `FR-RBAC-010` |
| `POL-OWN-005` | Authorized initiator | In scope | Target violates approved eligibility rule | DENY | `PROVISIONAL` | `OQ-009` |
| `POL-OWN-006` | Authorized initiator | In scope | Same as current owner | NO CHANGE / reject redundant action | `DERIVED` | `FR-CRM-007` |

After a successful reassignment, exactly one current owner remains. Owner-only detail and edit access move to the new owner; the previous owner no longer receives owner access.

### 5.5 Product Maintenance Authorization

| Rule | User condition | Requested action | Result | Status | Traceability |
|---|---|---|---|---|---|
| `POL-PROD-001` | Registration owner with product capability | Record sold product | ALLOW if product rules pass | `DERIVED` | `FR-PROD-001`, `US-PROD-001` |
| `POL-PROD-002` | Registration owner with product capability | Deactivate active product | ALLOW if lifecycle rules pass | `BASELINE` | `FR-PROD-002`, `US-PROD-001` |
| `POL-PROD-003` | User is not authorized to edit the registration | Add or deactivate product | DENY | `DERIVED` | `FR-RBAC-007`, `US-PROD-001` |

### 5.6 Administration and Reporting Scope

| Rule | User condition | Requested action | Result | Status | Traceability |
|---|---|---|---|---|---|
| `POL-ADM-001` | Administrator with effective maintenance permission | Create/edit worker, transfer worker, grant/revoke permission, create/edit/deactivate unit | ALLOW if integrity rules pass | `BASELINE` | `FR-IAM-003`, `FR-IAM-005` to `FR-IAM-011` |
| `POL-ADM-002` | Agent | Access administration area or perform maintenance | DENY | `BASELINE` | `FR-IAM-004` |
| `POL-ADM-003` | Manager | Access administration area | ALLOW | `BASELINE` | `FR-IAM-004` |
| `POL-ADM-004` | Manager | Perform maintenance operation | PENDING DECISION | `PROVISIONAL` | `OQ-014` |
| `POL-RPT-001` | Manager and `IN_REPORT_SCOPE(Activity, Manager)` | Include activity in report | ALLOW | `PROVISIONAL` | `FR-RPT-001` to `FR-RPT-004`, `OQ-015`, `OQ-016` |
| `POL-RPT-002` | Manager and activity is outside report scope | Include activity in report | DENY | `BASELINE` | `FR-RPT-001` to `FR-RPT-004` |
| `POL-RPT-003` | Agent or Administrator without approved reporting role | View Manager report | DENY | `DERIVED` | `FR-RPT-001` to `FR-RPT-004` |

## 6. Atomic Business Rule Catalog

### 6.1 Client and Registration Rules

| Rule ID | Rule statement | Status | Traceability |
|---|---|---|---|
| `BR-CRM-001` | A new registration may be created only by an Agent or Manager with effective create permission. | `BASELINE` | `FR-CRM-001`, `US-CRM-001` |
| `BR-CRM-002` | The system must evaluate the approved company matching rule before persisting a new registration. | `BASELINE` | `FR-CRM-002`, `FR-CRM-006`, `US-CRM-001` |
| `BR-CRM-003` | A confirmed duplicate company must not result in a second registration. | `BASELINE` | `FR-CRM-006`, `US-CRM-001` |
| `BR-CRM-004` | Concurrent registration requests must not create more than one registration for the same approved company key. | `DERIVED` | `FR-CRM-006`, `US-CRM-001` |
| `BR-CRM-005` | A duplicate response must not reveal owner identity or inaccessible client data. | `DERIVED` | `FR-RBAC-006`, `US-CRM-001` |
| `BR-CRM-006` | Every registration has exactly one current owner. | `BASELINE` | `FR-CRM-007` |
| `BR-CRM-007` | The worker who creates a registration becomes its initial owner. | `BASELINE` | `FR-RBAC-008` |
| `BR-CRM-008` | Search returns only summaries permitted by at least one approved list scope. | `DERIVED` | `FR-CRM-003`, `US-CRM-002` |

The approved company key remains unresolved through `OQ-001`. IČO is a proposal under `A-CRM-001`, not a baseline fact.

### 6.2 Product and Lifecycle Rules

| Rule ID | Rule statement | Status | Traceability |
|---|---|---|---|
| `BR-PROD-001` | A sold product record must be linked to the applicable client registration. | `DERIVED` | `FR-PROD-001` |
| `BR-PROD-002` | A client may not have more than one active relationship for the same product under the approved product-identity rule. | `BASELINE` | `FR-PROD-003` |
| `BR-PROD-003` | Concurrent product requests must not create duplicate active client-product relationships. | `DERIVED` | `FR-PROD-003`, `US-PROD-001` |
| `BR-PROD-004` | Product deactivation must record an effective deactivation date. | `DERIVED` | `FR-PROD-002`, `FR-LIFE-001` |
| `BR-LIFE-001` | A registration is eligible for termination only if no client product is active. | `BASELINE` | `FR-LIFE-001` |
| `BR-LIFE-002` | The one-year boundary is anchored to the effective deactivation event that leaves the client with no active product. | `DERIVED` | `FR-LIFE-001` |
| `BR-LIFE-003` | The registration must not terminate before the approved one-year boundary. | `BASELINE` | `FR-LIFE-001`, `US-LIFE-001` |
| `BR-LIFE-004` | The registration must terminate when the approved one-year boundary is reached and no product is active. | `BASELINE` | `FR-LIFE-001`, `US-LIFE-001` |
| `BR-LIFE-005` | If product reactivation is supported, reactivating a product before the boundary removes termination eligibility while that product remains active. | `PROVISIONAL` | `FR-LIFE-001`, `OQ-012`, `US-LIFE-001` |

Whether `one year` means a calendar anniversary or 365 days remains unresolved through `OQ-012`.

### 6.3 Ownership and Approval Rules

| Rule ID | Rule statement | Status | Traceability |
|---|---|---|---|
| `BR-OWN-001` | Only a Manager or valid deputy may initiate ownership reassignment. | `BASELINE` | `FR-RBAC-009` |
| `BR-OWN-002` | A Manager or deputy may initiate reassignment only when the current owner is that Manager or one of the Manager's direct reports. | `BASELINE` | `FR-RBAC-010` |
| `BR-OWN-003` | A successful reassignment replaces the current owner; it does not add a second current owner. | `DERIVED` | `FR-CRM-007`, `FR-RBAC-009` |
| `BR-OWN-004` | After reassignment, owner-only access follows the new owner and is removed from the previous owner. | `DERIVED` | `FR-RBAC-006`, `FR-RBAC-007`, `US-RBAC-001` |
| `BR-APR-001` | Every newly submitted registration requires one Manager approval before the registration process can complete. | `BASELINE` | `FR-APR-001` |
| `BR-APR-002` | One approval step may have no more than one final decision. | `DERIVED` | `FR-APR-001`, `US-APR-001` |
| `BR-APR-003` | Rejection is available only if `A-APR-001` is approved. | `ASSUMPTION` | `A-APR-001`, `OQ-006` |
| `BR-APR-004` | If rejection is approved, a rejection reason is mandatory only if `A-APR-002` is also approved. | `ASSUMPTION` | `A-APR-002`, `OQ-006` |
| `BR-APR-005` | A 48-hour reminder must be non-interrupting and must not decide or terminate the approval task. | `ASSUMPTION` | `A-APR-004` |

### 6.4 Organizational Integrity Rules

| Rule ID | Rule statement | Status | Traceability |
|---|---|---|---|
| `BR-ORG-001` | `Region` and `Sales Point` are the supported organizational-unit types. | `BASELINE` | `FR-ORG-001` |
| `BR-ORG-002` | The structure contains four Regions, each with one or more Sales Points. | `BASELINE` | `FR-ORG-002` |
| `BR-ORG-003` | Each distribution-network worker has exactly one current Sales Point. | `BASELINE` | `FR-ORG-003` |
| `BR-ORG-004` | Each distribution-network worker has exactly one current Manager. | `BASELINE` | `FR-ORG-004` |
| `BR-ORG-005` | Each Manager has one or more direct reports. | `BASELINE` | `FR-ORG-005` |
| `BR-ORG-006` | A Manager's direct reports may span Sales Points but must remain within one Region. | `BASELINE` | `FR-ORG-006` |
| `BR-ORG-007` | A Sales Point may have zero or more assigned workers. | `BASELINE` | `FR-ORG-007` |
| `BR-ORG-008` | A worker transfer must preserve exactly one Sales Point and one Manager assignment. | `DERIVED` | `FR-IAM-007`, `US-IAM-001` |
| `BR-ORG-009` | A worker transfer must not violate the Manager's single-Region scope. | `DERIVED` | `FR-ORG-006`, `FR-IAM-007` |
| `BR-ORG-010` | An organizational unit must not be deactivated while unresolved dependencies would violate ownership, hierarchy, access, or reporting integrity. | `PROVISIONAL` | `FR-IAM-011`, `OQ-013` |

### 6.5 Reporting Rules

| Rule ID | Rule statement | Status | Traceability |
|---|---|---|---|
| `BR-RPT-001` | Registered-company counts by worker include only applicable direct reports of the requesting Manager. | `BASELINE` | `FR-RPT-001` |
| `BR-RPT-002` | Registered-company counts by unit include only organizational units relevant to the Manager's direct reports. | `BASELINE` | `FR-RPT-002` |
| `BR-RPT-003` | Monthly product-sales counts by worker include only applicable direct reports and events inside the approved reporting month. | `PROVISIONAL` | `FR-RPT-003`, `OQ-015`, `OQ-016` |
| `BR-RPT-004` | Monthly product-sales counts by unit include only applicable units and events inside the approved reporting month. | `PROVISIONAL` | `FR-RPT-004`, `OQ-015`, `OQ-016` |
| `BR-RPT-005` | Report results must exclude out-of-scope workers, units, registrations, products, and activity. | `DERIVED` | `FR-RPT-001` to `FR-RPT-004`, `US-RPT-001` |

## 7. Sensitive Data and Response Rules

These rules support least-privilege behavior but do not constitute a claim of GDPR or security compliance.

| Rule ID | Rule statement | Status | Traceability |
|---|---|---|---|
| `BR-SEC-001` | Denial responses must not disclose whether an inaccessible registration exists. | `DERIVED` | `US-CRM-002` |
| `BR-SEC-002` | Duplicate responses must not disclose another worker's identity unless a separate policy authorizes it. | `DERIVED` | `US-CRM-001` |
| `BR-SEC-003` | List and report responses must expose only fields needed for the requested view. | `DERIVED` | `FR-RBAC-001` to `FR-RBAC-005`, `FR-RPT-001` to `FR-RPT-004` |
| `BR-SEC-004` | A denied action must not change protected business data. | `DERIVED` | All authorization scenarios in `02-user-stories.md` |
| `BR-SEC-005` | Administrative access does not imply access to client commercial data. | `DERIVED` | `FR-IAM-003` to `FR-IAM-011` |

## 8. Conflict and Decision Register

| Conflict / decision | Why it matters | Required resolution | Affected rules | Status |
|---|---|---|---|---|
| Owner-only detail vs Manager approval | A Manager cannot make an informed approval decision without an approved view or review mechanism | Decide approver, permitted data view, purpose limitation, and whether approval is an exception to owner-only access | `POL-OBJ-002`, `POL-APR-001`, `OQ-003`, `OQ-004` | `BLOCKED` |
| Manager self-approval | Manager may own registrations and is also the approval role | Decide whether self-approval is allowed or segregation of duties is required | `POL-APR-003`, `OQ-005` | `BLOCKED` |
| Rejection workflow | Rejection, reason, notification, and resubmission are absent from the source | Approve or reject `A-APR-001` to `A-APR-003` and define lifecycle behavior | `POL-APR-005`, `POL-APR-006`, `OQ-006`, `OQ-007` | `OPEN` |
| Manager organizational-unit view | The source wording does not produce one unambiguous record population | Define included owners, units, grouping, and whether results are summaries or aggregates | `POL-LIST-007`, `OQ-010` | `OPEN` |
| Deputy delegation | The source names a deputy but does not define assignment or scope | Define delegator, activation, expiry, capability scope, and revocation | `POL-OWN-002`, `OQ-008` | `OPEN` |
| Target owner eligibility | Reassignment initiator scope is stated; target-worker scope is not | Define same Manager, Sales Point, Region, role, and active-account constraints | `POL-OWN-001`, `POL-OWN-005`, `OQ-009` | `OPEN` |
| Product identity and active state | Duplicate prevention cannot be enforced without a canonical key and lifecycle | Define product key, offered/sold/active states, resale and reactivation rules | `BR-PROD-002`, `OQ-011` | `OPEN` |
| One-year calculation | Calendar-year and 365-day rules differ around leap years and time zones | Define boundary calculation, time zone, evaluation schedule, and reactivation behavior | `BR-LIFE-002` to `BR-LIFE-005`, `OQ-012` | `OPEN` |
| Manager administrative rights | Managers may access the area, but listed maintenance actions belong to Administrators | Define read-only access or explicitly approved Manager write capabilities | `POL-ADM-003`, `POL-ADM-004`, `OQ-014` | `OPEN` |
| Report populations and metrics | Counts may change based on statuses, sale dates, reassignment, and historical organization | Define metrics, history rules, filters, time zone, and export behavior | `POL-RPT-001`, `BR-RPT-001` to `BR-RPT-005`, `OQ-015`, `OQ-016` | `OPEN` |

## 9. Required Enforcement Points

The same rules must be enforced consistently at every applicable boundary:

| Boundary | Required behavior |
|---|---|
| User interface | Hide or disable unavailable actions for usability, without treating the UI as the security control |
| Application/API authorization | Enforce capability and record-scope policy on every request |
| Domain service | Enforce ownership, lifecycle, and business invariants independently of transport |
| Database | Enforce structural invariants such as one current owner and duplicate-prevention constraints where technically appropriate |
| Background processing | Re-evaluate authorization-independent lifecycle preconditions before automated changes |
| Reporting queries | Apply Manager scope and approved metric definitions in the query or governed semantic layer |
| Audit trail | If `A-AUD-001` is approved, record sensitive grants, denials, approval decisions, reassignments, permission changes, and automated termination outcomes |

Hiding an action in the UI is not sufficient. Direct API, concurrent, batch, and background paths must preserve the same policies.

## 10. Verification Coverage

At minimum, authorization testing must cover:

- every `ALLOW` rule with a valid in-scope example;
- every `DENY` rule with an out-of-role and out-of-scope example;
- ownership changes before and after reassignment;
- Sales Point, Manager, and Region boundary conditions;
- active, expired, and out-of-scope deputy assignments;
- attempts to infer inaccessible registrations through search and duplicate responses;
- Manager approval by authorized, unauthorized, and self-approving users after policy resolution;
- Agent and Administrator attempts to access Manager reports;
- Manager attempts to perform Administrator maintenance after `OQ-014` is resolved;
- concurrent company and product creation;
- permission revocation taking effect according to the approved access-control lifecycle;
- report queries containing out-of-scope activity;
- automated lifecycle processing with active products, pre-boundary dates, and reactivation.

Detailed executable test IDs will be added in `docs/06-test-scenarios.md`. Cross-artifact links will be consolidated in `docs/05-traceability-matrix.md`.

## 11. Policy Exit Criteria

This document may move from `Draft` to `Reviewed` when:

- all `BLOCKED` and material `OPEN` conflicts have recorded decisions;
- Manager approval is reconciled with owner-only detail access;
- deputy and target-owner eligibility rules are explicit;
- list, search, detail, edit, approval, reassignment, administration, and report scopes are independently testable;
- accepted `A-*` proposals are promoted to requirements and rejected proposals are removed from active policy;
- policy rules map to domain services, API authorization, database constraints, and tests;
- no role receives implicit access to client data beyond the documented policy.
