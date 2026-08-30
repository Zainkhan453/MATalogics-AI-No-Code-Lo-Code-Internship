# AI Operations Manager — Zapier

An autonomous, safety-controlled **AI Operations Manager** built with **Zapier Agents, Zapier Tables, Zapier Automations, Schedule by Zapier, and Slack**.

The system reviews business activity every morning across **Sales, Tasks, and Support**, identifies operational risks, prioritizes them, takes only approved low-risk internal actions, prevents duplicate actions, records executed actions, creates a structured Daily Operations Report, and delivers the report to an internal Slack channel.

---

## Project Objective

The goal of this project is not to build a simple:

> Schedule → Summarize Tables → Send Message

Instead, the system behaves like an internal operations manager:

> **Retrieve Data → Analyze → Detect Problems → Prioritize → Recommend → Permission Check → Execute Safe Actions → Prevent Duplicates → Log Actions → Generate Report → Notify Human**

The AI is intentionally given **limited autonomous authority**.

---

## Core Capabilities

### Sales

The AI Operations Manager can identify:

- inactive deals
- deals requiring follow-up
- stuck deals
- high-value opportunities requiring attention
- high-value stuck deals
- customer-level sales risk
- sales + support + task compound risk

### Tasks

The system can identify:

- overdue tasks
- unfinished tasks
- high employee workload
- critical workload
- high/critical overdue tasks
- workload bottlenecks affecting active deals

### Support

The system can identify:

- Critical unresolved tickets
- aging High-priority tickets
- escalation candidates
- resolved tickets that should be ignored
- support issues threatening active opportunities

### Autonomous Actions

Approved low-risk actions currently implemented:

- Create internal **Sales Follow-Up** tasks
- Update qualifying support tickets to **Escalation Required**
- Mark support updates as performed by the AI Operations Manager
- Log successful AI-created tasks automatically
- Log successful support escalation updates automatically

### Human-Controlled Actions

The Agent may recommend but must not autonomously execute:

- customer-facing communication
- task reassignment
- employee workload redistribution
- financial changes
- refunds
- destructive changes
- deal cancellation
- deal amount changes
- support-ticket closure/resolution
- deal Won/Lost changes

---

# System Architecture

```text
                         ┌─────────────────────────┐
                         │   Sales Pipeline Table  │
                         └────────────┬────────────┘
                                      │
                          Find Records│
                        Status = Active
                                      │
                                      ▼
┌──────────────────┐       ┌─────────────────────────┐
│ Operations Tasks │──────▶│                         │
│ Knowledge Source │       │  AI Operations Manager  │
└──────────────────┘       │      Zapier Agent       │
                           │                         │
┌──────────────────┐       └──────┬──────────┬───────┘
│ Support Tickets  │─────────────▶│          │
│ Knowledge Source │              │          │
└──────────────────┘              │          │
                                  │          │
                     Create Sales │          │ Update qualifying
                     Follow-Up    │          │ support escalation
                                  ▼          ▼
                         Operations Tasks   Support Tickets
                                  │          │
                                  ▼          ▼
                         Task Action      Support Action
                            Logger           Logger
                                  │          │
                                  └────┬─────┘
                                       ▼
                              AI Operations Actions

Schedule by Zapier
      │
      ▼
Run AI Operations Manager
      │
      ▼
Structured Report Output
      │
      ├────────▶ Daily Operations Reports
      │
      └────────▶ Slack Operations Channel
```

---

# Technology Stack

| Component | Purpose |
|---|---|
| Zapier Agents | Autonomous business analysis and decision-making |
| Zapier Tables | Operational data storage |
| Zapier Automations / Zaps | Scheduling, logging, report storage and delivery |
| Schedule by Zapier | Daily 8:00 AM trigger |
| Filter by Zapier | Ensures only valid AI actions are logged |
| Slack | Internal Daily Operations Report delivery |

---

# Zapier Tables

The system uses five core tables.

## 1. Sales Pipeline

Stores sales opportunities.

| Field | Purpose |
|---|---|
| Deal ID | Unique business-friendly identifier |
| Customer | Customer/company |
| Deal | Opportunity name |
| Amount | Deal value |
| Stage | Sales stage |
| Owner | Deal owner |
| Last Activity Date | Detect inactivity |
| Last Stage Change | Detect stuck deals |
| Follow-Up Date | Optional scheduled follow-up |
| Priority | Internal priority |
| Status | Active / Won / Lost / On Hold |
| Created At | Creation timestamp |
| Updated At | Last record update |

