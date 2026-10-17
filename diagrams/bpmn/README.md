# Client Registration and Approval — BPMN Guide

[Project overview](../../README.md) · [Requirements](../../docs/01-requirements-catalog.md) · [Business rules](../../docs/03-rbac-and-business-rules.md) · [Assumptions and questions](../../docs/04-assumptions-open-questions.md)

**Status: draft analytical model.** The diagram combines source-backed behavior with explicit proposals. Approval policy and registration-state decisions remain open; the model is not an executable workflow or a stakeholder-approved specification.

![Client registration and approval](client-registration-approval.svg)

## Files and Editing Ownership

| File | Role |
|---|---|
| [client-registration-approval.bpmn](client-registration-approval.bpmn) | Primary BPMN 2.0 editing source for Camunda Modeler; `isExecutable="false"` |
| [client-registration-approval.drawio](client-registration-approval.drawio) | Matching diagrams.net copy retained for interchange |
| [client-registration-approval.svg](client-registration-approval.svg) | Scalable preview of the reviewed diagram |
| [client-registration-approval.png](client-registration-approval.png) | Raster preview of the same diagram |

Edit the `.bpmn` file in Camunda Modeler first. The author selected Camunda for further modeling. Keep the secondary `.drawio` copy synchronized with the activities, lane assignments, connections, event semantics and evidence labels. The files do not synchronize automatically. Previews are generated artifacts, not separate process definitions.

## Process Boundary

This view covers an **Agent-initiated registration**: entry and submission, company-duplicate checking, record creation with initial ownership, routing, Manager review and its outcomes.

Managers may also create registrations under `FR-CRM-001`; their initiation variant is not drawn here. The diagram does not silently permit a Manager to approve their own record.

The following are outside this view:

- detailed form validation, correction loops, technical failures and duplicate-check concurrency handling;
- ownership reassignment, products, administration and reports;
- the separate registered-company verification capability whose meaning is open in `OQ-002`;
- automatic termination one year after the last product is deactivated.

These omissions are abstraction boundaries, not claims that the corresponding requirements or acceptance scenarios are unnecessary. See [user stories](../../docs/02-user-stories.md) for the related cases. The one-year rule belongs in the planned registration lifecycle/state model.

## How to Read the Diagram

| Lane | Responsibility |
|---|---|
| Agent | Enters and submits the registration |
| IFCOM+ System | Checks duplicates, creates the record, assigns work, records outcomes and performs proposed notifications |
| Manager | Reviews the assigned request and makes an authorized decision |

All three lanes are siblings inside one pool. Solid sequence flows may cross between these lanes. There are no message flows inside this pool.

The pool has its own 30-pixel title strip on the left. Lane bounds begin to the right of that strip and have their own separate title strips. In BPMN diagram interchange, the pool and lanes must not share the same left coordinate for this layout; otherwise Camunda renders their labels on top of each other. This geometry was corrected after the author's Camunda import screenshot exposed the overlap. Activities and process connections were preserved.

White activities show source-backed or derived behavior. Orange activities carry `PROPOSED A-*` labels. Red activities identify an unresolved policy dependency with `BLOCKED OQ-*`. These colors are explanatory conventions; they do not change BPMN execution semantics.

Activity borders and sequence flows are solid, including proposed behavior. Dotted associations connect explanatory notes. The double dashed boundary around the timer means **non-interrupting**, not merely "proposed". The empty diamonds are exclusive gateways; the optional X marker is omitted consistently.

`BLOCKED` is a documentation status: it does not represent a runtime wait or an implemented authorization check.

## Walkthrough

1. The Agent enters and submits company-registration data.
2. The system checks for an existing company using the matching rule to be agreed under `OQ-001`. IČO remains a proposal under `A-CRM-001`.
3. A confirmed duplicate leads to neutral feedback and an end event without creating another registration.
4. Otherwise, the system creates one registration and assigns its creator as the initial owner.
5. The system assigns a Manager. The selection and fallback rules remain open in `OQ-003`.
6. The Manager reviews the request. The permitted approval view and self-approval policy remain open in `OQ-004` and `OQ-005`.
7. On approval, the system records the decision. The owner-notification step is proposed under `A-APR-003`.
8. The proposed rejection branch requires a reason, records rejection and notifies the owner. Rejection, its reason and notification are separate proposals under `A-APR-001`, `A-APR-002` and `A-APR-003`.

"Approval completed" and "Proposed rejection outcome" describe process outcomes. They do not establish `Active` or `Rejected` as approved database states; state names and transitions remain open in `OQ-007`.

## The 48-Hour Reminder

The timer is attached to **Review registration request**. In the `.drawio` source it is a child of that task, so moving the task also moves the timer. In BPMN XML it uses `attachedToRef="Task_ReviewRegistration"` and `cancelActivity="false"`.

While review is active, the proposed duration is `PT48H`. If the duration elapses, the timer creates a separate reminder branch. The system sends the reminder and that branch reaches an ordinary end event. The original review stays active. The reminder does not loop back into review, merge with its decision path, or terminate the process.

