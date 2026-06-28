# AI Customer Support Ticket Automation

Automate your entire customer support workflow. From ticket submission to team routing, everything happens in seconds.

## What It Does

A customer submits a support request through a **Tally Form**. The AI then automatically:

✅ **Classifies the ticket** — Billing, Technical, Sales, or General  
✅ **Assigns priority** — High, Medium, or Low  
✅ **Generates a summary** — Concise issue description  
✅ **Creates email subject** — Professional subject line  
✅ **Drafts a response** — Customer confirmation email  

The workflow then automatically:

📊 **Stores the ticket** — Logged in Google Sheets for tracking  
💬 **Routes to the right team** — Notifies the correct Slack channel  
📧 **Sends confirmation** — Customer receives email reply  

## Why This Matters

Instead of manually reading every support request, identifying the department, writing replies, updating spreadsheets, and notifying teams—the entire process is automated in seconds.

This workflow reduces manual work, improves response times, and lets your support team focus on solving problems instead of administrative tasks.

## ⚡ Tech Stack

- **n8n** — Workflow automation platform
- **OpenAI API** — AI classification and content generation
- **Tally Forms** — Customer ticket submission form
- **Google Sheets** — Centralized ticket database
- **Slack** — Team notifications
- **Gmail** — Email responses to customers

## 🚀 Quick Start

1. **Clone this repository**
2. Follow the [SETUP.md](SETUP.md) guide to connect your services
3. Import the workflow into n8n
4. Test with a sample ticket submission

## 📋 How It Works

1. Customer fills out the Tally form
2. n8n triggers and captures the submission
3. OpenAI analyzes and classifies the ticket
4. Ticket data is appended to Google Sheets
5. Slack notification is sent to the appropriate team channel
6. Confirmation email is sent to the customer

For a detailed breakdown, see [ARCHITECTURE.md](ARCHITECTURE.md).

## 📦 Files in This Repository

- **README.md** — Overview and features (you are here)
- **SETUP.md** — Step-by-step setup instructions
- **ARCHITECTURE.md** — Workflow structure and data flow
- **CONTRIBUTING.md** — Guidelines for modifications
- **LICENSE** — MIT License
- **workflow.json** — n8n workflow export (ready to import)
- **.env.example** — Placeholder for API keys and credentials

## 🔧 Prerequisites

Before you start, you'll need:

- An **n8n account** (self-hosted or cloud)
- An **OpenAI API key** (or compatible LLM)
- A **Tally account** and form (free tier works)
- A **Google Sheets** spreadsheet
- A **Slack workspace** with multiple channels
- A **Gmail account** for sending replies

## 📖 Documentation

- [SETUP.md](SETUP.md) — Detailed setup and credential configuration
- [ARCHITECTURE.md](ARCHITECTURE.md) — Technical workflow breakdown
- [CONTRIBUTING.md](CONTRIBUTING.md) — How to modify and extend the workflow

## 💡 Customization

This workflow is built to be flexible. You can:

- Modify department categories in the OpenAI prompt
- Change priority rules
- Add additional data fields to Google Sheets
- Route tickets to different Slack channels
- Adjust the email template for responses

See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## 📝 License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

## 🤝 Support

Questions or issues? Open an issue in this repository or check [SETUP.md](SETUP.md) for troubleshooting.

---

**Built with n8n | Powered by OpenAI**
