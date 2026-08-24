# Employee Expense Approval System

## Project Overview

The **Employee Expense Approval System** is an internal expense-management workflow built in **Zapier**. Employees submit expenses through **Zapier Forms**. The automation generates a Request ID, evaluates risk, checks receipt availability, updates the request in **Zapier Tables**, and routes the request through the correct approval path.

The workflow supports:

- Automatic Request ID generation
- Risk classification
- Receipt validation
- Automatic approval for low-risk expenses
- Manager approval for medium-risk expenses
- Manager + Finance approval for high-risk expenses
- Approved / Rejected status updates in Zapier Tables
- Employee email notifications
- Human-in-the-Loop approvals

## Files

- [Zapier Form Link.md](Zapier%20Form%20Link.md) - public Employee Expense Portal form
- `Workflow/` - sanitized exported Zap configuration
- `Screenshots/` - publish-safe workflow and test evidence

---

## Zapier Form

### Form Name

**Employee Expense Portal**

### Form Fields

- Employee Name
- Employee Email
- Department
- Expense Type
- Amount
- Expense Date
- Description
- Receipt Upload
- Manager Email

### Expense Type Options

- Travel
- Food
- Software
- Equipment
- Other

### Department Options

- Engineering
- IT
- Sales
- Marketing
- Finance
- Human Resources
- Operations
- Customer Support
- Administration
- Other

> The **Receipt Upload** field is optional so the workflow can detect missing receipts.

---

## Zapier Table

### Table Name

**Expense Requests**

### Columns

- Request ID
- Employee
- Employee Email
- Department
- Type
- Amount
- Expense Date
- Description
- Receipt
- Manager
- Approval Status
- Risk Level
- Submitted At

### Approval Status Values

- Pending
- Approved
- Rejected
- Receipt Required

### Risk Level Values

- Low
- Medium
- High

---

## Workflow Architecture

```text
Zapier Form Submission
        ↓
Code by Zapier
        ↓
Zapier Tables - Update Record
        ↓
Paths by Zapier
        │
        ├── Receipt Required
        │       ↓
        │     Gmail
        │
        ├── Low Risk
        │       ↓
        │     Gmail
        │
        ├── Medium Risk
        │       ↓
        │   Human in the Loop
        │       ↓
        │   Manager Decision
        │     ↙       ↘
        │ Approved   Rejected
        │    ↓          ↓
        │ Update      Update
        │ Record      Record
        │
        └── High Risk
                ↓
            Manager Approval
                ↓
          Manager Decision
            ↙         ↘
       Rejected      Approved
          ↓             ↓
      Update Record   Finance Approval
                         ↓
                   Finance Decision
                     ↙       ↘
                Approved   Rejected
                   ↓          ↓
               Update       Update
               Record       Record
```

---

## Automation Logic

### Request ID

The Code by Zapier step generates IDs in the format:

```text
EXP-XXXX
```

Example:

```text
EXP-8227
```

### Risk Rules

**Low Risk**

```text
Amount < $100
```

Result:

```text
Risk Level = Low
Approval Status = Approved
```

**Medium Risk**

```text
Amount = $100 to $500
```

Result:

```text
Risk Level = Medium
Approval Status = Pending
```

Then the request is sent to the manager.

**High Risk**

```text
Amount > $500
```

Result:

```text
Risk Level = High
Approval Status = Pending
```

Then the request requires manager approval followed by Finance approval.

### Missing Receipt Rule

Receipt availability overrides the normal approval flow.

```text
If receipt is missing:
Approval Status = Receipt Required
```

Example:

```text
Expense Type: Software
Amount: $750
Receipt: Missing

Risk Level: High
Approval Status: Receipt Required
```

---

## Path Logic

### Path 1 - Receipt Required

Condition:

```text
Approval Status exactly matches Receipt Required
```

Action:

- Send receipt-required email to the employee.
- Keep status as `Receipt Required`.

### Path 2 - Low Risk

Conditions:

```text
Risk Level exactly matches Low
AND
Approval Status exactly matches Approved
```

Action:

- Automatically approve the expense.
- Send approval email to the employee.

### Path 3 - Medium Risk

Conditions:

```text
Risk Level exactly matches Medium
AND
Approval Status exactly matches Pending
```

Action:

- Send approval request to manager through Human in the Loop.
- Use Human in the Loop `Decision` output.

Decision branches:

```text
Decision = approved
→ Update Approval Status to Approved

Decision = rejected
→ Update Approval Status to Rejected
```

### Path 4 - High Risk

Conditions:

```text
Risk Level exactly matches High
AND
Approval Status exactly matches Pending
```

Flow:

```text
Manager Approval
        ↓
Manager Decision
        ↓
If Approved → Finance Approval
If Rejected → Approval Status = Rejected
```

Finance decision:

```text
Finance Approved → Approval Status = Approved
Finance Rejected → Approval Status = Rejected
```

---

## Human in the Loop Configuration

### Medium Risk Approval Message

```text
A medium-risk employee expense requires your approval.
Please review the expense details below and approve or reject the request.
```

### High Risk Manager Approval Message

```text
This is a high-risk employee expense.
Please review the expense details and approve or reject the request.
```

### High Risk Finance Approval Message

```text
HIGH-RISK expense requires Finance approval.
The manager has already approved this request.
Please review the expense details and approve or reject the request.
```

### Content to Review

- Request ID
- Employee Name
- Employee Email
- Department
- Expense Type
- Amount
- Expense Date
- Description
- Receipt
- Manager Email
- Risk Level
- Approval Status

### Approval Controls

```text
Approve Button: Approve
Decline Button: Reject
```

