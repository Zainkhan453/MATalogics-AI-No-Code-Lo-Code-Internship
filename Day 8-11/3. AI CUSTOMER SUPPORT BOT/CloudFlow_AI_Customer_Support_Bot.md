# CloudFlow AI Customer Support Bot

## Project Overview

The **CloudFlow AI Customer Support Bot** is a Zapier-based customer support automation project developed for a fictional software company named **CloudFlow**.

The chatbot is designed to:

- Answer customer questions using a controlled knowledge base.
- Handle pricing, product features, account questions, refund policy, support hours, and basic troubleshooting.
- Avoid inventing information when the answer is not available.
- Escalate unresolved issues or human-support requests.
- Collect customer details for support tickets.
- Detect ticket priority using AI.
- Update support tickets in Zapier Tables.
- Route tickets using Paths by Zapier.
- Send priority-based notifications to Slack.

---

## Project Objective

The objective of this project is to build an automated AI customer support workflow with the following flow:

```text
Customer
   ↓
CloudFlow AI Customer Support Chatbot
   ↓
Knowledge Base
   ↓
Known Question?
   ├── Yes → Answer Customer
   └── No / Human Support Requested
              ↓
      Collect Name + Email + Problem
              ↓
        Zapier Support Tickets Table
              ↓
          Zap Workflow Trigger
              ↓
         AI Priority Detection
              ↓
          Update Ticket Record
              ↓
          Generate Ticket ID
              ↓
          Update Ticket ID
              ↓
             Paths
      ┌────────┼────────┬────────┐
   Critical   High    Medium     Low
      ↓        ↓        ↓         ↓
   Slack     Slack     Slack      Slack
 Notification Notification Notification Notification
              ↓
      #cloudflow-support
```

---

# Tools Used

- **Zapier Chatbots**
- **Zapier Tables**
- **AI by Zapier**
- **Code by Zapier**
- **Paths by Zapier**
- **Slack**
- **CloudFlow Knowledge Base**

---

# 1. Zapier Table

A Zapier Table named:

```text
Support Tickets
```

was created.

## Table Columns

| Column | Purpose |
|---|---|
| Ticket ID | Unique identifier for each ticket |
| Customer | Customer name |
| Email | Customer email address |
| Issue | Customer problem/support request |
| Priority | Critical, High, Medium, or Low |
| Status | Current ticket status |
| Created At | Ticket creation date/time |

### Priority Values

- Critical
- High
- Medium
- Low

### Default Ticket Status

```text
New
```

---

# 2. CloudFlow AI Customer Support Chatbot

A chatbot named:

```text
CloudFlow AI Customer Support
```

was created in Zapier Chatbots.

The chatbot handles:

- Pricing questions
- Product features
- Account questions
- Basic troubleshooting
- Refund questions
- Support hours
- Support escalation

---

# 3. CloudFlow Knowledge Base

The chatbot uses a controlled knowledge base so that it only answers questions using approved CloudFlow information.

## Pricing Plans

### Starter

```text
$19/month
```

### Professional

```text
$49/month
```

### Business

```text
$149/month
```

---

## Support Hours

```text
Monday–Friday
9:00 AM–6:00 PM
```

---

## Refund Policy

Customers can request a refund within:

```text
14 days
```

of purchase.

---

## Product Features

CloudFlow provides workflow automation capabilities for individuals, teams, and businesses.

### Starter

- Basic workflow automation
- Suitable for individuals and small teams

### Professional

- Advanced workflow automation
- Integrations
- Suitable for growing teams

### Business

- Advanced automation
- Team collaboration
- Suitable for larger businesses

---

## Common Troubleshooting

### Workflow Not Running

The chatbot recommends:

1. Check whether the workflow is enabled.
2. Check connected accounts.
3. Verify required fields.
4. Retry the workflow.

### Integration Disconnected

The chatbot recommends:

1. Open integration settings.
2. Reconnect the application.
3. Authorize CloudFlow again.
4. Test the connection.

### Cannot Access Account

The chatbot recommends:

1. Verify the email address.
2. Use the **Forgot Password** option.
3. Check the spam/junk folder.
4. Try logging in again.

If the issue remains unresolved, the chatbot offers support escalation.

---

# 4. Chatbot Safety Rule

A major requirement of the project is that the chatbot must **not invent information**.

The chatbot was instructed to answer only from the provided CloudFlow knowledge base.

If the required information is unavailable, it responds with:

> I don't have enough information to answer that accurately. I can create a support request for you.

This prevents unsupported or hallucinated answers.

---

# 5. Human Support Escalation

If the customer asks for human support using phrases such as:

