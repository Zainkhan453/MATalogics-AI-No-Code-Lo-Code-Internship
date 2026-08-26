# SmartCare Clinic — Complete AI Appointment Booking System

> **Complete Zapier Project Report:** This document covers the entire working system: the patient-facing chatbot, confirmation button, Zap trigger, date formatter, AI extraction, validation filter, JavaScript processing, Zapier Tables availability search, booking and unavailable paths, record creation, double-booking prevention, and Gmail notifications.

## Project Overview

The SmartCare Clinic AI Appointment Booking System is a complete Zapier automation for collecting appointment requests, checking doctor availability, preventing double bookings, creating confirmed appointments, and notifying patients by email.

The chatbot is only the patient-facing part of the project. After the patient confirms the request, the connected Zap processes the complete conversation, normalizes its data, searches the Appointments table, chooses the correct availability path, and sends the final result through Gmail.

## Complete System Scope

| System Part | Responsibility |
|---|---|
| Zapier Chatbot | Collect and confirm patient information |
| Chatbot Logic/Button | Submit only a complete confirmed request |
| Formatter by Zapier | Supply the current clinic date in Pakistan time |
| AI by Zapier | Extract and normalize seven appointment fields |
| Filter by Zapier | Stop incomplete or unconfirmed requests |
| Code by Zapier | Validate values and generate the Appointment ID, Slot Key, and alternative times |
| Zapier Tables Search | Check whether the doctor is already booked at the requested date and time |
| Paths by Zapier | Separate available and unavailable requests |
| Zapier Tables Create Record | Store an appointment only when the slot is available |
| Gmail | Inform the patient whether the request was booked or unavailable |

## Project Objectives

- Collect complete patient and appointment information conversationally.
- Recognize either a doctor's name or medical specialty.
- Convert relative dates such as `tomorrow` into an actual date.
- Normalize appointment times into 24-hour format.
- Check the requested doctor's availability.
- Prevent more than one booked appointment for the same doctor, date, and time.
- Generate a unique Appointment ID.
- Create records only for available slots.
- Set successful appointments to `Booked`.
- Notify patients by Gmail whether the appointment was confirmed or unavailable.
- Suggest one hour earlier and one hour later when a requested slot is unavailable.

## Available Doctors

| Doctor | Specialty |
|---|---|
| Dr. Ahmed | General Physician |
| Dr. Sara | Dermatologist |
| Dr. Ali | Cardiologist |

## Technology Stack

- Zapier Chatbots
- AI by Zapier
- Formatter by Zapier
- Code by Zapier (JavaScript)
- Zapier Tables
- Paths by Zapier
- Gmail

## Information Collected

The chatbot collects and confirms the following seven fields:

1. Patient name
2. Email address
3. Phone number
4. Doctor or specialist
5. Preferred date
6. Preferred time
7. Reason for appointment

Email was added to the original requirements so the automation can send confirmation and unavailable-slot notifications to the patient.

## Appointments Table

Table name: `Appointments`

| Column | Field Type | Purpose |
|---|---|---|
| Appointment ID | Text | Unique appointment reference |
| Patient | Text | Patient's full name |
| Email | Email | Recipient for booking notifications |
| Phone | Phone Number/Text | Patient contact number |
| Doctor | Dropdown | Standardized doctor and specialty |
| Date | Date & Time | Appointment date |
| Time | Text | Time in `HH:mm` format |
| Reason | Long Text | Reason for appointment |
| Status | Dropdown | Set to `Booked` for confirmed appointments |
| Slot Key | Text | Unique doctor/date/time combination |

### Doctor Dropdown Values

```text
Dr. Ahmed - General Physician
Dr. Sara - Dermatologist
Dr. Ali - Cardiologist
```

### Status Value

```text
Booked
```

### Slot Key Format

```text
Doctor|YYYY-MM-DD|HH:mm
```

Example:

```text
Dr. Sara - Dermatologist|2026-09-15|14:00
```

## Final Workflow Architecture

```text
Zapier Chatbot Button Click
        ↓
Formatter — Current Clinic Date (Asia/Karachi)
        ↓
AI by Zapier — Extract Appointment Details
        ↓
Filter — Continue Only If Information Is Complete
        ↓
Code by Zapier — Generate Appointment ID, Slot Key and Alternative Times
        ↓
Zapier Tables — Find Existing Booked Appointment
        ↓
Paths by Zapier
        ├── Slot Unavailable
        │      └── Gmail — Send Unavailable-Slot Email
        │
        └── Slot Available
               ├── Zapier Tables — Create Appointment Record
               └── Gmail — Send Booking Confirmation
```

