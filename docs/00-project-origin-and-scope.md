# Project Origin and Scope

## 1. Purpose of This Document

This document explains where the InsuCorp / IFCOM+ case originated, which information came from the source brief, which elements were developed independently, and what the repository does and does not represent.

Its purpose is to make the portfolio case transparent and to preserve traceability between the original business context and the additional business and systems analysis artifacts.

## 2. Project Origin

This case is based on a technical take-home assignment received by **Oleksii Kim** during a real Business Analyst recruitment process. The material is presented here in anonymized portfolio form.

The source brief described the modernization of a legacy B2B insurance CRM used by a Czech distribution network. It presented:

- the AS-IS problems of the legacy IFCOM application;
- the expected TO-BE capabilities of IFCOM+;
- client registration and product-management rules;
- registration ownership, visibility, approval, and reassignment rules;
- the organizational structure and the Agent, Manager, and Administrator roles;
- user and organizational-unit administration;
- four management-reporting requirements;
- high-level technical, availability, localization, volume, and device constraints.

The original assignment asked the candidate to:

1. identify and organize the functional and non-functional requirements;
2. group the requirements into business areas or modules;
3. convert five to ten selected requirements into user stories;
4. define acceptance criteria for each selected story; and
5. define a project epic.

The repository uses **InsuCorp** as a portfolio-facing case name. The source brief used the scenario names **POJFIRM**, **IFCOM**, and **IFCOM+**. The recruiting organization and any potentially identifying information are intentionally not disclosed.

## 3. Nature of the Portfolio Work

This repository is an independent extension of the recruitment assignment into a structured Business and Systems Analysis portfolio case.

The source brief supplied the initial domain context, actors, business rules, and technical constraints. All detailed requirements, models, acceptance criteria, data structures, API proposals, SQL artifacts, assumptions, and test scenarios in this repository were created independently by Oleksii Kim for portfolio and learning purposes unless explicitly marked otherwise.

No stakeholder interviews were conducted for this portfolio case. The analysis is based on the written take-home brief and clearly identified analytical assumptions.

This repository does **not** represent:

- a production implementation;
- paid client work or employment experience;
- a completed commercial delivery;
- requirements formally validated or approved by business stakeholders;
- access to a client's production systems, databases, source code, or internal documentation;
- a legal, security, audit, or regulatory compliance assessment.

## 4. Traceability Classification

Every requirement, business rule, model, and technical decision should use one of the following classifications:

| Classification | Meaning |
|---|---|
| `SOURCE` | Explicitly stated in the original recruitment brief. |
| `DERIVED` | Logically elaborated from one or more source requirements without changing their business intent. |
| `ASSUMPTION` | Proposed by the analyst because the source brief does not provide enough detail. It requires stakeholder validation before implementation. |
| `OPEN QUESTION` | A material point that cannot be resolved from the available source and requires a stakeholder or Product Owner decision. |

An assumption must not silently replace or contradict a source requirement. Where a proposed solution changes the original scope or introduces a new business rule, it must remain clearly labeled as an assumption.

Examples of portfolio extensions that are **not stated in the source brief** and therefore require the `ASSUMPTION` classification include:

- integration with the Czech ARES register;
- local IČO checksum validation;
- REST endpoint design and specific HTTP status codes;
- a 48-hour reminder for a pending manager review;
- a rejection workflow and mandatory rejection reason;
- specific response-time targets;
- cloud deployment;
- detailed registration status names beyond those required to express the source lifecycle.

## 5. Source-Defined Scope

The following areas are within the business domain and solution boundary established by the original brief. Some capabilities are stated explicitly, while others are motivated by an AS-IS problem and therefore require `DERIVED` classification in the detailed requirements catalog.

### 5.1 Client and Product Management

- create new company registrations;
- verify, search, view, and edit registered companies;
- record products sold to a client;
- address the risk of selling or offering a product that the client already has;
- deactivate client products;
- automatically terminate a client registration one year after the client's last product is deactivated.

