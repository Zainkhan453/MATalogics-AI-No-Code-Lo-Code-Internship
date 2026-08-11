# Day 5 — Slack & Notion Automation

Connecting **Slack** and **Notion** to **n8n** to build three practical business automations: task notifications, client onboarding, and form-to-database registration.

---

## Objective

Learn the fundamentals of Slack and Notion as workspace tools, then wire them into n8n to automate real workflows — sending Slack notifications when work happens, creating Slack channels on demand, and turning form submissions into Notion database records.

---

## What was built

| # | Automation | Flow |
|---|-----------|------|
| 1 | **Task Notifications** | New task in Notion → Notion Trigger → Slack message to `#n8n-alerts` |
| 2 | **Client Onboarding** | New client in Notion → create a dedicated Slack channel → post a summary to `#client-updates` |
| 3 | **Form Registration** | Google Form → Google Sheets → Google Sheets Trigger → create a page in the Notion *Student Registrations* database |

All three workflows are **Published/Active** in n8n Cloud and were verified end-to-end (see screenshots).

---

## Slack setup

Workspace: **Zain AI Automation Lab**

Channels created for the internship, per the task brief:

- `#n8n-alerts` — automated workflow notifications
- `#client-updates` — new-client announcements
- `#internship` — internship communication
- `#workflow-errors` — error reporting
- `#project-status` — project tracking

Slack is connected to n8n. Because the community Slack node was limited, messages are sent with the **HTTP Request** node calling Slack's Web API directly:

- `chat.postMessage` — post a formatted message to a channel
- `conversations.create` — create a new channel during client onboarding

Authentication uses a Slack **Bot User OAuth token** (`xoxb-…`) passed in the `Authorization: Bearer` header, with scopes `chat:write` and `channels:manage`.

> **Security note:** the bot token in these exported JSON files has been replaced with the placeholder `xoxb-REPLACE_WITH_YOUR_SLACK_BOT_TOKEN`. Never commit a real Slack token to a public repository — Slack auto-revokes leaked tokens and anyone could use them to post to your workspace. Set your own token when importing.

---

## Notion setup

Page: **AI Automation Internship - Day 5**, containing three databases.

### Internship Tasks

| Property | Type |
|----------|------|
| Task Name | Title |
| Status | Status |
| Priority | Select |
| Due Date | Date |
| Assignee | Person |

Populated with sample tasks (e.g. *Build Notion & Slack Automation*, *Test Automatic Slack Notification*).

### Clients

| Property | Type |
|----------|------|
| Client Name | Title |
| Company | Text |
| Email | Email |
| Status | Status |

### Student Registrations

| Property | Type |
|----------|------|
| Name | Title |
| Email | Email |
| Course | Select |
| Status | Status |

Populated with 4 sample students (Ahmed Khan, Sara Ali, Zain Khan, Haseeb).

A full Notion export (Markdown + CSV per database) is in [`Notion/Export/`](Notion/Export/).

---

## Workflow details

### 1. Task Notifications — `Notion → Slack`

- **Notion Trigger** (`pageAddedToDatabase`) watches the *Internship Tasks* database, polling every minute.
- **HTTP Request** → `POST chat.postMessage` sends a formatted alert to `#n8n-alerts`, mapping the task's dynamic fields:

```
🚀 New Notion Task Created
📌 Task: {{ Task Name }}
📊 Status: {{ Status }}
⚡ Priority: {{ Priority }}
📅 Due Date: {{ Due Date }}
👤 Assignee: {{ Assignee }}
```

The `Due Date` and `Assignee` expressions fall back to `'Not set'` / `'Not assigned'` when empty, so the message never renders `undefined`.

### 2. Client Onboarding — `Notion → Slack channel → notify`

- **Notion Trigger** watches the *Clients* database.
- **HTTP Request** → `POST conversations.create` creates a channel named `client-<client-name>`, slugified so `TechNova Solutions` becomes `#client-technova-solutions`.
- **HTTP Request1** → `POST chat.postMessage` posts an onboarding summary to `#client-updates`, pulling the client's details back from the trigger node with `$('Notion Trigger').item.json[...]`.

### 3. Form Registration — `Google Form → Sheets → Notion`

- **Google Sheets Trigger** (`rowAdded`) fires when the linked form drops a new row into *Form Responses 1*.
- **Create a database page** (Notion node) creates a record in *Student Registrations*, mapping:

| Google Sheet field | Notion property |
|--------------------|-----------------|
| Name | Name (title) |
| Email | Email |
| Course | Course (select) |
| Status | Status |

---

## What I learned

- **Slack concepts** — workspaces, channels, threads, DMs, mentions, notifications, and app/bot integrations.
- **Notion concepts** — pages, blocks, databases, database property types (Title, Status, Select, Date, Person, Email), views, and integrations.
- **Slack authentication** — bot tokens, OAuth scopes, and calling the Slack Web API from n8n via the HTTP Request node when the native node falls short.
- **Selecting a channel and mapping dynamic data** — building message payloads that interpolate live fields from an upstream trigger, with sensible fallbacks for empty values.
- **Cross-node references** — using `$('Node Name').item.json` to reach back to a previous node's output after an intermediate step.

---

## Folder contents

```
Day-05/
├── n8n/
│   ├── Task Notifications - Notion to Slack.json
│   ├── Notion Client Onboarding - Slack Automation.json
│   ├── Google Form Registration - Notion.json
│   └── Screenshots/
│       ├── Task Notifications workflow.png
│       ├── Client Onboarding workflow.png
│       ├── Google Form to Notion workflow.png
│       └── Google Form to Notion execution.png
├── Slack/
│   └── Screenshots/
│       ├── n8n-alerts channel.png
│       └── client-updates channel.png
├── Notion/
│   ├── Export/            # Markdown + CSV export of all three databases
│   └── Screenshots/
│       ├── Internship Tasks and Clients databases.png
│       └── Student Registrations database.png
└── README.md
```

---

## Importing the workflows

1. In n8n: **Workflows → Import from File** → select any JSON in `Day-05/n8n/`.
2. In each **HTTP Request** node, replace `xoxb-REPLACE_WITH_YOUR_SLACK_BOT_TOKEN` with your own Slack bot token (create a Slack app, add `chat:write` and `channels:manage` scopes, install it, and copy the **Bot User OAuth Token**).
3. Reconnect the **Notion** and **Google Sheets** credentials to your own accounts.
4. Update the channel IDs (`C0…`), Notion data-source IDs, and the Google Sheet document ID to point at your own resources.
5. Click **Active** to publish.

---

**Zain Khan** — AI Automation Intern, MATalogics
