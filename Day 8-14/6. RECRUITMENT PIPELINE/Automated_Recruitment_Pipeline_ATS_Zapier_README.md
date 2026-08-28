# Automated Recruitment Pipeline & ATS in Zapier

An end-to-end, event-driven **Recruitment Management System (RMS) / Applicant Tracking System (ATS)** built entirely within the Zapier ecosystem using **Zapier Interfaces, Zapier Tables, Zapier Kanban, and Zapier Automations (Zaps)**.

The system automates candidate intake, experience-based priority scoring, Kanban stage management, stage-triggered email communication, duplicate-email prevention, and recruiter inactivity reminders.

---

## 📌 Project Overview

This project was designed for a company hiring software developers and provides a complete recruitment workflow from the moment a candidate submits an application until they are either **Hired** or **Rejected**.

### Core Capabilities

- Public developer application form
- Centralized candidate database
- Automatic candidate priority scoring
- Kanban-based recruitment pipeline
- Stage-based candidate and HR email notifications
- Duplicate-email and Zap loop prevention
- Recruiter inactivity monitoring
- Delayed state re-validation before reminders
- Fully native Zapier implementation with no external third-party services

---

## 🛠️ Tech Stack

| Component | Purpose |
|---|---|
| **Zapier Interfaces / Forms** | Public-facing Developer Application Form |
| **Zapier Tables** | Central candidate database and state tracking |
| **Zapier Kanban** | Interactive recruiter pipeline dashboard |
| **Zapier Paths** | Priority-based and stage-based branching |
| **Zapier Filters** | State validation and automation guards |
| **Delay by Zapier** | Recruiter inactivity waiting period |
| **Email by Zapier** | Candidate, HR, and recruiter notifications |
| **Zapier Automations (Zaps)** | End-to-end workflow orchestration |

---

## 🏗️ System Architecture

The solution is divided into three main automations:

1. **Process New Candidate Application**
   - Captures application data
   - Creates the candidate record
   - Generates a Candidate ID
   - Evaluates experience
   - Assigns priority
   - Places the candidate in the `Applied` stage

2. **Kanban Stage Change Actions**
   - Detects recruitment-stage changes
   - Sends the correct stage-based email
   - Updates stage tracking fields
   - Prevents duplicate notifications and trigger loops

3. **5-Day Recruiter Inactivity Reminder**
   - Watches active candidate records
   - Waits five days after the latest update
   - Re-fetches the candidate's current state
   - Sends a reminder only if the candidate is still inactive

---

# 📊 Database Design

## Zapier Table: `Recruitment Candidates`

| Column Name | Field Type | Purpose / Function | Source | Default / Initial Value |
|---|---|---|---|---|
| Candidate ID | Text | Unique candidate identifier | Automation | Generated on creation |
| Candidate Name | Text | Full applicant name | Candidate Form | Blank |
| Email | Email Address | Primary contact email | Candidate Form | Blank |
| Phone | Text | Candidate phone number | Candidate Form | Blank |
| Position | Text | Role applied for | Candidate Form | Blank |
| Experience (Years) | Number | Years of software-development experience | Candidate Form | Blank |
| Expected Salary | Text | Candidate salary expectation | Candidate Form | Blank |
| Resume | Link | Hosted candidate CV / resume | Candidate Form | Blank |
| Portfolio | Link | Portfolio, GitHub, or personal website | Candidate Form | Blank |
| Availability | Text | Start availability / notice period | Candidate Form | Blank |
| Priority | Dropdown | Candidate priority: High, Medium, or Low | Automation | Calculated |
| Recruitment Stage | Dropdown | Current Kanban pipeline stage | Automation / Recruiter | Applied |
| Application Date | Date/Time | Initial submission timestamp | Automation | Zap submission time |
| Last Updated Date | Date/Time | Latest recruitment-stage transition timestamp | Automation | Zap execution time |
| Last Emailed Stage | Text | Guard field preventing duplicate stage emails | Automation | Blank |
| Recruiter Notes | Long Text | Recruiter feedback and evaluation notes | Recruiter | Blank |

