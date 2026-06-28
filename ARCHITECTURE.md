# Architecture

This document explains how the workflow is structured, how data flows through it, and what each node does.

---

## Workflow Overview

The workflow is triggered by a customer support ticket submission and processes it through multiple stages:

1. **Trigger** — Customer submits via Tally form
2. **Data Extraction** — Extract and clean ticket fields
3. **AI Processing** — Classify, prioritize, and generate content
4. **Data Storage** — Save ticket to Google Sheets
5. **Team Routing** — Send to correct Slack channel
6. **Customer Notification** — Email confirmation to customer

---

## Data Flow Diagram

```
Tally Form (Trigger)
    ↓
Edit Fields (Extract data)
    ↓
Draft emails and summary (OpenAI)
    ↓
Append row in sheet (Google Sheets)
    ↓
Switch (Department routing)
    ├→ Billing Team (Slack)
    ├→ Technical Team (Slack)
    ├→ Sales Team (Slack)
    └→ General Inquiry Team (Slack)
         ↓
      Send email (Gmail)
```

---

## Node Breakdown

### 1. Customer Ticket (Tally Trigger)

**Type:** Trigger  
**What it does:** Listens for new form submissions on your Tally form.

**Input:**
- Full Name (text)
- Email Address (email)
- Problem Description (text)

**Output:** Webhook payload with form data

**Configuration:**
- Form ID must match your Tally form

---

### 2. Edit Fields (Data Extraction)

**Type:** Set node  
**What it does:** Extracts form fields and creates a consistent data structure.

**Input:** Raw Tally form data

**Output:**
```json
{
  "ticket_ID": "1234567890",
  "Full name": "John Doe",
  "Email ID": "john@example.com",
  "Problem": "Unable to access my account"
}
```

**Why it exists:** Normalizes data so downstream nodes can reference it consistently.

---

### 3. Draft emails and summary (OpenAI)

**Type:** OpenAI LLM node  
**What it does:** Analyzes the ticket and returns structured JSON with:

- `ticket_id` — Unique identifier
- `department` — Billing, Technical, Sales, or General
- `priority` — High, Medium, or Low
- `summary` — Concise issue description
- `email_subject` — Professional subject line
- `draft_reply` — Customer confirmation email

**Input:**
- Customer name
- Email address
- Ticket ID
- Problem description

**Output:**
```json
{
  "ticket_id": "1234567890",
  "department": "Technical",
  "priority": "High",
  "summary": "User unable to log in to account",
  "email_subject": "We're Looking Into Your Login Issue",
  "draft_reply": "Hi John, thanks for reaching out. We're investigating your login issue and will update you within 2 hours."
}
```

**Prompt Logic:**
- **Department classification** based on keywords (payment → Billing, login → Technical, pricing → Sales)
- **Priority assignment** based on urgency (outage → High, feature request → Low)
- **Summary generation** using natural language understanding
- **Professional email composition** maintaining brand voice

---

### 4. Append row in sheet (Google Sheets)

**Type:** Google Sheets node  
**What it does:** Saves the complete ticket record to a spreadsheet for historical tracking and team reference.

**Input:** Ticket data from OpenAI

**Output:** New row appended to Google Sheets

**Columns saved:**
- ticket_id
- full name
- email ID
- Problem they face
- Department
- Priority
- Summary
- Email Subject
- Draft Reply

**Why it exists:** Creates a centralized database for auditing, analytics, and team review.

---

### 5. Switch (Routing Logic)

**Type:** Switch node  
**What it does:** Routes the ticket to the correct Slack channel based on the department classification.

**Routing Rules:**
- `Department == "Billing"` → Send to #billing-tickets
- `Department == "Technical"` → Send to #technical-tickets
- `Department == "Sales"` → Send to #sales-tickets
- `Department == "General"` → Send to #general-tickets

**Input:** Department value from OpenAI

**Output:** Routes to one of four paths (Slack channels)

---

### 6. Slack Notifications (4 Nodes)

**Type:** Slack node (× 4)  
**What it does:** Notifies the appropriate team channel with ticket details.

