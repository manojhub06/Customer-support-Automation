# Contributing Guide

This guide explains how to modify, extend, and improve the workflow for your needs.

---

## Before You Start

- Read [ARCHITECTURE.md](ARCHITECTURE.md) to understand how the workflow works
- Have a copy of the workflow running in your n8n instance
- Test changes in a **development environment** before deploying to production

---

## Common Modifications

### 1. Add a New Department

**Goal:** Route tickets to a new department (e.g., "HR" for employee issues)

**Steps:**

1. Create a new Slack channel in your workspace (e.g., `#hr-tickets`)
2. Get its channel ID

3. In n8n, find the **Switch** node:
   - Add a new condition: `Department == "HR"`
   - Create a new Slack node pointing to the #hr-tickets channel

4. Update the OpenAI prompt to recognize HR-related keywords:
   - Find the **Draft emails and summary** node
   - Edit the system message to include:
   ```
   - HR → Payroll issues, benefits, employee concerns, time off requests
   ```

5. Test with a sample ticket containing HR keywords

---

### 2. Change the AI Model

**Goal:** Use a different OpenAI model (faster, cheaper, or more capable)

**Steps:**

1. In the **Draft emails and summary** node, change the Model ID
2. Options:
   - `gpt-4.1-mini` → Faster, cheaper (current)
   - `gpt-4` → More capable, slower
   - `gpt-3.5-turbo` → Cheapest, basic tasks

3. Test with a complex ticket to ensure quality

**Note:** Switching models may require prompt adjustments for optimal results.

---

### 3. Add More Data Fields to Google Sheets

**Goal:** Track additional information (e.g., customer company, phone number)

**Steps:**

1. Update your Tally form to include new fields (e.g., "Company Name")

2. In n8n, find the **Edit Fields** node:
   - Add new field mappings for the Tally data

3. Update the **Append row in sheet** node:
   - Add new columns to Google Sheets
   - Map the new Edit Fields to the new columns

4. Optionally, update the OpenAI prompt to use the new data for better classifications

---

### 4. Change the Email Template

**Goal:** Customize the confirmation email response

**Steps:**

1. Find the **Draft emails and summary** node (OpenAI)

2. Locate the system prompt section that says:
   ```
   "Write a polite, professional customer reply."
   ```

3. Modify it to match your brand voice:
   ```
   "Write a friendly, casual customer reply. Use first names. 
    Include a reference to the customer's specific issue."
   ```

4. Test by submitting a ticket and reviewing the generated email

---

### 5. Add Priority-Based SLA Notifications

**Goal:** Route high-priority tickets to a separate channel or send urgent alerts

**Steps:**

1. Create a new Slack channel: `#urgent-tickets`

2. Add a new **IF** node after the **Append row in sheet** node:
   - Condition: `Priority == "High"`
   - Route to the urgent-tickets channel

3. Optionally, add a delay node if you want to send escalation reminders after X hours

---

### 6. Add Error Notifications

**Goal:** Get alerted if something breaks

**Steps:**

1. Add error handling to key nodes by using **Error Workflow** trigger

2. Create a simple error notification flow:
   - OpenAI API fails → Send Slack message to admin
   - Google Sheets update fails → Send email to admin

3. This helps catch issues before customers are impacted

---

### 7. Add a Feedback Loop

**Goal:** Let support teams update ticket status back in Google Sheets

**Steps:**

1. Add a new workflow that listens to Slack reactions (e.g., ✅ = "Resolved")

2. When a team reacts to a ticket notification:
   - Update the Google Sheets row with the new status
   - Optionally, send a follow-up email to the customer

**Note:** This requires more complex n8n logic using Slack triggers and conditional logic.

---

## Advanced Modifications

### Custom AI Prompt Tuning

The OpenAI node uses a detailed prompt. You can fine-tune it for your use case:

1. Find the **Draft emails and summary** node
2. Edit the responses section where it says:
   ```
   "Summarize this email in exactly 4 sections with these headers..."
   ```

3. Experiment with:
   - Adding examples of expected outputs
   - Specifying tone (professional, casual, technical)
   - Adding constraints (max length, specific format)

4. Test and iterate

---

### Conditional Routing Based on Custom Rules

Instead of just department, route based on:

- **Customer tier** (if stored in Google Sheets)
- **Multiple departments** (escalation chain)
- **Time of day** (route after-hours tickets to escalation)

Use the **Switch** or **IF** node with custom expressions to implement this.

---

### Integration with External Systems

Add support for:

- **Zendesk/Jira** — Export tickets to your ticketing system
- **Salesforce** — Log tickets against customer records
- **Discord** — Route to Discord channels instead of Slack
- **PagerDuty** — Escalate high-priority tickets to on-call engineers

Each requires adding a new node type and OAuth credential setup.

---

## Testing Your Changes

### Before Deploying

1. **Create a test Tally response** with your custom fields
2. **Run the workflow** manually and check each step:
   - Does Google Sheets update?
   - Does Slack get notified?
   - Does the email send correctly?
3. **Review the output** — Are summaries accurate? Is routing correct?
4. **Check edge cases** — What if a field is empty? What if email is invalid?

### During Deployment

1. **Activate the workflow gradually**:
   - Test with a small portion of traffic first
   - Monitor for errors in n8n logs
   - Have a rollback plan if something breaks

2. **Monitor the first 24 hours**:
   - Check Google Sheets for data quality
   - Review Slack notifications
   - Confirm emails are being sent

---

## Troubleshooting Common Issues

### Workflow triggers but produces errors

1. Check the n8n **Execution History**
2. Click the failed execution to see error details
3. Common causes:
   - Missing API key → Re-authenticate credential
   - Wrong column names → Verify Google Sheets headers
   - Rate limits → Wait before retrying, upgrade if needed

### Tickets not appearing in Google Sheets

1. Verify spreadsheet ID is correct in **Append row in sheet** node
2. Verify column headers match exactly (case-sensitive)
3. Check that the Tally form is submitting data

### Slack not getting notified

1. Verify channel IDs are correct (get them from Slack channel details)
2. Ensure n8n has permission to post in those channels
3. Check for typos in the Switch node conditions

### Emails not being sent

1. Verify Gmail OAuth is properly authenticated
2. Check for "Less secure apps" restrictions on your Gmail account
3. Monitor OpenAI rate limits (if API is overloaded)

---

## Best Practices

✅ **Do:**
- Test changes in a development environment first
- Document any custom modifications you make
- Version control your workflow exports
- Monitor n8n execution logs for errors
- Keep API credentials secure

❌ **Don't:**
- Modify the workflow without understanding each node
- Share API keys in Git repositories
- Deploy untested changes to production
- Ignore error notifications
- Modify Google Sheets column headers without updating n8n nodes

---

## Sharing Your Changes

If you've created an improvement:

1. Document what you changed and why
2. Test thoroughly
3. Export your workflow as JSON
4. Share in a pull request with:
   - Description of the change
   - Before/after behavior
   - Any new setup steps required

---

## Need Help?

- Review [ARCHITECTURE.md](ARCHITECTURE.md) to understand node dependencies
- Check [SETUP.md](SETUP.md) for credential setup issues
- Check n8n official docs: https://docs.n8n.io
- Review your OpenAI prompt for classification issues

---

Happy building! 🚀