---

# 📋 Developer Application Form

The public-facing application form was created with **Zapier Interfaces**.

## Form Fields

| Field | Type | Required |
|---|---|---|
| Candidate Name | Short Text | Yes |
| Email | Email | Yes |
| Phone | Short Text | Yes |
| Position | Dropdown | Yes |
| Experience | Number | Yes |
| Expected Salary | Short Text | Yes |
| Resume | File Upload / Link | Yes |
| Portfolio | URL / Link | No |
| Availability | Dropdown | Yes |

### Position Options

- Full Stack Developer
- Backend Developer
- Frontend Developer
- DevOps Engineer

### Availability Options

- Immediate
- 2 Weeks Notice
- 1 Month Notice
- Negotiable

---

# 🗂️ Kanban Board Configuration

The recruiter dashboard is powered by **Zapier Kanban** and connected directly to the `Recruitment Candidates` table.

### Data Source

`Recruitment Candidates`

### Group By

`Recruitment Stage`

### Pipeline Stages

```text
Applied
  ↓
Screening
  ↓
Technical Interview
  ↓
HR Interview
  ↓
Offer
  ↓
Hired

Rejected can be reached from any appropriate stage.
```

### Card Information

Each Kanban card displays:

- Candidate Name
- Position
- Experience (Years)
- Priority

This gives recruiters a quick overview while allowing drag-and-drop candidate progression.

---

# ⚡ Automation 1 — Process New Candidate Application

## Trigger

**Zapier Interfaces → New Submission**

Triggered whenever a candidate submits the Developer Application Form.

## Workflow

```text
Candidate Form Submission
        ↓
Create Candidate Record
        ↓
Generate Candidate ID
        ↓
Evaluate Experience
        ↓
Assign Priority
        ↓
Set Recruitment Stage = Applied
        ↓
Store Application Date
        ↓
Store Last Updated Date
```

## Priority Scoring Logic

| Experience | Assigned Priority |
|---|---|
| 5+ years | High |
| 2 to less than 5 years | Medium |
| Less than 2 years | Low |

### Path A — High Priority

Condition:

```text
Experience >= 5
```

Action:

```text
Priority = High
```

### Path B — Medium Priority

Conditions:

```text
Experience >= 2
AND
Experience < 5
```

Action:

```text
Priority = Medium
```

### Path C — Low Priority

Condition:

```text
Experience < 2
```

Action:

```text
Priority = Low
```

## Candidate ID Format

Each record receives a unique ID using the submission identifier.

```text
CAND-2026-[Submission ID]
```

Example:

```text
CAND-2026-cmtd368hl0007pxqlvia9wycx
```

## Initial Record Values

```text
Recruitment Stage = Applied
Application Date = Current Zap execution timestamp
Last Updated Date = Current Zap execution timestamp
Last Emailed Stage = Blank
```

---

# ⚡ Automation 2 — Kanban Stage Change Actions

## Trigger

**Zapier Tables → Updated Record**

This Zap runs whenever a candidate record changes.

Because stage actions also update the candidate record, a protection mechanism is required to prevent recursive execution and duplicate emails.

---

## 🛡️ Duplicate Email & Loop Prevention

The field:

```text
Last Emailed Stage
```

acts as the system's state guard.

Before executing any stage-specific email, the Zap checks:

```text
Recruitment Stage != Last Emailed Stage
```

If both values are identical, the Zap stops.

This prevents:

- Duplicate candidate emails
- Infinite Zap trigger loops
- Emails firing when only Recruiter Notes are edited
- Repeated messages when unrelated fields are changed

---

## Stage-Based Actions

### Path A — Screening

Condition:

```text
Recruitment Stage = Screening
```

Actions:

1. Send candidate screening email.
2. Set:

```text
Last Emailed Stage = Screening
```

3. Update:

```text
Last Updated Date = Current timestamp
```

---

### Path B — Technical Interview

Condition:

```text
Recruitment Stage = Technical Interview
```

Actions:

1. Send technical interview scheduling email.
2. Update:

