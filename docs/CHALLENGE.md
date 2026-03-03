# Ergon Hackathon 2026 – QueryVault Challenge

- [Ergon Hackathon 2026 – QueryVault Challenge](#ergon-hackathon-2026--queryvault-challenge)
    - [Scenario](#scenario)
    - [Data Model Requirements](#data-model-requirements)
        - [1 Create the Core Custom Object](#1-create-the-core-custom-object)
        - [2 Required Fields and Intended Meaning](#2-required-fields-and-intended-meaning)
    - [App and UX Requirements (LWC)](#app-and-ux-requirements-lwc)
        - [3 Provide a Library Experience (Browse + Find)](#3-provide-a-library-experience-browse--find)
        - [4 Provide a Query Create/Edit Experience](#4-provide-a-query-createedit-experience)
        - [5 SObject Name Selection](#5-sobject-name-selection)
    - [Validation Requirements](#validation-requirements)
        - [6 Validate SOQL Syntax Before Save](#6-validate-soql-syntax-before-save)
        - [7 Prevent Harmful or Unbounded Validation Runs](#7-prevent-harmful-or-unbounded-validation-runs)
    - [CRUD Requirements](#crud-requirements)
        - [8 Create Records (CRUD: “C”)](#8-create-records-crud-c)
        - [9 Read Records (CRUD: “R”)](#9-read-records-crud-r)
        - [10 Update Records (CRUD: “U”)](#10-update-records-crud-u)
        - [11 Delete Records (CRUD: “D”)](#11-delete-records-crud-d)
    - [Security \& Access Requirements](#security--access-requirements)
        - [12 Enforce Access via Permission Sets](#12-enforce-access-via-permission-sets)
        - [13 Respect Object Access When Surfacing SObjects](#13-respect-object-access-when-surfacing-sobjects)
    - [Quality Requirements](#quality-requirements)
        - [14 Deliver Seed Data for Demo](#14-deliver-seed-data-for-demo)
        - [15 Provide a Minimal “How to Use” Guide](#15-provide-a-minimal-how-to-use-guide)
    - [MVP+ Requirements (Optional)](#mvp-requirements-optional)
        - [S1 “Run Preview” (Safe Sample Execution)](#s1-run-preview-safe-sample-execution)
        - [S2 Pagination \& Performance Scaling](#s2-pagination--performance-scaling)
        - [S3 Organized Query Library](#s3-organized-query-library)
        - [S4 Bulk Data Handling](#s4-bulk-data-handling)
    - [MVP++ Requirements (Optional)](#mvp-requirements-optional-1)
        - [A1 Usage Analytics](#a1-usage-analytics)
        - [A2 Enhanced Object Design](#a2-enhanced-object-design)
        - [A3 Advanced UX Pattern](#a3-advanced-ux-pattern)
    - [Submission Checklist (What “Done” Looks Like)](#submission-checklist-what-done-looks-like)
    - [Technical Resources](#technical-resources)

---

> ⚠️ **Official Source of Truth**  
> This Markdown version mirrors the official problem statement.  
> In the event of discrepancies, the official Word document controls.
>
> 📄 **Official Problem Statement (Word Doc):**  
> [Open the official document](https://ergonus-my.sharepoint.com/:w:/g/personal/aberrios_ergon_com/IQDNdQY3qlsVRKyyz4K3b66QASz6lmW6-iEa4dDNHutVQd4?e=r1CFka)

---

## Scenario

The Ergon Salesforce team wants a centralized, reusable library of “useful SOQL queries” that can be safely shared across Admins, Developers, and QA.

Build **QueryVault**, an LWC-based CRUD app that lets users create, validate, maintain, and reuse SOQL queries tied to Salesforce objects, without relying on scattered docs.

You are given the business intent and desired behaviors, but you must design the implementation details.

---

## Data Model Requirements

### 1 Create the Core Custom Object

**Requirement:**  
Implement a custom object to store reusable queries.

**Success Criteria**

- A custom object exists named `UsefulQuery__c`.
- Records represent one reusable query.
- The object includes all standard/custom fields outlined in #2.

---

### 2 Required Fields and Intended Meaning

**Requirement:**  
Add fields to support naming, reuse, and clarity.

**Success Criteria**

- A required **Name** field exists and is user-meaningful (e.g., “Active Customers by Status”, “Open Cases by Priority and Status”).
- A required **SOQL field** exists:
    - Stores the query text (supports multi-line content).
- A required **Description field** exists:
    - Provides verbose context and usage notes (not used as the record’s primary label).
- A required **SObject API Name field** exists (`SObjectApiName__c`).

---

## App and UX Requirements (LWC)

### 3 Provide a Library Experience (Browse + Find)

**Requirement:**  
Build an LWC library UI that allows users to discover existing query records quickly.

**Success Criteria**

- Users can view a list of saved queries.
- Users can search by at least one of:
    - Name
    - Description
    - SOQL text (optional)
- Users can filter by SObject (using the selected object mechanism described below).
- The list avoids unnecessarily loading heavy content (e.g., SOQL body) for every row unless it’s intentionally displayed.

---

### 4 Provide a Query Create/Edit Experience

**Requirement:**  
Build an LWC editor for creating and updating query records.

**Success Criteria**

- Users can create a new query record from the UI.
- Users can edit an existing query record from the UI.
- The editor includes:
    - Name (required)
    - Description (required)
    - SObject selection (required)
    - SOQL query text (required)
- The form UX clearly indicates required fields and prevents incomplete submission.

---

### 5 SObject Name Selection

**Requirement:**  
Users must select a valid SObject without typing the API name freehand. This should be done dynamically (e.g., via the Schema class or equivalent dynamic describe approach).

**Success Criteria**

- The UI provides a selectable list or searchable dropdown (see Lightning Combobox) of available SObjects.
- The stored SObject API Name is populated from the user’s selection.
- The user cannot save a record with an arbitrary/unknown SObject API Name that was typed manually.
- The solution works without hardcoding object API names.
- This requirement intentionally reduces reliance on controller-side Apex logic for “is object valid?” by providing query fetched object names.

---

## Validation Requirements

### 6 Validate SOQL Syntax Before Save

**Requirement:**  
Detect SOQL syntax errors and prevent saving invalid queries on record to create form submission.

**Success Criteria**

When the user attempts to create/save a useful query:

- Queries with SOQL syntax errors are detected reliably.
- The user receives a clear error message that indicates the query cannot run due to a syntax error.
- The system does not silently accept invalid SOQL.

> **Note:** You decide whether validation happens on-demand (“Validate” button) or automatically on Save, but your UX must make the validation outcome unambiguous.

---

### 7 Prevent Harmful or Unbounded Validation Runs

**Requirement:**  
Validation must be safe and performance conscious.

**Success Criteria**

- The validation design does not run an unbounded query that could return massive results.
- The implementation includes at least one guardrail such as:
    - Enforcing a maximum row limit during validation
    - Rejecting unsafe patterns for validation execution
    - Constraining validation to parse/compile checks rather than full data retrieval
- Validation completes quickly for typical inputs.

---

## CRUD Requirements

### 8 Create Records (CRUD: “C”)

**Requirement:**  
Users can create useful query records from the LWC.

**Success Criteria**

- A user can successfully create a record with all required fields populated.
- The record appears in the library list after creation.
- If validation fails, the user is prevented from creating the record.

---

### 9 Read Records (CRUD: “R”)

**Requirement:**  
Users can open and view stored queries.

**Success Criteria**

- Users can view Name, Description, SObject, and SOQL.
- Users can copy the SOQL text for reuse.
    - A copy button (or something similar) must be provided.
    - Cursor dragging and copying text is not sufficient.
- View experience is accessible from the library list.

---

### 10 Update Records (CRUD: “U”)

**Requirement:**  
Users can update saved queries and keep them trustworthy.

**Success Criteria**

- Users can edit Name/Description/SObject/SOQL.
- If SOQL or SObject changes, the system re-validates under the same rules as creation.
- Updates persist and are reflected in the library list.

---

### 11 Delete Records (CRUD: “D”)

**Requirement:**  
Users can delete records with guardrails.

**Success Criteria**

- Users can delete a query record from the UI.
- The delete action requires user intent (confirmation pattern or equivalent).
- After deletion, the record no longer appears in the library.

---

## Security & Access Requirements

### 12 Enforce Access via Permission Sets

**Requirement:**  
Ensure only the right users can create/edit/delete, while others can view.

**Success Criteria**

- At least two access levels are defined (e.g., Viewer vs Editor).
- Users without edit permissions can still browse and view records but cannot modify them.
- The app respects object-level CRUD and does not bypass security.

---

### 13 Respect Object Access When Surfacing SObjects

**Requirement:**  
The SObject selection list must be appropriate for the current user context.

**Success Criteria**

- The UI does not present objects the user can’t reasonably interact with (you choose the definition: “readable objects only” is a common approach).
- If a user somehow references an object they cannot access, the system handles it gracefully (clear message and safe failure).

---

## Quality Requirements

### 14 Deliver Seed Data for Demo

**Requirement:**  
Provide sample records demonstrating realistic usage.

**Success Criteria**

- At least 10 sample `UsefulQuery__c` records exist (insertable via script, data loader, or manual creation).
- Records span multiple objects (e.g., Account, Case, Contact, Contract\_\_c, etc.).
- Records include meaningful Names and Descriptions.

---

### 15 Provide a Minimal “How to Use” Guide

**Requirement:**  
Document how users should use the app.

**Success Criteria**

A short README or internal doc exists describing:

- What the app is for
- How to find and copy a query
- How validation works (high level)
- Any guardrails/limitations (e.g., validation enforces a limit)

---

## MVP+ Requirements (Optional)

### S1 “Run Preview” (Safe Sample Execution)

**Success Criteria**

- Users can preview results (first N rows).
- Preview is constrained to a safe row count and shows clear errors.

---

### S2 Pagination & Performance Scaling

**Success Criteria**

- Library view implements pagination (server-side preferred).
- A row limit is enforced per page (e.g., 25–50).
- Searching does not require loading all records into memory.
- UI clearly indicates the total record count.

---

### S3 Organized Query Library

**Success Criteria**

At least one of:

- Queries grouped by SObject in the UI
- `Category__c` picklist field for grouping
- Collapsible sections by object
- Tag-based grouping
- Category filter component

---

### S4 Bulk Data Handling

**Success Criteria**

At least one of:

- Documented mass-insert process using Salesforce Inspector
- Apex script for seed data
- Data Loader template
- Multi-record creation capability in the LWC

The goal is to demonstrate scalability of the solution, not necessarily build a full mass-edit UI.

---

## MVP++ Requirements (Optional)

### A1 Usage Analytics

**Success Criteria**

- Track interaction metrics (views, copies, last used).
- Provide a “most used queries” view.

---

### A2 Enhanced Object Design

**Success Criteria**

Teams extend `UsefulQuery__c` with thoughtful design enhancements such as:

- `Category__c` (Picklist)
- `Tags__c` (Text)
- `IsActive__c` (Checkbox)
- `UsageCount__c` (Number)
- `LastUsedDate__c` (DateTime)

Teams must justify additional fields in documentation.

---

### A3 Advanced UX Pattern

**Success Criteria**

- Clear separation between Browse and Create/Edit (tabs or routed view)
- Inline editing in datatable
- Modal-based editor
- Smart state handling between views

---

## Submission Checklist (What “Done” Looks Like)

- Custom Object + required fields (Name, Description, SOQL, SObject API Name)
- LWC Library + Editor
- Assisted SObject selection (MVP)
- SOQL syntax validation with safety guardrails
- Full CRUD in UI
- Permission-based access control
- Demo seed data + minimal guide

---

## Technical Resources

- [Lightning Component Library](https://developer.salesforce.com/docs/platform/lightning-component-reference/guide/components.html)
- [Lightning Design System](https://www.lightningdesignsystem.com/2e1ef8501/p/85bd85-lightning-design-system-2)
- [Apex Reference Docs](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_ref_guide.htm)