## Complete Zap: Node-by-Node Configuration

### Step 1 — Trigger: Zapier Chatbots

**App:** Zapier Chatbots
**Event:** Button/logic submission from the SmartCare Clinic chatbot
**Button text:** `Check Availability & Book`

The trigger must provide the complete conversation transcript. The chatbot displays the button only after the patient confirms the complete seven-field summary with `CONFIRM BOOKING`.

### Step 2 — Formatter: Current Clinic Date

**App:** Formatter by Zapier
**Transform:** Format Date/Time
**Timezone:** `Asia/Karachi`
**Output format:** `YYYY-MM-DD`

This output is sent to AI by Zapier as the current clinic date. It ensures that relative words such as `today` and `tomorrow` are calculated from the clinic's date rather than an unknown model date.

### Step 3 — AI by Zapier: Extract Appointment Details

**Model:** Standard (Auto)
**Input:** Chatbot transcript and current clinic date
**Return as array of objects:** Off
**Tools:** None required

Configured output fields:

| Field | Required output |
|---|---|
| `patient` | Patient's confirmed full name |
| `email` | Confirmed lowercase email address |
| `phone` | Confirmed phone number |
| `doctor` | One standardized doctor value |
| `preferred_date` | `YYYY-MM-DD` |
| `preferred_time` | `HH:mm` |
| `reason` | Confirmed appointment reason |
| `booking_complete` | `Yes` or `No` |

### Step 4 — Filter: Continue Only for Complete Requests

**App:** Filter by Zapier

Continue only when:

```text
AI output: booking_complete
Condition: Exactly matches
Value: Yes
```

This prevents incomplete conversations from reaching the availability search or creating table records.

### Step 5 — Code by Zapier: Prepare Booking Data

**App:** Code by Zapier
**Event:** Run JavaScript

Input mapping:

| Input Data Key | Source |
|---|---|
| `doctor` | AI output: `doctor` |
| `date` | AI output: `preferred_date` |
| `time` | AI output: `preferred_time` |

The code validates the normalized date and time, generates a unique Appointment ID, constructs the Slot Key, and calculates an earlier and later alternative time.

### Step 6 — Zapier Tables: Check Requested Slot Availability

**App:** Zapier Tables
**Event:** Find Record
**Table:** `Appointments`

Use exact matching for:

| Search Field | Search Value |
|---|---|
| Doctor | AI output: `doctor` |
| Date | AI output: `preferred_date` |
| Time | AI output: `preferred_time` |
| Status | Static text: `Booked` |

Settings:

```text
Successful if no search results are found: True
If multiple search results are found: Return first search result
Create Zapier Tables record if it does not exist: Disabled
```

The search step must never create a record. Its only job is to report whether a matching booked appointment exists.

### Step 7 — Paths by Zapier

Create two paths using the search output `Zap Search Was Found`.

| Path | Condition | Result |
|---|---|---|
| A — Slot Unavailable | `Zap Search Was Found` exactly matches `true` | Do not create a record; send unavailable email |
| B — Slot Available | `Zap Search Was Found` exactly matches `false` | Create one appointment; send confirmation email |

### Step 8A — Unavailable Path: Gmail

**App:** Gmail
**Event:** Send Email
**To:** AI output `email`

No Create Record node is placed in this path. The email states that no appointment was booked and suggests the earlier and later times produced by the Code step.

### Step 8B — Available Path: Create Appointment Record

**App:** Zapier Tables
**Event:** Create Record
**Table:** `Appointments`

Map every field from the AI and Code steps. Set `Status` manually to `Booked`. This action exists only inside the available path.

### Step 9 — Available Path: Gmail Confirmation

**App:** Gmail
**Event:** Send Email
**To:** AI output `email`

Place this step after Create Record so an email is sent only when the appointment has actually been stored successfully.

## Chatbot Configuration

### Conversation Behaviour

The chatbot:

- Extracts any information already supplied by the patient.
- Requests all missing fields together in one message.
- Asks only for fields that are missing or invalid.
- Rejects doctors outside the approved list.
- Validates that an email address contains a username, `@`, and domain.
- Preserves the patient's valid information across messages.
- Shows the final summary only after all seven fields are complete.
- Allows corrections before submission.
- Does not claim that an appointment is booked before the Zap completes.

### Final Confirmation Phrase

The patient must enter:

```text
CONFIRM BOOKING
```

The chatbot button logic is configured as:

```text
Show this logic: When certain keywords are used
Keyword: CONFIRM BOOKING
Button: Check Availability & Book
```