```text
talk to a human
human
support agent
speak to an agent
customer support
contact support
talk to support
create support request
support request
```

the chatbot starts the escalation process.

The chatbot collects:

- Name
- Email
- Problem

---

# 6. Zap Trigger

The chatbot support information is stored in the **Support Tickets** Zapier Table.

The Zap workflow starts with:

```text
Zapier Tables → New Record
```

The trigger receives customer information such as:

```text
Name: Zain Khan
Email: customer@example.com
Problem: I can't access my account.
```

---

# 7. AI Priority Detection

After the Zap is triggered, **AI by Zapier** analyzes the customer's support problem.

The AI classifies the ticket into exactly one of four priorities.

## Priority Rules

### Critical

Used when:

- The entire system is completely down.
- The complete service is unavailable.
- A major system-wide outage is reported.

### High

Used when:

- The customer cannot access their account.
- The customer cannot log in.
- There is an important account-access issue.

### Medium

Used for:

- Normal technical problems.
- Regular support questions.
- Issues requiring support but not urgent escalation.

### Low

Used for:

- General requests.
- Suggestions.
- Feedback.
- Other non-urgent requests.

The AI is instructed to return only:

```text
Critical
High
Medium
Low
```

---

# 8. Update Support Ticket Record

After priority detection, the existing Support Tickets record is updated.

## Field Mapping

| Support Ticket Field | Source |
|---|---|
| Customer | Name collected by chatbot |
| Email | Email collected by chatbot |
| Issue | Problem collected by chatbot |
| Priority | AI by Zapier output |
| Status | New |
| Created At | Record creation timestamp |

This updates the existing record instead of creating a duplicate.

---

# 9. Ticket ID Generation

A unique Ticket ID is generated using **Code by Zapier**.

## Input

```text
record_id
```

The value is mapped from the Zapier Tables Record ID.

## JavaScript

```javascript
return {
  ticket_id: "TKT-" + inputData.record_id
};
```

## Example Output

```text
TKT-01M0WV9976CFV8P5J3R62RZXMY
```

The same Support Tickets record is updated again with this Ticket ID.

---

# 10. Paths by Zapier

The workflow contains four Paths:

```text
Path A → Critical
Path B → High
Path C → Medium
Path D → Low
```

## Path Conditions

### Critical Path

```text
Priority exactly matches Critical
```

### High Path

```text
Priority exactly matches High
```

### Medium Path

```text
Priority exactly matches Medium
```

### Low Path

```text
Priority exactly matches Low
```

Only the path matching the detected AI priority continues.

---

# 11. Slack Notification System

A Slack channel was created:

```text
#cloudflow-support
```

A Slack integration/app is connected to Zapier and notifications are sent to this channel.

Each priority path sends a different Slack message.

---

## Critical Notification

```text
🚨 CRITICAL SUPPORT TICKET

Ticket ID: {{Ticket ID}}
Customer: {{Customer}}
Email: {{Email}}
Priority: Critical
Status: New

Issue:
{{Issue}}

Immediate attention is required.
```

---

## High Notification

```text
⚠️ HIGH PRIORITY SUPPORT TICKET

Ticket ID: {{Ticket ID}}
Customer: {{Customer}}
Email: {{Email}}
Priority: High
Status: New

Issue:
{{Issue}}
```

---

## Medium Notification

```text
📩 NEW SUPPORT TICKET

Ticket ID: {{Ticket ID}}
Customer: {{Customer}}
Email: {{Email}}
Priority: Medium
Status: New

Issue:
{{Issue}}
```

---

## Low Notification

```text
ℹ️ GENERAL SUPPORT REQUEST

Ticket ID: {{Ticket ID}}
Customer: {{Customer}}
Email: {{Email}}
Priority: Low
Status: New

Issue:
{{Issue}}
```

---

# Final Zap Workflow

```text
1. Zapier Tables
   New Record
        ↓
2. AI by Zapier
   Detect Priority
        ↓
3. Zapier Tables
   Update Record
        ↓
4. Code by Zapier
   Generate Ticket ID
        ↓
5. Zapier Tables
   Update Ticket ID
        ↓
6. Paths by Zapier
   ├── Critical
   ├── High
   ├── Medium
   └── Low
        ↓
7. Slack
   Send Channel Message
        ↓
#cloudflow-support
```

---

# Test Cases

All test cases below were manually verified and **passed successfully**.

---

## Test Case 1 — Critical Priority

### Customer Message

```text
The CloudFlow system is completely down and none of our workflows are working.
```

### Test Data

```text
Name: Ahmed Khan
Email: ahmed@example.com
Problem: The CloudFlow system is completely down and none of our workflows are working.
```

### Expected Result

