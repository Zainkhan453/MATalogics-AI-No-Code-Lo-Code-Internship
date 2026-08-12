# Day 6 — Airtable as a No-Code Database + n8n Automation

Using **Airtable** as the central relational database for an AI agency, then wiring it into **n8n** to run CRUD operations and five event-driven automation workflows across Slack and Gmail.

---

## Objective

Learn Airtable as a no-code relational database — bases, tables, fields, views, forms, and automations — then connect it to n8n to build a lightweight AI-agency operations system: lead management, client onboarding, project tracking, AI-agent monitoring, and an internship performance tracker.

---

## Module 1 & 2 — The Airtable base

**Base:** `MATalogics AI Operations Base` (5 linked tables, each with a Grid view).

| Table | Fields | Sample record |
|-------|--------|---------------|
| **Clients** | Client ID, Name, Company, Email, Status | Ali Raza · NexaTech Solutions · Onboarding |
| **Projects** | Project Name, Deadline, Status, Last Modified | AI Customer Support Chatbot · 25/8/2026 · Completed |
| **Leads** | Lead Name, Source, Contact Number, Interested Service, Created Time | zain khan · Website · AI Automation |
| **AI Agents** | Agent Name, Type, Deployment Status, Last Updated | Customer Support RAG Agent · RAG Agent · Deployed |
| **Interns** | Intern Name, Status, Performance Score, Last Modified | Zain Khan · Task Completed · 80 |

Table screenshots are in [`Airtable/Screenshots/`](Airtable/Screenshots/).

**Airtable terminology learned:** Base → Table → Record (row) → Field (column) → View (Grid / Kanban / Calendar / Gallery / Form) → Interface → Automation.

---

## Module 3 — Airtable + n8n CRUD

**Workflow:** `Airtable CRUD Operations - Day 6.json`

A single manual-trigger workflow demonstrating all four operations against the base via the native n8n **Airtable** node:

| Operation | n8n action |
|-----------|-----------|
| **Create** | `create` — add a new record |
| **Search** | `search` — look up records |
| **Update** | `update` — modify an existing record |
| **Delete** | `deleteRecord` — remove a record |

Authentication uses an **Airtable OAuth2** credential (connected once, reused across every workflow) — no tokens are hardcoded in the JSON.

---

## Module 4 — Automation workflows

All five workflows are triggered by the native n8n **Airtable Trigger** (polling for new/changed records) and were verified end-to-end — proof screenshots are in [`n8n/Screenshots/`](n8n/Screenshots/).

### 1. Lead Management — `Lead Management - Day 6.json`
New record in **Leads** → **Airtable Trigger** → **Slack** message to `#lead_allerts`:

> 🔔 New Lead Received — Name, Source, Contact, Interested Service, Created Time.

### 2. Client Onboarding — `Client Onboarding - Day 6.json`
New record in **Clients** → **Code** node generates a unique Client ID → **Airtable** writes it back → **Slack** notifies `#client-onboarding`.

The Client ID is generated as `CL-YYYYMMDD-XXXX`, where `XXXX` is the last 4 characters of the Airtable record ID, uppercased:

```js
const shortId = record.id.replace('rec', '').slice(-4).toUpperCase();
const clientId = `CL-${year}${month}${day}-${shortId}`;
```

Result: `CL-20260812-UIGJ` was written to Airtable and posted to Slack automatically.

### 3. Project Tracking — `Project Tracking - Day 6.json`
Project status changes in **Projects** → **Airtable Trigger** → **Gmail** sends a status-update email.

Verified with two emails: one at status *In Progress*, one at *Completed* — same project, showing the trigger fires on each status change.

### 4. AI Agent Monitoring — `AI Agent Monitoring - Day 6.json`
Deployment status changes in **AI Agents** → **Airtable Trigger** → **Slack** message to `#operations-alerts`:

> 🤖 AI Agent Status Update — Agent Name, Type, Deployment Status, Last Updated.

### 5. Internship Tracker — `Internship Tracker - Day 6.json`
Record changes in **Interns** → **Airtable Trigger** → **If** (status = *Task Completed*) → **Code** increments the score → **Airtable** updates the Performance Score automatically.

---

## Module 5 — Practical assignment: AI Agency Database

The five tables + five workflows together form the required AI-agency system:

| System | Built from |
|--------|-----------|
| Client Management System | Clients table + Client Onboarding workflow |
| Lead Management System | Leads table + Lead Management workflow |
| Project Tracker | Projects table + Project Tracking workflow |
| Internship Tracker | Interns table + Internship Tracker workflow |
| AI Agent Tracker | AI Agents table + AI Agent Monitoring workflow |

---

## What I learned

- **Airtable as a database** — how a base, tables, records, fields, and views map onto relational-database concepts, and why Airtable beats a plain spreadsheet for structured, automatable data.
- **Field types** — single-line text, email, single-select (Status/Source/Type), number (Performance Score), and auto-computed `Created Time` / `Last Modified` fields.
- **Airtable + n8n** — connecting via OAuth2 and running Create / Search / Update / Delete from n8n.
- **Event-driven automation** — using the Airtable Trigger to react to new and changed records, then fanning out to Slack and Gmail.
- **Generating & writing back data** — using a Code node to compute a value (Client ID) and writing it back into the source record.
- **Conditional logic** — gating an automation on a field value with an If node before updating a score.

---

## Folder contents

```
Day-06/
├── Airtable/
│   └── Screenshots/          # Clients, Projects, Leads, AI Agents, Interns tables
├── n8n/
│   ├── Airtable CRUD Operations - Day 6.json
│   ├── Lead Management - Day 6.json
│   ├── Client Onboarding - Day 6.json
│   ├── Project Tracking - Day 6.json
│   ├── AI Agent Monitoring - Day 6.json
│   ├── Internship Tracker - Day 6.json
│   └── Screenshots/          # Slack + Gmail automation results
└── README.md
```

---

## Importing the workflows

1. In n8n: **Workflows → Import from File** → select any JSON in `Day-06/n8n/`.
2. Reconnect the **Airtable OAuth2**, **Slack**, and **Gmail** credentials to your own accounts (the JSON references credentials by ID only — no secrets are included).
3. In each Airtable node, point the **Base** and **Table** selectors at your own `MATalogics AI Operations Base` and matching table.
4. Update the Slack channel IDs and the Gmail recipient to your own.
5. Click **Active** to enable the trigger.

---

**Zain Khan** — AI Automation Intern, MATalogics