Using a unique confirmation phrase prevents the button from appearing before all appointment information has been collected and reviewed.

## Date and Time Processing

### Current Clinic Date

Formatter by Zapier supplies the current clinic date using:

```text
Timezone: Asia/Karachi
Output format: YYYY-MM-DD
```

AI by Zapier treats this value as `today` and converts expressions such as:

| Patient Input | Normalized Result |
|---|---|
| Today | Current clinic date |
| Tomorrow | Current clinic date + 1 day |
| 4 PM | `16:00` |
| 9:30 AM | `09:30` |

## AI Extraction Output

AI by Zapier returns these structured fields:

| Output Field | Format |
|---|---|
| `patient` | Text |
| `email` | Lowercase email |
| `phone` | Text |
| `doctor` | Standardized doctor value |
| `preferred_date` | `YYYY-MM-DD` |
| `preferred_time` | `HH:mm` |
| `reason` | Text |
| `booking_complete` | `Yes` or `No` |

`booking_complete` is `Yes` only when all seven fields exist and the patient confirmed the final summary.

## Appointment ID and Slot Generation

Code by Zapier validates the normalized date and time, generates a unique Appointment ID, creates the requested Slot Key, and calculates one-hour alternatives.

```javascript
const doctor = inputData.doctor?.trim();
const date = inputData.date?.trim();
const time = inputData.time?.trim();

if (!doctor || !date || !time) {
  throw new Error("Doctor, date and time are required.");
}

if (!/^\d{4}-\d{2}-\d{2}$/.test(date)) {
  throw new Error("Date must use YYYY-MM-DD format.");
}

const timeMatch = time.match(/^([01]\d|2[0-3]):([0-5]\d)$/);

if (!timeMatch) {
  throw new Error("Time must use HH:mm format.");
}

const hour = Number(timeMatch[1]);
const minute = Number(timeMatch[2]);

const formatTime = totalMinutes => {
  const normalized = ((totalMinutes % 1440) + 1440) % 1440;
  const hours = String(Math.floor(normalized / 60)).padStart(2, "0");
  const minutes = String(normalized % 60).padStart(2, "0");
  return `${hours}:${minutes}`;
};

const requestedMinutes = hour * 60 + minute;
const earlierTime = formatTime(requestedMinutes - 60);
const laterTime = formatTime(requestedMinutes + 60);

const randomCode = Math.random()
  .toString(36)
  .slice(2, 8)
  .toUpperCase();

return {
  appointment_id: `APT-${date.replace(/-/g, "")}-${randomCode}`,
  requested_time: time,
  requested_slot_key: `${doctor}|${date}|${time}`,
  earlier_time: earlierTime,
  later_time: laterTime
};
```

Example outputs for Dr. Sara at 4 PM:

```text
appointment_id: APT-20260915-A7K29Q
requested_time: 16:00
requested_slot_key: Dr. Sara - Dermatologist|2026-09-15|16:00
earlier_time: 15:00
later_time: 17:00
```

## Availability Search

Node: `Zapier Tables — Find Record`

The search uses four exact conditions:

| Lookup Field | Lookup Value |
|---|---|
| Doctor | AI output: Doctor |
| Date | AI output: Preferred Date |
| Time | AI output: Preferred Time |
| Status | `Booked` |

Additional settings:

```text
Successful if no results are found: True
If multiple results are found: Return first result
Create a record if it does not exist: Disabled
```

### Search Result Meaning

| `Zap Search Was Found` | Meaning |
|---|---|
| `true` | Slot is unavailable |
| `false` | Slot is available |

Searching Doctor, Date, Time, and Status directly avoids Slot Key formatting mismatches.

## Path A — Slot Unavailable

Condition:

```text
Zap Search Was Found = true
```

Actions:

- Do not create an appointment record.
- Send an unavailable-slot email through Gmail.
- Include the requested doctor, date, and time.
- Suggest one hour earlier and one hour later.
- Tell the patient to return to the chatbot and submit another time.

Example email:

```text
Subject: Your requested appointment is unavailable

Hello [Patient],

Unfortunately, your requested appointment with [Doctor] on [Date] at [Time]
is unavailable.

No appointment has been booked.

Would you like to try [Earlier Time] or [Later Time] instead? Please return to
the SmartCare Clinic chatbot and submit your preferred alternative time.

Regards,
SmartCare Clinic
```

## Path B — Slot Available

Condition:

```text
Zap Search Was Found = false
```

### Create Appointment Mapping

