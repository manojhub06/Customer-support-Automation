# Setup Guide

Follow these steps to get the workflow up and running. Each section covers one integration.

## Prerequisites

Before starting, make sure you have:

- An **n8n account** (or self-hosted n8n instance)
- An **OpenAI API key**
- A **Tally account** with a support form created
- A **Google Sheets** spreadsheet (we'll create the structure)
- A **Slack workspace** with admin access
- A **Gmail account** for sending emails

---

## 1. OpenAI API Setup

### Get Your API Key

1. Go to [OpenAI API](https://platform.openai.com/)
2. Sign in with your account
3. Navigate to **API keys** → **Create new secret key**
4. Copy the key and save it safely (you won't see it again)
5. Keep the key handy—you'll need it in n8n

### Important Notes

- The workflow uses **gpt-4.1-mini** (adjust in n8n if you prefer a different model)
- Ensure your OpenAI account has API credits or a valid billing method
- Free trial credits work if you haven't exceeded the limit

---

## 2. Google Sheets Setup

### Create a New Spreadsheet

1. Go to [Google Sheets](https://sheets.google.com)
2. Click **+ New** → **Blank spreadsheet**
3. Name it: `Support Tickets Database`

### Create Column Headers

Add these headers in the first row (Row 1):

| Column | Header |
|--------|--------|
| A | ticket_id |
| B | full name |
| C | email ID |
| D | Problem they face |
| E | Department |
| F | Priority |
| G | Summary |
| H | Email Subject |
| I | Draft Reply |

### Get Your Spreadsheet ID

1. Look at the spreadsheet URL: `https://docs.google.com/spreadsheets/d/{SPREADSHEET_ID}/edit`
2. Copy the `{SPREADSHEET_ID}` part
3. Save it—you'll use this in n8n

---

## 3. Tally Form Setup

### Create a Support Form

1. Go to [Tally](https://tally.so)
2. Create a **new form**
3. Add these fields:

   - **Full Name** (Text) — Label: "Full Name"
   - **Email** (Email) — Label: "Email Address"
   - **Problem Description** (Long Text) — Label: "What's your issue?"

4. Click **Publish** and copy the form ID from the URL
5. Save the form ID—you'll need it in n8n

### Example Tally URL
```
https://tally.so/r/{FORM_ID}
```

The `{FORM_ID}` is what you need.

---

## 4. Slack Setup

### Create Slack Channels

Create 4 support channels in your Slack workspace:

1. **#billing-tickets** — For billing issues
2. **#technical-tickets** — For technical issues
3. **#sales-tickets** — For sales inquiries
4. **#general-tickets** — For general inquiries

### Get Channel IDs

For each channel:

1. Right-click the channel name
2. Select **View channel details**
3. Look for the channel ID (format: `C0BBKPXQWQG`)
4. Copy and save all 4 channel IDs

### Connect Slack to n8n

1. In n8n, add a **Slack** node
2. Click **Connect my account** → **Create new credential**
3. Click **Authenticate with Slack**
4. Authorize n8n to access your workspace
5. Save the credential

---

## 5. Gmail Setup

### Enable Gmail in n8n

1. In n8n, add a **Gmail** node
2. Click **Connect my account** → **Create new credential**
3. Click **Authenticate with Google** and sign in
4. Grant n8n permission to send emails
5. Save the credential

### Important

- Use a dedicated support email (e.g., `support@yourcompany.com`) if possible
- If using personal Gmail, be aware of rate limits
- Gmail's "Less secure apps" setting may need adjustment depending on your setup

---

## 6. n8n Workflow Import

### Import the Workflow

1. In n8n, click **+** → **Import from JSON**
2. Upload or paste the contents of `workflow.json`
3. n8n will automatically map the nodes

### Connect Credentials

After importing, n8n will show **missing credentials** warnings:

1. For each warning, click the node and select **Create new credential**
2. Enter your API keys and OAuth tokens:
   - **OpenAI API Key** → Your API key from Step 1
   - **Google Sheets** → Select your spreadsheet ID from Step 2
   - **Tally Form ID** → Your form ID from Step 3
   - **Slack Channel IDs** → Your 4 channel IDs from Step 4
   - **Gmail** → Your authenticated Gmail account

### Update Spreadsheet References

1. In the **Append row in sheet** node, ensure:
   - **Document ID** = Your Spreadsheet ID (from Step 2)
   - **Sheet Name** = `Sheet1` (or whatever your sheet is named)

### Update Tally Form ID

1. In the **Customer Ticket** trigger node:
   - **Form ID** = Your Tally Form ID (from Step 3)

### Update Slack Channel IDs

1. For each Slack node (Billing Team, Technical Team, Sales Team, General Inquiry Team):
   - **Channel ID** = Corresponding channel ID (from Step 4)

---

## 7. Environment Variables (.env)

Create a `.env.example` file with placeholders:

```
# OpenAI
OPENAI_API_KEY=YOUR_OPENAI_API_KEY

# Google Sheets
GOOGLE_SHEETS_SPREADSHEET_ID=YOUR_SPREADSHEET_ID

# Tally Form
TALLY_FORM_ID=YOUR_TALLY_FORM_ID

# Slack Channels
SLACK_BILLING_CHANNEL_ID=YOUR_BILLING_CHANNEL_ID
SLACK_TECHNICAL_CHANNEL_ID=YOUR_TECHNICAL_CHANNEL_ID
SLACK_SALES_CHANNEL_ID=YOUR_SALES_CHANNEL_ID
SLACK_GENERAL_CHANNEL_ID=YOUR_GENERAL_CHANNEL_ID

# Gmail
GMAIL_ACCOUNT=YOUR_GMAIL_EMAIL@gmail.com
```

---

## 8. Test the Workflow

### Submit a Test Ticket

1. Open your Tally form (the published link)
2. Fill in a test ticket with sample data
3. Submit the form

### Verify Each Step

1. **Check Google Sheets** — New row should appear with ticket data
2. **Check Slack** — Correct channel should receive a notification
3. **Check Gmail** — Test email address should receive a confirmation

### Troubleshooting

If something doesn't work:

- **Workflow didn't trigger** — Make sure the Tally trigger node is connected to your form
- **Slack notification not sent** — Verify channel IDs are correct and n8n has access
- **Email not sent** — Check Gmail credentials and rate limits
- **Missing ticket in Sheets** — Verify spreadsheet ID and column headers match the node configuration

---

## 9. Activate the Workflow

Once testing passes:

1. In n8n, click the **Workflow** toggle to **ON**
2. The workflow is now live and will process all incoming tickets
3. You can disable it anytime by toggling **OFF**

---

## Need Help?

- **OpenAI errors** — Check your API key and account balance
- **Slack connection issues** — Verify your workspace permissions
- **Google Sheets not updating** — Check spreadsheet ID and column names
- **Tally form not triggering** — Ensure the trigger node has the correct form ID

For more details on how the workflow flows, see [ARCHITECTURE.md](ARCHITECTURE.md).
