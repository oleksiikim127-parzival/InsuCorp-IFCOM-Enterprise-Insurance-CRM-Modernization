# Business Requirements Document (BRD)

## Module 1: Client & Product Management (CRM Core)

| ID | Type | Requirement Description |
| :--- | :--- | :--- |
| **FR_1.01** | Functional | The system must allow Agents to input new client (company) registrations. |
| **FR_1.02** | Functional | The system must prevent duplicate client registration (validation of IČO before saving). |
| **FR_1.03** | Functional | The system must allow tracking sold products linked to a specific client and their subsequent deactivation. |
| **FR_1.04** | Functional | The system must prevent the sale of a duplicate product that the client already owns. |
| **FR_1.05** | Security/Access | The system must display to the Agent only the list of registrations owned by agents who share the same Manager AND belong to the same organizational unit. |
| **FR_1.06** | Security/Access | The system must display to the Manager the list of registrations from their direct subordinates and registrations from the organizational units of their subordinates. |
| **FR_1.07** | Workflow | A newly created client registration must be subject to Manager approval. The system must allow the Manager to reject the registration and mandate inputting a rejection reason. |
| **FR_1.08** | Automated | The system must automatically terminate a client registration if it remains in the "Pending Approval" status for more than 30 days without Manager action. |

*Note: These high-level business requirements serve as the foundation for the BPMN process models and detailed BDD User Stories.*