### Stage Values

- New Lead
- Qualified
- Proposal
- Negotiation
- Won
- Lost

### Status Values

- Active
- Won
- Lost
- On Hold

---

## 2. Operations Tasks

Stores human and AI-created internal tasks.

| Field | Purpose |
|---|---|
| Task ID | Unique task identifier |
| Task | Task title |
| Owner | Responsible employee |
| Deadline | Due date |
| Status | Current task state |
| Related Customer | Related customer |
| Related Deal | Related Deal ID |
| Task Type | Type of operational task |
| Priority | Task priority |
| Created By | Human / AI Operations Manager / Automation |
| Created At | Creation timestamp |
| Completed At | Completion timestamp |

### Task Types

- General
- Sales Follow-Up
- Support Escalation
- Internal Review
- Operations

### Status Values

- Pending
- In Progress
- Completed
- Blocked

---

## 3. Support Tickets

Stores customer support activity.

| Field | Purpose |
|---|---|
| Ticket ID | Unique ticket identifier |
| Customer | Related customer |
| Issue | Support problem |
| Priority | Low / Medium / High / Critical |
| Status | Current ticket state |
| Assigned Agent | Support owner |
| Created At | Ticket creation |
| Last Updated | Latest activity |
| Escalation Status | Internal escalation state |
| Resolution Date | Resolution timestamp |
| Last Updated By | Human / AI Operations Manager / Automation |

### Escalation Status Values

- Not Required
- Recommended
- Escalation Required
- Escalated

---

## 4. AI Operations Actions

Audit table for successful AI operational actions.

| Field | Purpose |
|---|---|
| Action ID | Unique action identifier |
| Action Date | Business date |
| Source Type | Sales / Task / Support / Cross-Functional |
| Source Record ID | Related Deal/Task/Ticket ID |
| Customer | Related customer |
| Problem | Detected operational problem |
| Severity | Operational severity |
| Recommended Action | Recommended response |
| Action Type | Standardized action |
| Action Taken | Actual successful action |
| Requires Approval | Human approval flag |
| Approval Status | Approval state |
| Result | Action outcome |
| Action Status | Executed / Failed / etc. |
| Created At | Audit timestamp |

This table is populated by dedicated logging Zaps rather than directly by the Agent.

---

## 5. Daily Operations Reports

Stores each structured Daily Operations Report.

| Field | Purpose |
|---|---|
| Report ID | Daily report identifier |
| Report Date | Business date |
| Overall Status | Normal / Attention Required / Critical |
| High Priority Count | Number of HIGH findings |
| Medium Priority Count | Number of MEDIUM findings |
| Low Priority Count | Number of LOW findings |
| Executive Summary | Management overview |
| High Priority Issues | Consolidated HIGH findings |
| Medium Priority Issues | Consolidated MEDIUM findings |
| Low Priority Issues | Consolidated LOW findings |
| Recommended Actions | AI recommendations |
| Actions Taken | Successfully executed actions from that run |
| Approval Required | Human-controlled actions |
| Generated At | Report creation timestamp |

---

# Business Rules

## Sales Rules

### High-Value Deal

```text
Amount >= 10,000
```

### Follow-Up Needed

```text
Status = Active
AND
No meaningful activity for at least 3 days
```

### Stuck Deal

```text
Status = Active
AND
No stage movement for at least 5 days
```

### Critical High-Value Stuck Deal

```text
Amount >= 10,000
AND
Status = Active
AND
No stage movement for at least 5 days
```

---

## Task Rules

### Overdue Task

```text
Deadline < Current Business Date
AND
Status != Completed
```

### High Workload

```text
More than 5 unfinished tasks assigned to one owner
```

### Critical Workload

```text
At least 5 overdue tasks assigned to one owner
```

---

## Support Rules

### Critical Unresolved Ticket

```text
Priority = Critical
AND
Status != Resolved
AND
Status != Closed
```

### Critical Escalation Candidate

```text
Priority = Critical
AND
Status != Resolved
AND
Status != Closed
AND
Unresolved for more than 1 day
```

