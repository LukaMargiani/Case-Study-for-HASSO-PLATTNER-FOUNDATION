# Case-Study-for-HASSO-PLATTNER-FOUNDATION: Intelligent Inquiry Routing System for Donation Management  


---

##  Overview

This workflow automates the triage and handling of incoming emails for a foundation’s donation management system.  
Instead of manually reading and classifying every inquiry, the workflow leverages **AI classification** to determine the type of email and route it to the appropriate action automatically.

The documented workflow segment begins with the **Email Trigger (IMAP)**, processes the message through the **AI Triage** (OpenAI GPT-4o), passes through a **Switch** node for logic routing, and concludes with two possible actions:  
1. **Bot Reply** — sends an acknowledgment to new applicants.  
2. **Log Data** — stores structured results for all emails.

This design eliminates manual processing and demonstrates a scalable, intelligent automation approach for email management.

---

##  Architecture and Workflow Design

###  End-to-End Flow

1. **Email Trigger (IMAP)**  
   - Watches a dedicated mailbox (e.g., `foundation@domain.com`) for new, unread messages.  
   - Each new email triggers the workflow and passes relevant fields (From, To, Subject, Body) downstream.

2. **AI Triage (OpenAI GPT-4o)**  
   - Uses a Large Language Model (LLM) to:
     - Classify the email into one of four categories:  
       **New Application**, **Status Update**, **General Question**, or **Irrelevant**.  
     - Extract entities: **Organization Name**, **Contact Person**, and **Project Title**.  
   - Returns a **strictly valid JSON object** in the following format:
     ```json
     {
       "category": "",
       "organization_name": "",
       "contact_person": "",
       "project_title": "",
       "email_from": "",
       "email_to": "",
       "email_subject": "",
       "email_body": ""
     }
     ```

3. **Switch (Routing Logic)**  
   - Evaluates the AI output field `message.content.category`.  
   - Routes as follows:
     - If `"New Application"` → send **Bot Reply** and **Log Data**.  
     - For all other categories → only **Log Data**.

4. **Bot Reply (Auto-Acknowledgment)**  
   - Sends a personalized reply back to the applicant confirming receipt.  
   - Example template:
     > Dear [Contact Person],  
     >  
     > Thank you for submitting your application for [Project Title] with [Organization Name].  
     > You can track your application here: [link to official portal].  
     >  
     > Best regards,  
     > Recruiting Team  
   - Uses the **SMTP** credentials to send from the foundation’s official address.

5. **Log Data (PostgreSQL)**  
   - Records every processed email and extracted information in the table `email_triage_log`.  
   - Stored fields include:  
     `email_from`, `email_to`, `subject`, `body`, `category`, `organization`, `contact_person`, and `project_title`.

---

## Architectural Choices and Rationale

| Component | Purpose | Reason for Selection |
|------------|----------|----------------------|
| **IMAP Trigger** | Monitors inbox for incoming messages | Native, reliable integration with standard mail servers |
| **OpenAI GPT-4o** | Performs classification and entity extraction | High-accuracy model with natural language understanding |
| **Switch Node** | Controls workflow branching | Simplifies conditional logic within n8n |
| **SMTP Node (Bot Reply)** | Sends automated acknowledgment emails | Demonstrates end-to-end automation |
| **PostgreSQL Node (Log Data)** | Persists structured results | Enables reporting, analytics, and auditability |

All nodes operate within a **self-hosted n8n** environment to maintain data privacy and full control over workflow logic.

---

##  Setup Instructions

### 1. Prerequisites

- A running **self-hosted n8n instance** (e.g., on **Azure VM**, **App Service**, or **Docker**).  
- Credentials for:
  - IMAP mailbox (for incoming emails)
  - SMTP account (for outbound messages)
  - OpenAI API (GPT-4o access)
  - PostgreSQL database

### 2. Import the Workflow

1. Log in to your n8n instance:  
   `https://<YOUR_INSTANCE_URL>`
2. Click **Import Workflow** → upload `workflow.json`.  
3. Verify that the sequence is:  
   `Email Trigger (IMAP)` → `AI Triage` → `Switch` → (`Bot Reply`, `Log Data`).

### 3. Configure Credentials

| Node | Credential Type | Example |
|------|------------------|---------|
| IMAP | Email (IMAP) | `foundation@domain.com` |
| SMTP | Email (SMTP) | `foundation@domain.com` |
| OpenAI API | OpenAI Key | `sk-...` |
| PostgreSQL | Database Connection | `postgres://user:password@host:5432/dbname` |

### 4. Database Setup

Before running the workflow, create the following table in your database:

 ```sql
CREATE TABLE public.email_triage_log (
  id SERIAL PRIMARY KEY,
  received_at TIMESTAMP DEFAULT NOW(),
  email_from TEXT,
  email_to TEXT,
  subject TEXT,
  body TEXT,
  category TEXT,
  organization TEXT,
  contact_person TEXT,
  project_title TEXT
);
```

## 5. Run the Workflow
Activate the workflow in n8n.
Send a test email to the monitored inbox.
Observe the following behavior:
Email is fetched via IMAP.
AI Triage classifies and extracts key fields.
Switch routes based on category.
New Application → triggers acknowledgment email.
All cases → log entry created in PostgreSQL.

## Assumptions
Email Input: Plaintext messages; HTML parsing not required.
AI Output: Always valid JSON adhering to the provided schema.
LLM Temperature: Set to 0 for deterministic classification.
Infrastructure: n8n instance has network access to IMAP, SMTP, and PostgreSQL endpoints.
Table Exists: email_triage_log table is pre-created.
Acknowledgment Text: Can be freely modified within the Bot Reply node.

## Contact & References
Author: Luka Margiani
Email: lukamarg@gmail.com

### References
[official n8n documentation](https://docs.n8n.io/)
[OpenAI API Reference](https://platform.openai.com/docs/api-reference)
[PostgreSQL Documentation](https://www.postgresql.org/docs/)
[IMAP & SMTP Setup Guide](https://docs.n8n.io/integrations/builtin/email/)