- Chatbot collects Name, Email, and Problem.
- Support ticket is created.
- AI detects priority as **Critical**.
- Ticket ID is generated.
- Status is **New**.
- Critical Path runs.
- Slack notification is sent to `#cloudflow-support`.

### Actual Result

All expected actions completed successfully.

### Result

```text
PASS
```

---

## Test Case 2 — High Priority

### Customer Message

```text
I can't access my CloudFlow account even after resetting my password.
```

### Test Data

```text
Name: Sara Ali
Email: sara@example.com
Problem: I can't access my CloudFlow account even after resetting my password.
```

### Expected Result

- Support request is collected.
- AI detects priority as **High**.
- Ticket ID is generated.
- Status is **New**.
- High Path runs.
- Slack notification is sent successfully.

### Actual Result

All expected actions completed successfully.

### Result

```text
PASS
```

---

## Test Case 3 — Medium Priority

### Customer Message

```text
My workflow is not running properly. I checked that it is enabled, but it is still not working.
```

### Test Data

```text
Name: Bilal Ahmed
Email: bilal@example.com
Problem: My workflow is not running properly. I checked that it is enabled, but it is still not working.
```

### Expected Result

- AI detects priority as **Medium**.
- Medium Path runs.
- Support ticket is updated successfully.
- Slack receives a Medium Priority notification.

### Actual Result

All expected actions completed successfully.

### Result

```text
PASS
```

---

## Test Case 4 — Low Priority

### Customer Message

```text
I would like to suggest adding a dark mode feature to CloudFlow.
```

### Test Data

```text
Name: Fatima Noor
Email: fatima@example.com
Problem: I would like to suggest adding a dark mode feature to CloudFlow.
```

### Expected Result

- AI detects priority as **Low**.
- Low Path runs.
- Ticket is stored successfully.
- Slack receives a Low Priority support notification.

### Actual Result

All expected actions completed successfully.

### Result

```text
PASS
```

---

## Test Case 5 — Knowledge Base / No Ticket Creation

### Customer Message

```text
How much does the Professional plan cost?
```

### Expected Chatbot Response

```text
The Professional plan costs $49/month.
```

### Expected Result

- Chatbot answers directly from the knowledge base.
- Name, Email, and Problem are not requested.
- No support ticket is created.
- Zap workflow does not run.
- No Slack notification is sent.

### Actual Result

The chatbot answered correctly and no unnecessary support ticket was created.

### Result

```text
PASS
```

---

## Test Case 6 — Unknown Information / Hallucination Prevention

### Customer Message

```text
Does CloudFlow have a mobile Android application?
```

### Expected Chatbot Response

```text
I don't have enough information to answer that accurately. I can create a support request for you.
```

### Expected Result

- Chatbot does not invent an answer.
- Chatbot offers support escalation.
- If the customer accepts, Name, Email, and Problem are collected.
- The support request continues through the Zap workflow.

### Actual Result

The chatbot correctly avoided inventing information and offered support escalation.

### Result

```text
PASS
```

---

# Test Summary

| Test Case | Scenario | Expected Priority/Behavior | Result |
|---|---|---|---|
| TC-01 | System completely down | Critical | PASS |
| TC-02 | Cannot access account | High | PASS |
| TC-03 | Workflow technical problem | Medium | PASS |
| TC-04 | General feature suggestion | Low | PASS |
| TC-05 | Pricing knowledge-base question | Direct answer, no ticket | PASS |
| TC-06 | Unknown product information | No hallucination, offer escalation | PASS |

## Overall Result

```text
6 / 6 TEST CASES PASSED
```

The complete CloudFlow AI Customer Support Bot automation was tested successfully.

---

# Key Features Implemented

- AI-powered customer support chatbot
- Controlled knowledge base
- Pricing support
- Product feature support
- Refund policy support
- Account support
- Basic troubleshooting
- Hallucination prevention
- Human escalation
- Customer information collection
- AI ticket priority classification
- Unique Ticket ID generation
- Zapier Tables integration
- Priority-based branching
- Slack notifications
- Automated support ticket management

---

# Conclusion

The **CloudFlow AI Customer Support Bot** successfully demonstrates an end-to-end AI customer support automation using Zapier.

The system can answer standard customer questions directly from a knowledge base while safely escalating unknown or unresolved issues. Support requests are automatically stored, prioritized using AI, assigned unique Ticket IDs, routed according to urgency, and sent to the CloudFlow support team through Slack.

All configured functional and priority test cases were verified successfully, with a final result of:

```text
6 / 6 TEST CASES PASSED
```

This project demonstrates practical use of AI, chatbot automation, workflow orchestration, conditional routing, structured data handling, and real-time team notifications.