### High-Priority Escalation Candidate

```text
Priority = High
AND
Status != Resolved
AND
Status != Closed
AND
Unresolved for more than 2 days
```

No escalation threshold is automatically invented for Medium or Low priority tickets.

---

# Priority Model

## HIGH

Examples:

- Critical unresolved support issue
- Critical escalation candidate
- high-value seriously stuck deal
- critical employee workload
- Critical overdue task
- severe cross-functional customer risk
- major revenue or retention threat

## MEDIUM

Examples:

- overdue sales follow-up
- aging High-priority support ticket
- several overdue/pending tasks
- medium-value inactive opportunity
- combined risk that requires attention but is not immediately critical

## LOW

Examples:

- upcoming deadline
- Medium support ticket opened recently
- minor operational warning
- monitoring item

---

# AI Operations Manager Agent

## Data Access

### Sales

The Agent uses:

**Zapier Tables → Find Records**

Configuration:

```text
Table: Sales Pipeline
Filter Count: 1
Lookup Field: Status
Lookup Value: Active
```

The Sales Pipeline table is the authoritative source for:

- Amount
- Stage
- Status
- Last Activity Date
- Last Stage Change
- Owner
- Deal ID

The Agent must not infer these values from Tasks or Support.

### Tasks

`Operations Tasks` is connected as a **Knowledge Source**.

### Support

`Support Tickets` is connected as a **Knowledge Source**.

---

# Agent Write Tools

## Create Record — Sales Follow-Up

Target:

`Operations Tasks`

Fixed values include:

```text
Status = Pending
Task Type = Sales Follow-Up
Created By = AI Operations Manager
```

The Agent must check for an existing unfinished Sales Follow-Up for the same Deal ID before creating another one.

---

## Update Record — Support Escalation

Target:

`Support Tickets`

Permitted update:

```text
Escalation Status = Escalation Required
Last Updated By = AI Operations Manager
```

The Agent does not:

- resolve the ticket
- close the ticket
- change Priority
- send customer messages

If the ticket is already `Escalation Required` or `Escalated`, no repeat update is performed.

---

# Duplicate Prevention

Duplicate prevention is implemented at the Agent decision layer.

## Sales Follow-Up

Before creating a task:

```text
Task Type = Sales Follow-Up
AND
Related Deal = same Deal ID
AND
Status != Completed
```

If an unfinished task already exists:

```text
Existing action already pending.
```

No duplicate task is created.

## Support Escalation

If:

```text
Escalation Status = Escalation Required
```

or:

```text
Escalation Status = Escalated
```

the Agent does not perform the update again.

---

# Zap 1 — AI Operations — Task Action Logger

## Purpose

Automatically logs AI-created Sales Follow-Up tasks.

## Workflow

```text
Zapier Tables — New Record
Operations Tasks

        ↓

Filter by Zapier
Created By = AI Operations Manager

        ↓

Zapier Tables — Create Record
AI Operations Actions
```

### Example Logged Action

```text
Action ID: ACT-TASK-010
Source Type: Sales
Source Record ID: DEAL-002
Customer: Bright Solutions
Action Type: Create Follow-Up Task
Action Status: Executed
Requires Approval: false
Approval Status: Not Required
```

---

# Zap 2 — AI Operations — Support Action Logger

## Purpose

Automatically logs valid AI support escalation updates.

## Workflow

```text
Zapier Tables — Updated Record
Support Tickets
Trigger Field: Escalation Status

        ↓

Filter by Zapier

Last Updated By = AI Operations Manager
AND
Escalation Status = Escalation Required
AND
Status != Resolved
AND
Status != Closed

        ↓

Zapier Tables — Create Record
AI Operations Actions
```

### Example Logged Action

```text
Action ID: ACT-ESC-TICKET-003
Source Type: Support
Source Record ID: TICKET-003
Customer: Nova Enterprises
Action Type: Update Escalation Status
Action Taken: Updated TICKET-003 Escalation Status to Escalation Required.
Action Status: Executed
Requires Approval: false
Approval Status: Not Required
```

---

# Zap 3 — AI Operations — Daily Operations Manager

The main daily automation.

## Workflow

