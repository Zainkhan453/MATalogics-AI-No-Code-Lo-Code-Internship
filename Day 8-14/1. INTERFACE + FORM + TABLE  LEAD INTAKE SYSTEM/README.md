# Task 1 - Sales Lead Intake System

Status: **Completed**

This project uses Zapier Interfaces, a lead form, Zapier Tables, Code by Zapier, Paths and Gmail to collect and score sales leads and notify the sales team when attention is required.

## Workflow

1. A lead submits the Sales Lead Intake form.
2. The Zap normalizes the submitted values and calculates a lead score.
3. The lead receives a generated ID in the format `LEAD-YYYY-NNNN`.
4. The score is classified as Hot, Warm or Cold.
5. A Path evaluates the priority/urgency and sends the configured Gmail notification.

## Scoring rules

| Input | Score |
|-------|------:|
| High / Medium / Low urgency | +30 / +20 / +10 |
| Budget above $5,000 | +30 |
| Budget from $1,000 to $5,000 | +20 |
| Budget below $1,000 | +10 |
| Referral source | +20 |
| LinkedIn source | +15 |

Classification: `70+ = Hot`, `40-69 = Warm`, and `below 40 = Cold`.

## Files

- [Zapier Form Link.md](Zapier%20Form%20Link.md) - public submission form
- `Workflow/` - exported Zap configuration with the Gmail recipient and app connection reference replaced by placeholders
- `Screenshots/` - publish-safe implementation evidence

## Import notes

After importing the workflow, reconnect the required Zapier apps, choose the correct Table/Form resources and replace `YOUR_SALES_TEAM_EMAIL` with the intended notification address. Connection credentials are not included in this repository.

Screenshots containing personal email addresses, phone numbers or test lead records remain local and are intentionally excluded by the repository `.gitignore`.