```text
Last Emailed Stage = Technical Interview
Last Updated Date = Current timestamp
```

---

### Path C — HR Interview

Condition:

```text
Recruitment Stage = HR Interview
```

Actions:

1. Send an internal HR briefing containing:
   - Candidate Name
   - Position
   - Experience
   - Expected Salary
   - Resume
   - Portfolio
   - Availability

2. Update:

```text
Last Emailed Stage = HR Interview
Last Updated Date = Current timestamp
```

---

### Path D — Hired

Condition:

```text
Recruitment Stage = Hired
```

Actions:

1. Send congratulations / hiring confirmation email.
2. Update:

```text
Last Emailed Stage = Hired
Last Updated Date = Current timestamp
```

---

### Path E — Rejected

Condition:

```text
Recruitment Stage = Rejected
```

Actions:

1. Send a professional rejection email.
2. Update:

```text
Last Emailed Stage = Rejected
Last Updated Date = Current timestamp
```

---

## Offer Stage

The `Offer` stage is available in the Kanban pipeline for recruiter workflow management.

In this implementation, the formal hiring / offer confirmation email is triggered when the candidate reaches the `Hired` stage.

---

# ⚡ Automation 3 — 5-Day Recruiter Inactivity Reminder

This automation prevents candidates from remaining unattended in the recruitment pipeline.

## Trigger

**Zapier Tables → Updated Record**

---

## Step 1 — Ignore Closed Candidates

The Zap filters out records where:

```text
Recruitment Stage = Hired
OR
Recruitment Stage = Rejected
```

Only active candidates continue.

---

## Step 2 — Delay

Production configuration:

```text
Delay For = 5 Days
```

Testing configuration:

```text
Delay For = 5 Minutes
```

---

## Step 3 — Re-Fetch Candidate State

After the delay, the Zap performs a fresh lookup of the candidate record using its Zapier Table Record ID.

This is important because the candidate may have progressed to a new stage while the delayed Zap execution was waiting.

---

## Step 4 — State Validation

The automation compares:

```text
Fresh Record → Last Updated Date
```

with:

```text
Original Trigger → Last Updated Date
```

The reminder is allowed to continue only when:

```text
Fresh Last Updated Date
=
Original Last Updated Date
```

and:

```text
Recruitment Stage != Hired
AND
Recruitment Stage != Rejected
```

---

## Step 5 — Recruiter Alert

If the state is unchanged after five days, an internal recruiter inactivity email is sent.

This design prevents outdated delayed Zap executions from generating false reminders.

---

# 🔄 Inactivity Reminder Logic

```text
Candidate Record Updated
        ↓
Is Candidate Active?
        ↓ Yes
Wait 5 Days
        ↓
Fetch Fresh Candidate Record
        ↓
Compare Last Updated Date
        ↓
Has Record Changed?
   ┌─────────────┴─────────────┐
   │                           │
 Yes                          No
   │                           │
Stop Automation      Candidate Still Active?
                                ↓
                              Yes
                                ↓
                     Send Recruiter Reminder
```

---

# 📧 Automated Email Types

The system includes the following email workflows:

### Screening Email

Sent when the candidate moves to:

```text
Screening
```

Purpose:

- Confirms that the application is under review
- Keeps the candidate informed about progress

---

### Technical Interview Invitation

Sent when the candidate moves to:

```text
Technical Interview
```

Purpose:

- Invites the candidate to the technical-assessment stage
- Provides scheduling instructions

---

### HR Briefing Email

Sent internally when the candidate reaches:

```text
HR Interview
```

Includes:

- Candidate profile
- Position
- Experience
- Expected salary
- CV / resume
- Portfolio
- Availability

---

### Hired / Offer Confirmation Email

Sent when the candidate moves to:

```text
Hired
```

Purpose:

- Congratulates the candidate
- Confirms selection
- Introduces next onboarding steps

---

### Rejection Email

Sent when the candidate moves to:

```text
Rejected
```

Purpose:

- Professionally informs the applicant
- Maintains a respectful candidate experience