**Nodes:**
- Billing Team
- Technical Team
- Sales Team
- General Inquiry Team

**Input:** Ticket ID, Priority, Summary

**Output:** Message in Slack channel

**Example Slack Message:**
```
Ticket ID: 1234567890
Priority: High
Need to fixed: User unable to log in to account
```

**Why it exists:** Alerts teams immediately so they can start working on the issue.

---

### 7. Send email (Gmail)

**Type:** Gmail node  
**What it does:** Sends a professional confirmation email to the customer.

**Input:**
- Customer email address
- Email subject (from OpenAI)
- Email body (draft reply from OpenAI)

**Output:** Email sent to customer

**Example Email:**
```
To: john@example.com
Subject: We're Looking Into Your Login Issue

Hi John, thanks for reaching out. We're investigating your login issue 
and will update you within 2 hours.

Ticket ID: 1234567890
```

**Why it exists:** Confirms ticket receipt and sets customer expectations.

---

## Data Transformations

### Stage 1: Form Submission
```
Tally Input → ticket_ID, Full name, Email ID, Problem
```

### Stage 2: AI Processing
```
Ticket details → OpenAI → Classification + Content generation
```

### Stage 3: Storage & Routing
```
Processed ticket → Google Sheets (archived)
             → Slack (team notification)
             → Gmail (customer email)
```

---

## Department Classification Rules

The OpenAI prompt uses these heuristics:

| Department | Triggers | Example |
|------------|----------|---------|
| **Billing** | payment, invoice, refund, charge, subscription, credit card | "Why was I charged twice?" |
| **Technical** | login, bug, error, crash, access, password, feature not working | "I can't log in to my account" |
| **Sales** | pricing, demo, enterprise, plan, upgrade, features | "What's the cost of the Pro plan?" |
| **General** | Anything else | "Do you have a phone number?" |

---

## Priority Assignment Rules

| Priority | Triggers | Examples |
|----------|----------|----------|
| **High** | Outage, security, payment issue, account access lost | "Our entire team is locked out" |
| **Medium** | Feature bugs, performance issues, feature requests | "The report button is slow" |
| **Low** | General inquiries, documentation requests | "Do you have API docs?" |

---

## Error Handling

### What happens if something fails?

Currently, the workflow has **limited error handling**. Here's what you should monitor:

1. **OpenAI API fails** → Workflow pauses; check API key and rate limits
2. **Google Sheets update fails** → Ticket won't be logged; check spreadsheet ID
3. **Slack notification fails** → Team won't be notified; check channel IDs and permissions
4. **Email fails** → Customer won't get confirmation; check Gmail credentials

**Recommendation:** Add error notification nodes (email to admin) for production use.

---

## Performance Considerations

- **Workflow execution time:** 5-15 seconds per ticket
- **OpenAI API calls:** 1 per ticket
- **Google Sheets writes:** 1 per ticket
- **Slack messages:** 1 per ticket
- **Emails sent:** 1 per ticket

For high-volume scenarios (100+ tickets/hour), consider:
- Upgrading OpenAI to a faster model
- Implementing batch processing
- Adding rate limiting logic

---

## Customization Points

### Easy to change:

- **Department categories** — Modify the OpenAI prompt
- **Priority rules** — Update the prompt logic
- **Slack channels** — Swap channel IDs in Switch node
- **Email template** — Edit the draft_reply in the OpenAI prompt
- **Google Sheets columns** — Add/remove fields in the Append node

### Harder to change:

- **Trigger source** — Would need to replace Tally trigger with different form tool
- **AI model** — Requires changing OpenAI node and potentially prompt restructuring
- **Storage backend** — Would require replacing Google Sheets with different database

---

## Security Notes

- API keys are stored in n8n credentials (encrypted)
- Tickets are stored in Google Sheets (accessible to anyone with the link—keep it private)
- Slack messages visible to channel members only
- Emails sent via authenticated Gmail account
- No PII is logged beyond what's needed for ticket processing

---

## Next Steps

For setup instructions, see [SETUP.md](SETUP.md).  
For contribution guidelines, see [CONTRIBUTING.md](CONTRIBUTING.md).