| Table Column | Source |
|---|---|
| Appointment ID | Code output: `appointment_id` |
| Patient | AI output: `patient` |
| Email | AI output: `email` |
| Phone | AI output: `phone` |
| Doctor | AI output: `doctor` |
| Date | AI output: `preferred_date` |
| Time | Code output: `requested_time` |
| Reason | AI output: `reason` |
| Status | Static value: `Booked` |
| Slot Key | Code output: `requested_slot_key` |

The Gmail confirmation step runs only after the table record is successfully created.

Example confirmation email:

```text
Subject: Your SmartCare Clinic appointment is confirmed

Hello [Patient],

Your appointment has been successfully booked.

Appointment ID: [Appointment ID]
Doctor: [Doctor]
Date: [Date]
Time: [Time]
Reason: [Reason]
Status: Booked

Regards,
SmartCare Clinic
```

## Double-Booking Prevention

The system prevents duplicates by searching for an existing record with the same:

```text
Doctor + Date + Time + Status Booked
```

If a match exists:

- The unavailable path runs.
- No second appointment record is created.
- The patient receives an unavailable-slot email.

If no match exists:

- The available path runs.
- Exactly one record is created.
- The patient receives a booking confirmation email.

## Test Cases

All test scenarios below were completed successfully.

### Test 1 — Complete Request in One Message

```text
Patient: Ayesha Khan
Email: Valid controlled test email
Phone: +92 301 1234567
Doctor: Dr. Sara
Date: September 22, 2026
Time: 2 PM
Reason: Skin rash
```

Expected and verified:

- All seven fields recognized.
- Complete summary displayed.
- Button hidden until `CONFIRM BOOKING`.

Status: **Passed**

### Test 2 — Partial Initial Request

```text
I want to see Dr. Ali tomorrow at 5 PM.
```

Expected and verified:

- Doctor, date, and time retained.
- Chatbot requested name, email, phone, and reason together.
- Final summary appeared only after missing fields were supplied.

Status: **Passed**

### Test 3 — Missing Email

Expected and verified:

- Chatbot asked only for the missing email.
- Final summary and button were withheld until email was supplied.

Status: **Passed**

### Test 4 — Invalid Email

Input:

```text
sara.gmail.com
```

Expected and verified:

- Invalid email rejected.
- Other valid appointment information retained.
- Patient asked to provide a valid email.

Status: **Passed**

### Test 5 — Information Correction

Expected and verified:

- Patient could change the time after seeing the summary.
- Only the requested field changed.
- Updated summary required a new confirmation.

Status: **Passed**

### Test 6 — Early Confirmation Attempt

Input:

```text
CONFIRM BOOKING
```

Expected and verified:

- Incomplete request was not accepted.
- Chatbot requested all missing fields.
- Booking button remained hidden.

Status: **Passed**

### Test 7 — Invalid Doctor

Input:

```text
Dr. Zain
```

Expected and verified:

- Invalid doctor rejected.
- Approved doctors displayed.
- No booking proceeded until a valid doctor was selected.

Status: **Passed**

### Test 8 — Available Appointment

Expected and verified:

- Availability search returned `false`.
- One appointment record was created.
- Appointment ID was populated.
- Status was `Booked`.
- Confirmation email was received.

Status: **Passed**

### Test 9 — Duplicate Appointment

Expected and verified:

- Same doctor, date, and time returned `true` from the search.
- No duplicate record was created.
- Unavailable-slot email was received.
- Confirmation email was not sent.

Status: **Passed**

### Test 10 — Relative Date and Time Conversion

Input:

```text
I want to see Dr. Ali tomorrow at 5 PM.
```

Expected and verified:

- `tomorrow` converted using the Asia/Karachi clinic date.
- `5 PM` converted to `17:00`.
- Doctor normalized to `Dr. Ali - Cardiologist`.

Status: **Passed**

## Platform Behaviour

Zapier Chatbot button results are not returned dynamically inside the same chatbot conversation. The implemented solution therefore uses Gmail for the final availability result:

- Available slot → confirmation email
- Unavailable slot → rejection and alternative-time email

The chatbot clearly informs the patient that the request is not confirmed until a confirmation email is received.

## Final Outcome

The completed system successfully:

- Collects and validates all required patient information.
- Processes explicit and relative dates.
- Normalizes doctors and appointment times.
- Generates unique Appointment IDs.
- Checks live availability in Zapier Tables.
- Prevents duplicate bookings.
- Creates only available appointments.
- Assigns `Booked` status automatically.
- Sends confirmation or unavailable-slot emails to the patient.

The project meets the core appointment-booking requirements while handling the native Zapier Chatbot response limitation through automated Gmail notifications.
