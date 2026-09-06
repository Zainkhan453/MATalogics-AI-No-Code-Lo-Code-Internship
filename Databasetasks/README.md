# SQL Database Development and AI Automation Tasks

This folder contains the completed MATalogics Day 14-17 database assignment. The work covers PostgreSQL fundamentals, SQL analytics, relational database design, n8n automation, Supabase synchronization, persistent AI memory, safe natural-language SQL, and automated reporting.

The original assignment is included as [`Day 14-17.pdf`](Day%2014-17.pdf).

## Completion Summary

| Task | Project | Status | Repository evidence |
|---|---|---|---|
| 1 | Lead Management Database | Completed | Details recorded below; screenshots were not retained |
| 2 | SQL Sales Analytics | Completed | Details recorded below; screenshots were not retained |
| 3 | AI Automation CRM Database Design | Completed | Details recorded below; screenshots were not retained |
| 4 | AI Customer Support Knowledge System | Completed | Details recorded below; screenshots were not retained |
| 5 | Lead Capture Automation | Completed and tested | [README](Task-5-Lead-Capture-Automation/README.md), [workflow folder](Task-5-Lead-Capture-Automation/) |
| 6 | AI Lead Qualification and Database | Completed and tested | [README](Task-6-AI-Lead-Qualification/README.md), [workflow folder](Task-6-AI-Lead-Qualification/) |
| 7 | AI Customer Conversation Memory | Completed and tested | [README](Task-7-AI-Customer-Conversation-Memory/README.md), [workflow folder](Task-7-AI-Customer-Conversation-Memory/) |
| 8 | Natural Language to SQL AI Agent | Completed and tested | [README](Task-8-Natural-Language-to-SQL-AI-Agent/README.md), [workflow folder](Task-8-Natural-Language-to-SQL-AI-Agent/) |
| 9 | Automated Database Reporting | Completed and tested | [README](Task-9-Automated-Database-Reporting/README.md), [workflow folder](Task-9-Automated-Database-Reporting/) |

## Tasks 1-4: Completed Without Screenshots

The first four tasks were completed properly, but their screenshots were not retained. The following record documents the work completed for each task without claiming that screenshot evidence is available in this repository.

### Task 1 - Lead Management Database

A PostgreSQL `leads` table was created with the required fields: `id`, `name`, `email`, `phone`, `company`, `source`, `lead_score`, `status`, and `created_at`.

The completed SQL work included:

- inserting at least 10 lead records;
- displaying all leads;
- filtering qualified leads;
- filtering leads with a score greater than 70;
- updating a lead's status;
- deleting a lead; and
- displaying the top five leads by `lead_score`.

**Status:** Completed properly. Screenshots are not available.

### Task 2 - SQL Sales Analytics

Analytics queries were completed against the `leads` table to calculate the total number of leads, counts by status, average lead score, highest-scoring lead, and lead totals grouped by source. A range query also displayed leads scoring between 50 and 80 in descending score order.

The work demonstrated `SELECT`, `WHERE`, `ORDER BY`, `LIMIT`, `COUNT()`, `AVG()`, `GROUP BY`, and `BETWEEN`.

**Status:** Completed properly. Screenshots are not available.

### Task 3 - AI Automation CRM Database Design

An ER-style relational design was completed for an AI-powered CRM receiving leads from website, Facebook, Instagram, LinkedIn, and WhatsApp sources. The design covered `customers`, `leads`, `conversations`, and `messages`, including columns, data types, primary keys, foreign keys, and the required one-to-many relationships.

The design supports AI-generated lead scores, statuses, and categories, along with multiple conversations and messages per customer.

**Status:** Completed properly. Screenshots are not available.

### Task 4 - AI Customer Support Knowledge System

A complete ER-style database design was completed for an AI customer-support system. It covered customers, support tickets, conversations, messages, products, orders, and FAQs, with primary keys, foreign keys, independent tables, connected tables, and one-to-many relationships identified.

The structure was designed to let an AI agent retrieve a customer's previous order and the reason a support ticket was opened before generating an answer.

**Status:** Completed properly. Screenshots are not available.

## Tasks 5-9: Evidence-Backed Submissions

Each remaining task includes a detailed Markdown report, an importable n8n workflow JSON export, SQL/schema documentation, test results, and screenshot evidence:

- **Task 5:** webhook lead capture, validation, PostgreSQL storage, Supabase synchronization, and HTTP success/error responses.
- **Task 6:** AI lead scoring, status/category assignment, PostgreSQL and Supabase persistence, validation, and qualified-lead notification.
- **Task 7:** persistent customer conversation memory, customer isolation, PostgreSQL history retrieval, AI context construction, and Supabase synchronization.
- **Task 8:** natural-language analytics, structured SQL generation, deterministic read-only enforcement, destructive-query rejection, PostgreSQL execution, and Supabase audit logging.
- **Task 9:** scheduled reporting, PostgreSQL metrics, AI business summary, Supabase report history, duplicate prevention, Gmail delivery, and delivery-status tracking.

## Repository Quality Checks

- All five n8n workflow exports are valid JSON.
- All local screenshot references in the Markdown reports resolve to existing files.
- The assignment PDF contains 15 readable pages and covers Tasks 1-9.
- No database passwords, secret API keys, OAuth tokens, or service-role keys are stored in the submitted text or workflow exports.
- Personal sender and recipient email addresses in public screenshot evidence have been redacted.
- Workflow notification recipients use the placeholder `manager@example.com` and must be configured after import.

## Import Note

After importing a workflow into n8n, configure your own PostgreSQL, Supabase, AI model, and Gmail credentials. For Tasks 6 and 9, replace `manager@example.com` with the intended notification recipient before activation.
