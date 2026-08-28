# Zonix Customer Onboarding System — Technical Documentation

## System Overview
The **Zonix Customer Onboarding System** is an automated pipeline designed to streamline lead intake, stage-based client/team notifications, and SLA tracking using Zapier Tables, Zapier Interfaces, and Zapier Automation Workflows.

---

## 1. System Architecture & Database Schema

### Database: `Customer Onboarding Pipeline` (Zapier Table)

| Field Name | Data Type | Logic / Dropdown Options | Description |
| :--- | :--- | :--- | :--- |
| **`Client Name`** | Text | User Input | Name of the client contact |
| **`Company`** | Text | User Input | Name of the client's company |
| **`Email`** | Text / Email | User Input | Client's primary email address |
| **`Service`** | Dropdown | `Web Design`, `Consulting`, `Marketing` | Requested service type |
| **`Project Budget`** | Currency | User Input | Total project budget value |
| **`Start Date`** | Date | User Input | Requested project start date |
| **`Requirements`** | Long Text | User Input | Project scope and client needs |
| **`Account Manager Email`**| Dropdown | Team Email Choices | Assigned internal manager email |
| **`Due Date`** | Date | Calculated (+7 days) | Automatically set upon creation |
| **`Status`** | Single Select | `New Lead`, `Qualified`, `Proposal`, `Onboarding`, `In Progress`, `Completed` | Pipeline Kanban stage |

### Interface
* **Kanban Board & Form Path:** `/pipeline-board`
* **Features:** Public intake form for clients and an internal Kanban board for team stage management.

---

## 2. Zap Automation Workflows

### Zap 1: `01 - New Lead Onboarding`
* **Trigger:** Zapier Tables $\rightarrow$ **New Record** in `Customer Onboarding Pipeline`.
* **Step 2 (Action):** Zapier Tables $\rightarrow$ **Update Record**
  * Auto-assigns default **`Account Manager Email`**.
  * Calculates **`Due Date`** as `+7 days`.
* **Step 3 (Action):** Email by Zapier $\rightarrow$ **Send Outbound Email**
  * Sends dynamic client welcome email from `Zonix Operations Team`.

---

### Zap 2: `02 - Pipeline Stage Movements`
* **Trigger:** Zapier Tables $\rightarrow$ **Updated Record** (`Trigger Field: Status`).
* **Router Steps (Paths by Zapier):**
  * **Path A (`Qualified`):** Emails Account Manager $\rightarrow$ `Lead Qualified: [1. Client Name]`.
  * **Path B (`Proposal`):** Emails Account Manager $\rightarrow$ `Action Required: Prepare Proposal for [1. Client Name]`.
  * **Path C (`Onboarding`):** Emails Client (`1. Email`) $\rightarrow$ `Welcome to Onboarding with Zonix!`.
  * **Path D (`In Progress`):** Emails Project Manager $\rightarrow$ `New Active Project: [1. Company]`.
  * **Path E (`Completed`):** Emails Client (`1. Email`) $\rightarrow$ `Your project with Zonix is Complete!`.

---

### Zap 3: `03 - Qualified 3-Day SLA Alert`
* **Step 1 (Trigger):** Zapier Tables $\rightarrow$ **Updated Record** (`Trigger Field: Status`).
* **Step 2 (Filter 1):** `1. Status` (Value) `(Text) Exactly matches` `Qualified`.
* **Step 3 (Delay):** Delay by Zapier $\rightarrow$ **Delay For** `3 Days`.
* **Step 4 (Find Record):** Zapier Tables $\rightarrow$ Searches record by `1. ID`.
* **Step 5 (Filter 2):** `4. Status` (Value) `(Text) Exactly matches` `Qualified`.
* **Step 6 (Action):** Email by Zapier $\rightarrow$ Sends escalation alert to `4. Account Manager Email` (Value):
  * **Subject:** `URGENT: Lead stuck in Qualified stage for 3+ days`

---

## 3. Test Cases & Verification Results

| Test ID | Test Objective | Procedure | Expected Result | Status |
| :--- | :--- | :--- | :--- | :---: |
| **TC-01** | Lead Ingestion & Auto-Assignment | Submit form with `Service: Web Design`, budget, and contact info. | New row created in table, `Account Manager Email` populated, `Due Date` (+7 days) set, client confirmation email delivered. | **PASSED** |
| **TC-02** | Kanban Stage Notifications | Move card sequentially through all 5 stage columns (`Qualified` $\rightarrow$ `Completed`). | Triggered dedicated notification emails for each path (Paths A–E) to respective internal managers and clients. | **PASSED** |
| **TC-03A** | SLA Delay Escalation | Move card to `Qualified` and allow delay timer to elapse. | Account Manager receives SLA alert: `URGENT: Lead stuck in Qualified stage...`. | **PASSED** |
| **TC-03B** | SLA Gatekeeper Cancellation | Move card to `Qualified`, then move to `Proposal` before delay timer expires. | Zap 3 Step 5 (Filter 2) cancels execution. No false escalation email is sent. | **PASSED** |

---

## 4. Operational Best Practices

1. **Mapping Variables:**
   * Use **`Value`** for technical dropdown matching (Filter conditions, `To:` fields, search steps).
   * Use **`Label`** for human-readable text placed in email bodies and subject lines.
2. **Kanban Operations:**
   * Always perform stage movements via the live published interface (`/pipeline-board`) rather than inside the Interface Editor layout view.
   * Allow 15–30 seconds between column drags to permit Zapier webhooks to process sequentially.