### Review Settings

```text
Action if reviewer declines: Continue
Allow reviewer to edit content: No
Send request via: Email
Timeout: 7 Days
Reminder: Enabled
Reminder interval: 1 Day
```

---

## Record Update Logic

Zapier Tables requires an internal **Record ID** when updating a row.

The workflow finds the correct record using the generated Request ID and then updates the matching internal record.

```text
Request ID = EXP-XXXX
        ↓
Find Record
        ↓
Internal Zapier Table Record ID
        ↓
Update Record
        ↓
Approval Status = Approved / Rejected
```

This prevents duplicate rows and ensures the existing expense request is updated.

---

## Test Cases

All four required branches were tested successfully.

## Test Case 1 - Receipt Required

### Input

```text
Employee Name: Ahmed Raza
Employee Email: ahmed.raza@example.com
Department: Engineering
Expense Type: Software
Amount: $750
Expense Date: 08/25/2026
Description: Annual software subscription for development tools.
Receipt Upload: Missing
Manager Email: Working reviewer email
```

### Expected Result

```text
Risk Level: High
Approval Status: Receipt Required
Path: Receipt Required
```

### Actual Result

```text
Risk Level: High
Approval Status: Receipt Required
Receipt Required path executed successfully.
Employee notification email was sent successfully.
```

### Status

**PASSED**

---

## Test Case 2 - Low Risk

### Input

```text
Employee Name: Sara Khan
Employee Email: sara.khan@example.com
Department: Marketing
Expense Type: Food
Amount: $65
Expense Date: 08/25/2026
Description: Team lunch during a client meeting.
Receipt Upload: Uploaded
Manager Email: Working reviewer email
```

### Expected Result

```text
Risk Level: Low
Approval Status: Approved
Path: Low Risk
```

### Actual Result

```text
Risk Level: Low
Approval Status: Approved
Low Risk path executed successfully.
Expense was automatically approved.
Employee approval email was sent successfully.
```

### Status

**PASSED**

---

## Test Case 3 - Medium Risk

### Input

```text
Employee Name: Usman Ali
Employee Email: usman.ali@example.com
Department: Sales
Expense Type: Equipment
Amount: $350
Expense Date: 08/25/2026
Description: Headset and accessories for sales calls.
Receipt Upload: Uploaded
Manager Email: Working Zapier reviewer email
```

### Expected Result

```text
Risk Level: Medium
Initial Approval Status: Pending
Path: Medium Risk
```

### Actual Result

Both approval outcomes were tested successfully:

```text
Manager Approve → Approval Status = Approved
Manager Reject  → Approval Status = Rejected
```

The correct expense record was found and updated in Zapier Tables.

### Status

**PASSED**

---

## Test Case 4 - High Risk

### Input

```text
Employee Name: Hamza Ahmed
Employee Email: hamza.ahmed@example.com
Department: Engineering
Expense Type: Equipment
Amount: $850
Expense Date: 08/25/2026
Description: High-performance monitor for development workstation.
Receipt Upload: Uploaded
Manager Email: Working Zapier reviewer email
```

### Expected Result

```text
Risk Level: High
Initial Approval Status: Pending
Path: High Risk
```

### Approval Flow

```text
High Risk
    ↓
Manager Approval
    ↓
Manager Approves
    ↓
Finance Approval
    ↓
Finance Approves
    ↓
Approval Status = Approved
```

Rejection logic was also implemented:

```text
Manager Rejects → Rejected

Manager Approves
    ↓
Finance Rejects → Rejected
```

### Actual Result

The complete Manager + Finance approval workflow executed successfully, and the final table status was updated correctly.

### Status

**PASSED**

---

## Test Summary

| Test | Scenario | Amount | Receipt | Risk | Final Result | Status |
|---|---|---:|---|---|---|---|
| TC-01 | Missing Receipt | $750 | Missing | High | Receipt Required | PASSED |
| TC-02 | Low Risk | $65 | Uploaded | Low | Approved Automatically | PASSED |
| TC-03 | Medium Risk | $350 | Uploaded | Medium | Manager Approval / Rejection | PASSED |
| TC-04 | High Risk | $850 | Uploaded | High | Manager + Finance Approval | PASSED |

---

## Technologies Used

- Zapier Forms
- Zapier Tables
- Code by Zapier
- JavaScript
- Paths by Zapier
- Gmail
- Human in the Loop
- Find Record
- Update Record

---

## Final Result

The **Employee Expense Approval System** was successfully implemented and tested.

The completed workflow can:

- accept employee expense submissions
- generate unique expense Request IDs
- store and update expenses in Zapier Tables
- classify Low, Medium, and High Risk expenses
- detect missing receipts
- automatically approve low-risk expenses
- request manager approval for medium-risk expenses
- request manager and Finance approval for high-risk expenses
- record both approved and rejected decisions
- notify employees through Gmail
- maintain the final approval status in Zapier Tables

All required test scenarios passed successfully.

---

## Import and plan notes

- Reconnect Zapier Forms, Zapier Tables, Gmail and Human in the Loop after import.
- Replace the placeholder Table and reviewer IDs with your own Zapier resources.
- Recheck the approval-decision and record-ID field mappings after import; Zapier's export metadata flags several copied mappings for verification.
- Human in the Loop is a paid Zapier feature. A suitable plan or active trial is required for the approval branches to keep running.
- Gmail connection references were removed from the exported JSON before publication.
- Screenshots containing personal Gmail/account data remain local and are intentionally excluded by the repository `.gitignore`.

---

## Project Status

**Completed and Successfully Tested**
