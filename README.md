# 🏢 InsuCorp (IFCOM+) — Enterprise Insurance CRM Modernization

## 📖 1. Business Context & Executive Summary

**POJFIRM** is a commercial insurance provider operating in the Czech Republic, specializing in B2B insurance products. The company is migrating from its legacy, slow, and restricted desktop application (**IFCOM**) to a modern, cloud-based, and highly responsive web platform (**IFCOM+**).

### 🛑 The Problem (AS-IS)
- **Operational Inefficiency:** Field sales agents lack direct access to the legacy CRM system. They are forced to fill out paper forms with clients, mail them to the home office, and wait days for manual data entry.
- **Data Corruption & Duplicates:** Manual transcription leads to frequent typos. Without real-time verification, agents routinely register duplicate company profiles or offer insurance products the client already owns.
- **Process Bottlenecks:** The contract approval workflow is opaque, leading to stagnant applications and lost sales.

### 🎯 Project Objectives & Business Value (TO-BE)
- **100% Paperless Workflows:** Equipping field agents with a responsive web-app to onboard clients directly from mobile devices.
- **Data Integrity & Prescriptive Validation:** Eradicating database clutter via automated real-time checks (e.g., duplicate IČO lookup).
- **Role-Based Governance:** Securing commercial data visibility based on a strict regional and managerial hierarchy.

---

## 📋 2. Business Requirements Document (BRD)

*Extracted from initial stakeholder interviews and translated into formal functional requirements.*

| ID | Category | Requirement Description |
| :--- | :--- | :--- |
| **FR_1.01** | Functional | The system must allow Agents to input new B2B client registrations. |
| **FR_1.02** | Validation | The system must prevent duplicate client registrations by validating the Czech Company ID (IČO) before saving. |
| **FR_1.03** | Functional | The system must allow tracking of sold products linked to a specific client and their subsequent deactivation. |
| **FR_1.04** | Validation | The system must prevent the sale of a duplicate product that the client currently actively owns. |
| **FR_1.07** | Workflow | A newly created client registration must be routed for Manager approval. The Manager can approve or reject (mandating a rejection reason). |
| **FR_1.08** | Automated SLA | The system must automatically archive/terminate a client registration if it remains in "Pending Approval" for > 30 days. |

---

## 🔐 3. Security & Role-Based Access Control (RBAC) Matrix

*The system enforces strict data isolation based on organizational structure (4 Regions $\rightarrow$ Sales Points).*

| Role | Data Visibility Scope (Read) | Permissions (Write/Execute) |
| :--- | :--- | :--- |
| **Agent** | Only records owned by agents who share the *same Manager* AND belong to the *same Sales Point*. | Create Client, Add Product |
| **Manager** | Records from their direct subordinates + records from the Sales Points of their subordinates. | Approve/Reject Registrations |
| **System Admin**| Full system visibility (metadata only, no commercial data). | Manage Org Structure & Users |

*(Ref: Requirements FR_1.05 and FR_1.06)*

---

## 🏃‍♂️ 4. Agile Requirements (BDD User Stories)

To ensure clear hand-offs to the development and QA teams, critical business rules are documented using Gherkin syntax.

### 📌 US-1.02: Prevent Duplicate Company Registration (IČO Validation)
**As a** Field Agent,  
**I want** the system to automatically check if a company already exists based on its IČO before saving,  
**So that** I do not create duplicate records and database integrity is maintained.

> **Acceptance Criteria (BDD - Gherkin):**
> 
> **Given** the Agent is on the "New Client Registration" form
> **When** the Agent enters "12345678" into the "IČO" field
> **And** a client profile with IČO "12345678" already exists in the database
> **Then** the system should disable the "Submit" button
> **And** display a validation error: "Company with this IČO is already registered by [Agent Name]."

### 📌 US-1.07: Managerial Approval Workflow
**As a** Regional Manager,  
**I want** to review and approve or reject new client registrations submitted by my agents,  
**So that** only verified and compliant companies enter our active CRM portfolio.

---

## 🗄️ 5. Data Architecture (Core Data Dictionary)

Primary entity specification designed to support the workflow statuses, duplicate prevention constraints, and security filtering.

**Entity:** `Client_Profile`

| Attribute Name | Data Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `client_id` | UUID | PK, Auto-generate | Unique identifier for the client record. |
| `company_name` | VARCHAR(255) | NOT NULL | Legal name of the registered company. |
| `ico` | VARCHAR(8) | NOT NULL, UNIQUE | 8-digit Czech Company ID. Prevents duplicates. |
| `registration_status`| ENUM | Default: 'Pending' | Values: *Pending, Active, Rejected, Archived*. |
| `rejection_reason` | TEXT | NULLABLE | Populated only if status = 'Rejected' by Manager. |
| `assigned_agent_id` | UUID | FK, NOT NULL | Links to Users table (Creator). |
| `assigned_manager_id`| UUID | FK, NOT NULL | Links to Users table (Used for RBAC routing). |
| `created_at` | TIMESTAMP | CURRENT_TIMESTAMP | Audit and SLA tracking (Ref: FR_1.08). |

---

## 📊 6. Business Process Modeling (BPMN 2.0)

**Process:** New Client Registration & Approval Workflow (TO-BE)

This diagram visualizes the end-to-end automated process, highlighting system lane validations (IČO duplicates), managerial approval XOR gateways, and the automated 30-day SLA timeout event (FR_1.08).

![BPMN - Client Registration Workflow](diagrams/client_registration_flow.png)

## ⚙️ 7. System Integration (UML Sequence Diagram)

**Process:** IČO Validation & ARES API Data Enrichment (US-1.02)

To ensure high data quality and optimize UX, the system performs a multi-tier validation process when an Agent inputs a company IČO. It uses local frontend checksum validation (Modulo 11) to prevent unnecessary API calls, checks the local CRM database for duplicates, and finally integrates with the Czech State Register (ARES API) to auto-populate company details.

```mermaid
sequenceDiagram
    autonumber
    actor Agent
    participant UI as IFCOM+ Frontend
    participant Backend as IFCOM+ Backend (CRM Core)
    participant DB as CRM Database
    participant ARES as ARES API (Gov Register)

    Agent->>UI: Enters IČO (e.g., 12345678)
    Note over UI: Tier 1: Mathematical Validation (Modulo 11)
    
    alt Invalid Checksum
        UI-->>Agent: Validation Error: "Invalid IČO format"
    else Valid Checksum
        UI->>Backend: GET /api/clients/validate/{ico}
        
        Note over Backend: Tier 2: Internal Duplicate Check
        Backend->>DB: SELECT id FROM Client_Profile WHERE ico = {ico}
        DB-->>Backend: Result
        
        alt IČO exists in DB
            Backend-->>UI: 409 Conflict (Duplicate Error)
            UI-->>Agent: Error: "Client already registered by [Agent Name]"
        else IČO is unique
            Note over Backend: Tier 3: External Data Enrichment
            Backend->>ARES: GET /ares/api/v2/ekonomicke-subjekty/{ico}
            ARES-->>Backend: 200 OK (JSON: Company Name, Address)
            Backend-->>UI: 200 OK (Mapped Company Data)
            UI-->>Agent: Auto-fills form & Displays "Confirm Company"
        end
    end