If review finishes before the timer fires, that reminder does not start. `PT48H` models one elapsed-time reminder, not a recurring timer or 48 business hours. The duration, calendar and operational handling remain unvalidated details of `A-APR-004`.

`OQ-018` and `OQ-019` concern availability and performance/capacity; they do not themselves define an approval SLA or business calendar. `OQ-017` concerns authentication, not notification delivery. Notification channels and failure behavior are not specified by this diagram.

## Traceability

| Element | Classification | References |
|---|---|---|
| Enter and submit registration | SOURCE; Agent variant | `FR-CRM-001`, `US-CRM-001` |
| Check company duplication | DERIVED; matching key open | `FR-CRM-006`, `BR-CRM-002` to `BR-CRM-005`, `OQ-001`; separate verification meaning: `OQ-002` |
| Neutral duplicate feedback | DERIVED | `BR-CRM-003`, `BR-CRM-005`, `BR-SEC-002` |
| Create registration; assign creator | SOURCE | `FR-CRM-007`, `FR-RBAC-008`, `BR-CRM-006`, `BR-CRM-007` |
| Assign authorized approver | SOURCE obligation; policy BLOCKED | `FR-APR-001`, `OQ-003`, `POL-APR-001`, `POL-APR-002` |
| Manager review | SOURCE obligation; policy BLOCKED | `FR-APR-001`, `FR-RBAC-006`, `OQ-004`, `OQ-005`, `POL-OBJ-002`, `POL-APR-003` |
| Record approval | SOURCE; state representation open | `FR-APR-001`, `BR-APR-001`, `BR-APR-002`, `OQ-007` |
| Rejection branch | ASSUMPTION | `A-APR-001`, `OQ-006`, `OQ-007` |
| Rejection reason | ASSUMPTION | `A-APR-002`, `OQ-006` |
| Outcome notification | ASSUMPTION | `A-APR-003` |
| Non-interrupting reminder | ASSUMPTION | `A-APR-004`, `BR-APR-005` |

## Revision History and What Was Preserved

The earlier repository diagram was renamed to `client-registration-approval.drawio` in [commit 741fb0c](https://github.com/oleksiikim127-parzival/InsuCorp-IFCOM-Enterprise-Insurance-CRM-Modernization/commit/741fb0c60f07c1747d8e68a65364f48eedd71c44) without changing its contents. A revised BPMN model and a later Draw.io equivalent were prepared separately. This revision reconciles that existing work.

| Original repository model | Previously prepared revision retained here |
|---|---|
| Manager lane nested inside the system lane | Three sibling lanes with explicit responsibilities |
| `Check IČO for duplicates` | Agreed matching rule; IČO explicitly remains proposed |
| `Save record (Status: Pending)` | Record creation and initial ownership; status vocabulary stays open |
| `Notify Manager` | Explicit approver assignment with `OQ-003`; notification is not treated as an agreed routing rule |
| Review without a visible access-policy dependency | `OQ-004` and `OQ-005` beside Manager review |
| Interrupting timer and a return connector without a bound target | Non-interrupting timer; independent reminder branch with its own end |
| Rejection, reason and notifications appeared unconditional | Separate `A-APR-*` proposal labels |
| `Active` and `Rejected` appeared as final status names | Recorded outcomes with `OQ-007` for the state model |

The original duplicate message did **not** name another Agent; the neutral-feedback wording clarifies the access boundary rather than correcting a demonstrated disclosure. The original diagram also did **not** contain the one-year termination flow; this guide makes its separate scope explicit.

The current reconciliation makes sequence flows and activity borders solid, attaches the Draw.io timer to its task, routes its connector outside review, aligns BPMN layout with the visual source, regenerates the previews and corrects the guide's references. Core BPMN flow-node and sequence-flow IDs are retained. Draw.io cells carry `bpmnElement` attributes for cross-format comparison.

## Editing Checklist

1. Open the `.bpmn` file in Camunda Modeler and edit the primary source.
2. Preserve the pool, three sibling lanes, event types and the explicit proposal/question labels.
3. Keep every sequence flow attached to its source and target. Keep notes connected with associations.
4. Synchronize the secondary `.drawio` copy when nodes, flows, lane ownership, labels or timer behavior change.
5. Check duplicate, approval, proposed rejection and reminder paths. The reminder must not cancel or restart review.
6. Export updated SVG and PNG previews from the reviewed diagram. Inspect the exports for missing labels and overlapping connectors.
7. Check the guide, root README, related requirements and relative links before committing the complete set.

For this revision, structural XML checks, flow/lane equivalence, reference checks, image decoding and visual preview inspection were performed. The author imported the previous file into Camunda and reported overlapping pool/lane labels; the corrected file has separate title-strip bounds and unchanged process semantics. A subsequent Camunda re-import has not yet been verified. The model was not deployed to a workflow engine.

## Next Modeling Step

Decide how the portfolio will represent the unresolved approval policy and document any selected option as an analyst design assumption unless actual stakeholder evidence exists. Then create the separate lifecycle/state model. The BPMN file being readable and internally consistent does not close those business questions or finish stage 4.