```text
1. Schedule by Zapier
   Every Day
   8:00 AM
   Asia/Karachi

        ↓

2. Zapier Agents
   Run Agent
   AI Operations Manager

        ↓

3. Zapier Tables
   Create Record
   Daily Operations Reports

        ↓

4. Slack
   Send Channel Message
   Internal Operations Channel
```

---

# Structured Agent Output

The Daily Operations Zap receives these structured outputs:

```text
report_date
overall_status
high_priority_count
medium_priority_count
low_priority_count
executive_summary
high_priority_issues
medium_priority_issues
low_priority_issues
recommended_actions
actions_taken
approval_required
```

The Zap maps those outputs directly into the `Daily Operations Reports` table and Slack message.

---

# Slack Report Format

Example structure:

```text
🤖 DAILY OPERATIONS REPORT

📅 Report Date: YYYY-MM-DD
📊 Overall Status: Critical

━━━━━━━━━━━━━━━━━━
EXECUTIVE SUMMARY
━━━━━━━━━━━━━━━━━━

Management-level operational summary.

━━━━━━━━━━━━━━━━━━
🔴 HIGH PRIORITY (N)
━━━━━━━━━━━━━━━━━━

• High-priority finding

━━━━━━━━━━━━━━━━━━
🟡 MEDIUM PRIORITY (N)
━━━━━━━━━━━━━━━━━━

• Medium-priority finding

━━━━━━━━━━━━━━━━━━
🟢 LOW PRIORITY (N)
━━━━━━━━━━━━━━━━━━

• Low-priority finding

━━━━━━━━━━━━━━━━━━
📌 RECOMMENDED ACTIONS
━━━━━━━━━━━━━━━━━━

• Recommended action

━━━━━━━━━━━━━━━━━━
⚙️ ACTIONS AUTOMATICALLY TAKEN
━━━━━━━━━━━━━━━━━━

• Successfully executed action
or
None

━━━━━━━━━━━━━━━━━━
👤 APPROVAL / HUMAN ACTION REQUIRED
━━━━━━━━━━━━━━━━━━

• Human-controlled action
```

Structured Agent outputs use plain text with the Unicode bullet `•` so Slack does not display escaped Markdown such as `\-` or `\*\*`.

---

# Key Test Data

## Sales

| Deal | Customer | Amount | Stage | Expected Behavior |
|---|---|---:|---|---|
| DEAL-001 | Alpha Technologies | $4,500 | Qualified | Normal/recent activity |
| DEAL-002 | Bright Solutions | $7,000 | Proposal | Follow-up / combined risk |
| DEAL-003 | Nova Enterprises | $15,000 | Negotiation | High-value stuck risk |
| DEAL-004 | Vertex Systems | $12,000 | Won | Excluded from active-risk analysis |
| DEAL-005 | Orion Systems | $18,000 | Proposal | High-value stuck test case |

---

## Tasks

Important validated cases included:

- Completed task with past deadline → **not overdue**
- In Progress task with past deadline → **overdue**
- Critical overdue task → HIGH attention
- Ahmad with 7 overdue tasks → **Critical workload**
- AI-created Sales Follow-Up task → logged automatically
- Existing follow-up for same deal → duplicate creation prevented

---

## Support

| Ticket | Priority | Status | Expected |
|---|---|---|---|
| TICKET-001 | Medium | Open | Monitor only |
| TICKET-002 | High | Open | Escalation candidate |
| TICKET-003 | Critical | Open | Critical escalation candidate |
| TICKET-004 | High | Resolved | Ignore as active escalation risk |
| TICKET-005 | Critical | Open | Fresh escalation/cross-functional test |

---

# Validated Test Scenarios

## Test 1 — Normal Active Deal

Expected:

- No unnecessary Sales Follow-Up
- No HIGH alert

**Passed**

---

## Test 2 — Stuck High-Value Deal

Example:

`DEAL-003`

Expected:

- HIGH operational finding
- internal follow-up recommended
- no customer email sent autonomously

**Passed**

---

## Test 3 — Follow-Up Duplicate Prevention

Existing unfinished Sales Follow-Up for same Deal ID.

Expected:

```text
Existing action already pending.
```

No duplicate task created.

**Passed**

---

## Test 4 — AI Follow-Up Creation

`DEAL-002` initially had no matching unfinished Sales Follow-Up.