### 5.2 Registration Ownership and Approval

- assign ownership to the worker who created a registration or to a worker selected by an authorized Manager or deputy;
- restrict registration detail access and editing to the registration owner;
- allow ownership reassignment only within the scope defined by the organizational hierarchy;
- require newly submitted registrations to be approved by a Manager.

### 5.3 Organizational Structure and Access

- support four regions containing one or more sales points;
- assign each worker to one sales point and one Manager;
- allow Managers to supervise workers across sales points within one region;
- provide role- and hierarchy-based registration lists for Agents and Managers;
- provide a personal `My Registrations` list to both Agents and Managers.

### 5.4 User and Organizational-Unit Administration

- create and edit workers;
- transfer a worker between organizational units;
- grant and revoke application permissions;
- create, edit, and deactivate organizational units;
- maintain the organizational structure, users, and roles inside IFCOM+ during the first project phase because no external identity and authorization repository is planned for that phase.

### 5.5 Management Reporting

- number of registered companies per subordinate worker;
- number of registered companies per organizational unit containing subordinate workers;
- number of products sold per subordinate worker per month;
- number of products sold per organizational unit per month.

### 5.6 Source Technical and Non-Functional Constraints

- web application;
- Java based on J2EE standards;
- deployment on the application-server environment identified as `Boss` in the supplied brief;
- stated availability of 24 hours per day, 365 days per year;
- expected processing volume of approximately 20 registrations per day;
- Czech and Slovak language versions;
- support for desktop computers, laptops, and tablets.

These source statements still require refinement into measurable and testable requirements where the brief does not provide sufficient acceptance thresholds.

## 6. Independent Portfolio Extension Scope

The portfolio may extend the source-defined scope with clearly classified analytical and technical artifacts, including:

- a complete modular requirements catalog;
- user stories and BDD acceptance criteria;
- business rules and RBAC decision tables;
- BPMN process models;
- UML domain, state, and sequence models;
- a data dictionary and relational data model;
- a proposed OpenAPI 3.1 contract;
- SQL DDL, constraints, validation queries, and reporting queries;
- negative, boundary, authorization, and integration test scenarios;
- a requirements traceability matrix;
- assumptions, open questions, and out-of-scope decisions.

These artifacts demonstrate an analysis approach. They do not imply that the proposed solution was implemented or approved by a real client.

## 7. Out of Scope

Unless added later as a separately identified portfolio extension, the following items are outside the scope of this case:

- development of a working IFCOM+ application;
- production deployment, hosting, and operational support;
- final infrastructure or cloud architecture;
- migration of production data from the legacy IFCOM application;
- detailed insurance pricing, underwriting, claims, billing, or payment processing;
- integration with external systems other than explicitly labeled proposals;
- formal privacy, GDPR, cybersecurity, or financial-audit certification;
- proof of SLA, performance, availability, or security compliance;
- final UI design and usability validation with real users.

## 8. Known Limitations

- The available source is a written recruitment brief rather than a complete stakeholder-approved specification.
- Some business terms, lifecycle states, exception paths, and authorization edge cases are not fully defined.
- No Product Owner or subject-matter expert was available to confirm assumptions.
- Proposed integration and technical designs are illustrative and must be validated against organizational architecture, security, privacy, and operational standards before implementation.
- Any quantitative target not explicitly present in the source brief must be treated as an open question or an assumption, not as an approved requirement.

## 9. Intended Use

The repository is intended to demonstrate the ability to:

- extract and structure requirements from an incomplete business brief;
- distinguish source requirements from analytical proposals;
- model business processes, roles, data, system behavior, and exceptions;
- maintain end-to-end traceability from source to requirement, design, and test;
- communicate limitations and unresolved decisions transparently.

This document should be read before the detailed requirements and solution artifacts because it defines the evidence boundary for the entire case.