---

### Recruiter Inactivity Alert

Sent internally when an active candidate remains unchanged for five days.

Purpose:

- Prevent candidate stagnation
- Improve recruitment response time
- Reduce missed follow-ups

---

# 🧪 Testing & Verification

All major workflow branches and safeguards were tested using live records.

---

## Test Case 1 — High Priority Candidate / Full Pipeline

### Candidate Data

| Field | Value |
|---|---|
| Candidate Name | Zain Khan Baloch |
| Email | zaynbaloch89@gmail.com |
| Phone | 03088320756 |
| Position | Full Stack Developer |
| Experience | 7 years |
| Expected Salary | $135,000/year |
| Availability | Immediate |
| Portfolio | zainkhanbaloch.tech |

### Expected Priority

```text
High
```

### Expected Pipeline

```text
Applied
→ Screening
→ Technical Interview
→ HR Interview
→ Offer
→ Hired
```

### Verification

- ✅ Priority correctly evaluated as **High**
- ✅ Candidate record created successfully
- ✅ Candidate ID generated successfully:

```text
CAND-2026-cmtd368hl0007pxqlvia9wycx
```

- ✅ Screening email received
- ✅ Technical Interview invitation received
- ✅ HR notification received
- ✅ Hiring / offer email received
- ✅ `Last Emailed Stage` updated after every applicable stage transition
- ✅ No duplicate emails were generated

---

## Test Case 2 — Medium Priority / Rejection Flow

### Candidate Data

| Field | Value |
|---|---|
| Candidate Name | Alex Rivera |
| Email | zaynbaloch89@gmail.com |
| Phone | 03088320756 |
| Position | Backend Developer |
| Experience | 3 years |
| Expected Salary | $95,000/year |
| Availability | 2 Weeks Notice |
| Portfolio | zainkhanbaloch.tech |

### Expected Priority

```text
Medium
```

### Pipeline

```text
Applied
→ Screening
→ Technical Interview
→ Rejected
```

### Verification

- ✅ Priority correctly evaluated as **Medium**
- ✅ Rejection email delivered successfully
- ✅ Editing `Recruiter Notes` after rejection did not trigger another rejection email
- ✅ Duplicate-email protection worked correctly

---

## Test Case 3 — Low Priority / Direct Rejection

### Candidate Data

| Field | Value |
|---|---|
| Candidate Name | Liam Chen |
| Email | zaynbaloch89@gmail.com |
| Phone | 03088320756 |
| Position | Frontend Developer |
| Experience | 1 year |
| Expected Salary | $70,000/year |
| Availability | 1 Month Notice |
| Portfolio | zainkhanbaloch.tech |

### Expected Priority

```text
Low
```

### Pipeline

```text
Applied
→ Rejected
```

### Verification

- ✅ Priority correctly evaluated as **Low**
- ✅ Direct rejection email sent successfully
- ✅ Candidate record remained consistent after rejection

---

## Test Case 4 — Inactivity Reminder

### Test Configuration

For testing purposes:

```text
Production Delay = 5 Days
Test Delay = 5 Minutes
```

### Scenario

Candidate remained in:

```text
Screening
```

without any updates.

### Result

- ✅ Recruiter inactivity alert was triggered after the five-minute testing delay
- ✅ Candidate state was re-fetched before sending the reminder
- ✅ Reminder was sent only after the state-validation checks passed

---

## Test Case 5 — Delayed Reminder Cancellation After Stage Progression

### Scenario

1. Candidate moved to `Screening`
2. Inactivity timer started
3. Two minutes later, candidate moved to `Technical Interview`
4. Original delayed execution resumed after five minutes
5. Zap fetched the current record
6. `Last Updated Date` no longer matched the original timestamp

### Result

- ✅ Timestamp mismatch detected
- ✅ Original inactivity execution stopped
- ✅ No false inactivity email was sent

This confirms that delayed jobs cannot act on outdated candidate state.

---

# 🛡️ Reliability & Safeguards

## 1. Stage Email Guard

