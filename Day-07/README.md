# Day 7 — AI Client Onboarding System (End-to-End Automation)

A single, real end-to-end business automation: a client **calls a Vapi voice agent**, and the system automatically understands the conversation, classifies the request with AI, stores the client, creates an onboarding page, and alerts the team — across **Vapi → n8n → OpenAI → Airtable → Notion → Slack**.

---

## Scenario

A company wants an AI-powered system that handles a new client's initial onboarding. A client calls the voice agent and explains what they need; the system understands the conversation, stores the client, creates an onboarding task, and notifies the team. **One simple workflow — but every tool has a real purpose.**

---

## Architecture

```
              Client
                │  (phone call)
                ▼
        Vapi Voice Agent          ← collects 6 fields in English / Urdu / Roman Urdu
                │  (POST end-of-call)
                ▼
          n8n Webhook
                │
                ▼
   AI Agent (OpenAI) — classify   → service_category, summary, next_action, normalized fields
                │
                ▼
   Calculate Lead Priority (Code) → HIGH / MEDIUM / LOW from budget
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
     Airtable         Notion           Slack
  Client record   Client Projects   #new-clients
                     page            priority alert
```

---

## The n8n workflow

`n8n/AI Client Onboarding System.json` — 8 functional nodes (verified, one run **Succeeded in 7.235s**):

| # | Node | Role |
|---|------|------|
| 1 | **Webhook** | Receives the end-of-call payload from Vapi at `/webhook/client-onboarding` |
| 2 | **Prepare Vapi Call Data** (Set) | Extracts the transcript/fields from the Vapi message body |
| 3 | **AI Agent** (+ OpenAI Chat Model) | Extracts, normalizes and **translates** the client details into professional English and returns structured JSON |
| 4 | **Calculate Lead Priority** (Code) | Deterministically maps budget → priority and sets `status: "New"` |
| 5 | **Create Client Record** (Airtable) | Writes the row to the *Client Onboarding* table |
| 6 | **Create a database page** (Notion) | Creates the *Client Projects* onboarding page |
| 7 | **Prepare Slack Message** (Code) | Builds the priority-based message text |
| 8 | **Send a message** (Slack) | Posts the alert to `#new-clients` |

> The workflow JSON has had its pinned test-call data (a full call transcript, a client email, IPs and recording URLs) stripped, so it imports clean and carries no personal data. Credentials are referenced by ID only — no tokens are included.

### AI classification

The AI Agent is prompted as a **bilingual (English/Urdu/Roman Urdu) client-intake processor**. It normalizes messy spoken input into clean English and returns:

```json
{
  "service_category": "Web & Mobile Development",
  "priority": "High",
  "summary": "Restaurant requires website and mobile ordering application.",
  "next_action": "Schedule technical consultation"
}
```

This is why the Urdu-script caller (احمد رضا / ارین کرافٹ) was correctly stored as **Ahmad Raza — Urban Craft** in English.

### Priority logic

Priority is **not** left to the LLM — it's computed deterministically in the *Calculate Lead Priority* Code node, exactly per the brief:

| Budget (PKR) | Priority |
|--------------|----------|
| ≥ 500,000 | **HIGH** |
| 200,000 – 499,999 | **MEDIUM** |
| < 200,000 | **LOW** |

---

## The One Extra Feature — priority-based Slack messages

The Slack alert changes headline and tone with the computed priority:

| Priority | Slack headline |
|----------|----------------|
| HIGH | 🔥 **HIGH PRIORITY CLIENT** |
| MEDIUM | ⚠️ **MEDIUM PRIORITY CLIENT** |
| LOW | ✅ **STANDARD CLIENT** |

---

## Required test — 3 calls, verified end-to-end

All three required test budgets were called through the agent and flowed Vapi → n8n → AI → Airtable → Notion → Slack:

| Client | Company | Budget (PKR) | Expected | Result |
|--------|---------|--------------|----------|--------|
| Ali Ahmed | Monal Restaurant | 600,000 | HIGH | ✅ HIGH |
| Sara Khan | BrightEdge Solutions | 300,000 | MEDIUM | ✅ MEDIUM |
| Ahmad Raza | Urban Craft | 100,000 | LOW | ✅ LOW (STANDARD) |

(A fourth HIGH call — Zain Khan / Nova Solutions, 600,000 — was also captured.)

Proof of the three data stores, with matching records, is in the screenshot folders:

- **Airtable** — `Airtable/Screenshots/` — the *Client Onboarding* table with all clients, budgets, AI category, priority and `Status = New`.
- **Notion** — `Notion/Screenshots/` — the *Client Projects* database with per-client pages (Priority, Summary, Next Action).
- **Slack** — `Slack/Screenshots/` — the four `#new-clients` notifications, one per priority tier.

---

## Deliverables (per the brief)

| Deliverable | Location |
|-------------|----------|
| n8n workflow JSON | [`n8n/AI Client Onboarding System.json`](n8n/AI%20Client%20Onboarding%20System.json) |
| Vapi agent configuration | [`Vapi/Vapi Agent Configuration.md`](Vapi/Vapi%20Agent%20Configuration.md) + screenshot |
| Airtable / Notion / Slack screenshots (3 clients, pages, notifications) | `Airtable/`, `Notion/`, `Slack/` Screenshots |

---

## Folder contents

```
Day-07/
├── n8n/
│   ├── AI Client Onboarding System.json   # clean, importable (pinData stripped)
│   └── Screenshots/                        # workflow execution (succeeded)
├── Vapi/
│   ├── Vapi Agent Configuration.md
│   └── Screenshots/                        # agent system prompt
├── Airtable/Screenshots/                   # Client Onboarding table (2 views)
├── Notion/Screenshots/                     # Client Projects database (2 views)
├── Slack/Screenshots/                      # 4 priority-based notifications
└── README.md
```

---

## Importing / reproducing

1. In n8n: **Workflows → Import from File** → `n8n/AI Client Onboarding System.json`.
2. Reconnect the **OpenAI**, **Airtable**, **Notion** and **Slack** credentials to your own accounts.
3. Point the Airtable node at your *Client Onboarding* table and the Notion node at your *Client Projects* database.
4. Copy the Webhook **Production URL** and set it as the Vapi assistant's end-of-call webhook.
5. Recommended hardening: enable Vapi's server secret and verify the `x-vapi-secret` header in n8n, so only genuine Vapi calls can create records (the webhook is otherwise an open write endpoint into Airtable/Notion/Slack and OpenAI).

---

**Zain Khan** — AI Automation Intern, MATalogics