Agent created:

`TASK-010`

Expected:

- Pending
- Sales Follow-Up
- AI Operations Manager
- Related Deal = DEAL-002

**Passed**

---

## Test 5 — Task Action Logging

`TASK-010` triggered:

**AI Operations — Task Action Logger**

Result:

`ACT-TASK-010`

created in `AI Operations Actions`.

**Passed**

---

## Test 6 — Critical Workload

Ahmad:

- 8 unfinished tasks
- 7 overdue tasks

Expected:

- Critical workload
- HIGH operational finding
- workload redistribution recommendation
- no autonomous reassignment

**Passed**

---

## Test 7 — High Support Escalation

`TICKET-002`

Expected:

- High ticket older than 2 days
- Escalation candidate
- Escalation Status updated safely when applicable

**Passed**

---

## Test 8 — Critical Support Escalation

`TICKET-003`

Expected:

- Critical unresolved >1 day
- HIGH operational concern
- `Escalation Status = Escalation Required`

**Passed**

---

## Test 9 — Resolved Ticket Safety

`TICKET-004`

Expected:

- No active escalation
- No closure/resolution changes
- excluded from active-risk analysis

**Passed after logger safeguards were added**

---

## Test 10 — Support Action Logging

Valid escalation update for:

`TICKET-003`

created:

`ACT-ESC-TICKET-003`

in `AI Operations Actions`.

**Passed**

---

## Test 11 — Cross-Functional Risk

Nova Enterprises:

- $15,000 high-value stuck deal
- Critical unresolved production support issue
- multiple overdue related tasks
- overloaded owner

Expected:

- consolidated HIGH risk
- management intervention recommended

**Passed**

---

## Test 12 — Daily Report Generation

Expected:

- structured report output
- report stored in `Daily Operations Reports`
- counts populated
- recommendations separated from actions taken
- approval-required section populated

**Passed**

---

## Test 13 — Slack Delivery

Expected:

- Daily report automatically delivered
- HIGH / MEDIUM / LOW sections visible
- clean Slack formatting
- no escaped Markdown artifacts after formatting fix

**Passed**

---

## Test 14 — Repeated Run / Idempotency

Expected:

- existing Sales Follow-Up tasks are not recreated
- already escalated tickets are not updated again
- `Actions Taken = None` when no new safe action is required

**Passed**

---

# Safety Guardrails

The AI Operations Manager must never autonomously:

- delete records
- cancel deals
- change deal amounts
- modify financial information
- alter payment information
- issue refunds
- send external customer communications
- close support tickets
- mark support tickets Resolved
- mark deals Won or Lost
- reassign employee workload without human approval
- perform destructive operations

Restricted actions are added to:

**Approval / Human Action Required**

instead of pausing or executing them.

---

# Error Handling

The Agent is instructed to:

1. use authoritative sources only
2. retry Sales retrieval once if it fails
3. never invent unavailable data
4. never infer Sales fields from Tasks or Support
5. never claim tool execution if a tool fails
6. continue analyzing other available domains
7. complete the automated Zap run without asking for manual completion

---

# Project Outcome

The final system demonstrates:

- autonomous operational analysis
- cross-functional AI reasoning
- exception detection
- business-rule enforcement
- prioritization
- safe autonomous internal actions
- controlled authority
- duplicate prevention
- automatic action logging
- human-approval separation
- structured daily reporting
- automated Slack delivery

This makes the project a genuine **AI Operations Manager**, rather than a scheduled summarization workflow.

---

# Final Workflow Summary

```text
Every Day at 8:00 AM

        ↓

AI Operations Manager

        ↓

Analyze:
Sales + Tasks + Support

        ↓

Identify:
Risks + Bottlenecks + Escalations

        ↓

Prioritize:
HIGH / MEDIUM / LOW

        ↓

Permission Check

        ├── Safe Action
        │      ↓
        │   Execute
        │      ↓
        │   Logger Zap
        │      ↓
        │   AI Operations Actions
        │
        └── Restricted Action
               ↓
           Human Approval Required

        ↓

Generate Structured Daily Report

        ↓

Daily Operations Reports

        ↓

Slack Internal Operations Channel
```

---

## Status

**Project Completed — End-to-End Workflow Implemented and Tested**