```text
Recruitment Stage != Last Emailed Stage
```

Prevents duplicate stage notifications.

---

## 2. Loop Prevention

Because Zap 2 updates the same table that triggers it, the `Last Emailed Stage` guard prevents update-trigger-update recursion.

---

## 3. Fresh-State Validation

The inactivity Zap does not trust the candidate state captured five days earlier.

Instead, it fetches the latest database record before taking action.

---

## 4. Terminal Stage Protection

Candidates in:

```text
Hired
Rejected
```

are excluded from inactivity alerts.

---

## 5. Timestamp-Based Change Detection

`Last Updated Date` acts as a lightweight state version.

If the timestamp changes during the delay window, the original reminder execution becomes invalid.

---

# 🚀 Key Technical Highlights

### Event-Driven Recruitment Workflow

Every major recruitment action is triggered by a real system event:

- Form submission
- Candidate record update
- Kanban stage change
- Inactivity timeout

---

### Experience-Based Candidate Prioritization

Applicants are automatically classified into:

```text
High
Medium
Low
```

priority groups based on software-development experience.

---

### State-Aware Automation

The system remembers which stage has already generated an email and validates record state before performing delayed actions.

---

### Loop-Free Table Automations

Zapier Table updates normally create a risk of recursive automation triggers.

This project handles that problem using explicit state comparison through:

```text
Last Emailed Stage
```

---

### Delay-Safe Reminder Architecture

Delayed automation executions can become outdated.

The workflow solves this by:

1. Capturing the original update timestamp
2. Waiting five days
3. Re-fetching the current candidate record
4. Comparing timestamps
5. Sending a reminder only when the candidate has not changed

---

### Native Zapier Ecosystem

The complete ATS was implemented without external automation or database platforms.

Used technologies include:

- Zapier Interfaces
- Zapier Tables
- Zapier Kanban
- Zapier Paths
- Zapier Filters
- Delay by Zapier
- Email by Zapier
- Zapier Automations

---

# 📈 Business Benefits

This system helps recruitment teams:

- Reduce manual candidate tracking
- Standardize recruitment processes
- Respond to applicants more consistently
- Automatically identify experienced candidates
- Prevent duplicate candidate communication
- Avoid candidates being forgotten in the pipeline
- Keep HR informed at the correct stage
- Maintain a centralized recruitment database
- Improve recruiter accountability
- Provide a better candidate experience

---

# 📚 Skills Demonstrated

This project demonstrates practical experience with:

- Workflow automation
- Event-driven system design
- Applicant Tracking System architecture
- Zapier Tables database design
- Zapier Interfaces
- Zapier Kanban
- Paths and conditional routing
- Filters and guard conditions
- State management
- Delayed workflow validation
- Loop prevention
- Automated email communication
- Business process automation
- Recruitment workflow optimization
- End-to-end testing and verification

---

# ✅ Project Status

**Completed and fully tested.**

Verified components include:

- ✅ Candidate form submission
- ✅ Candidate record creation
- ✅ Candidate ID generation
- ✅ High priority scoring
- ✅ Medium priority scoring
- ✅ Low priority scoring
- ✅ Kanban stage progression
- ✅ Screening email
- ✅ Technical Interview email
- ✅ HR notification
- ✅ Hired / offer confirmation email
- ✅ Rejection email
- ✅ Duplicate-email prevention
- ✅ Recruiter Notes edit protection
- ✅ Five-day inactivity architecture
- ✅ Delayed state re-validation
- ✅ False-reminder prevention

---

# 🏁 Conclusion

The **Automated Recruitment Pipeline & ATS in Zapier** is a complete recruitment automation system that combines candidate intake, data management, recruitment-stage tracking, automated communication, and inactivity monitoring into a single event-driven workflow.

A major strength of the project is not only automating recruitment actions, but also protecting those automations against common workflow issues such as **duplicate triggers, recursive table updates, stale delayed executions, and false inactivity notifications**.

The result is a practical, scalable, and portfolio-ready Applicant Tracking System built entirely within the Zapier ecosystem.
