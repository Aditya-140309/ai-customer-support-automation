# AI-Powered Customer Support Automation Platform

**n8n Capstone Project — Summer School '26**
**Author:** Aditya Mane | MIS: 612302034 | B.Tech. Robotics and Artificial Intelligence, COEP Technological University, Pune

---

## Overview

This project automates the full customer support ticket lifecycle — from intake to resolution to feedback — using **5 interconnected n8n workflows**. It uses **Google Gemini** for AI-powered ticket classification, **Google Sheets** as the ticket database, and **Gmail** for all customer/agent notifications.

A support ticket flows automatically through classification, team assignment, human escalation (for critical issues), resolution tracking, and feedback collection — with zero manual triage.

📄 Full documentation: [`Documentation/Problem_Analysis_and_Documentation.pdf`](./Documentation/Problem_Analysis_and_Documentation.pdf)
🖼 Architecture diagram: [`Documentation/Architecture_Diagram.png`](./Documentation/02_Architecture_Diagram.png)
🎥 Demo video: **[link here]**

---

## System Architecture

```
Customer → Workflow 1 (Ticket Collection)
         → Workflow 2 (AI Classification — Gemini)
         → Workflow 3 (Assignment & Escalation)
         → Workflow 4 (Communication & Resolution)
         → Workflow 5 (Feedback & Analytics)
```

Workflows are chained using n8n's **Execute Sub-workflow** trigger/action pattern. See the architecture diagram for the full node-level view, including the Switch/IF branching logic and the three independent webhook/Cron entry points.

---

## Workflows

| # | Workflow | Trigger | What it does |
|---|---|---|---|
| 1 | **Ticket Collection & Registration** | Webhook (`POST /ticket-intake`) | Receives ticket, saves to Google Sheets, sends acknowledgment email |
| 2 | **AI Ticket Classification & Prioritization** | Execute Sub-workflow | Google Gemini classifies category + priority, updates the sheet |
| 3 | **Agent Assignment & Escalation** | Execute Sub-workflow | Routes ticket by category (Switch), escalates P1 tickets to a human via email (IF + approval step) |
| 4 | **Customer Communication & Resolution Tracking** | Execute Sub-workflow + Webhook (`POST /ticket-resolve`) | Sends assignment update; on resolution, sends closure email |
| 5 | **Feedback Collection & Support Analytics** | Execute Sub-workflow + Webhook (`POST /submit-feedback`) + Cron | Sends feedback survey, logs responses, emails a scheduled analytics report |

**Total:** ~35 nodes across 5 workflows (exceeds the 20–25 node / 4–6 workflow minimum).

---

## Advanced Features Implemented

- **AI-powered decision making** — Gemini classifies category & priority from ticket text (Workflow 2)
- **Human approval step** — Critical (P1) tickets trigger a manager escalation email before proceeding (Workflow 3)
- **Error handling** — Resolved data-loss issues caused by Gmail nodes overwriting payloads, by referencing upstream nodes explicitly (`$('NodeName').item.json`)
- **Logging & audit trail** — Every stage (status, category, priority, assigned team, timestamps) is logged to Google Sheets
- **Scheduled workflow (Cron)** — Weekly analytics report (Workflow 5)
- **Webhook-triggered workflows** — Ticket intake, resolution update, feedback submission
- **Conditional branching** — Switch node (category routing) + IF node (priority-based escalation)

---

## Tech Stack

- **Automation:** n8n (self-hosted via Docker, WSL2 / Ubuntu 24.04)
- **AI model:** Google Gemini (Google AI Studio API)
- **Database:** Google Sheets (`Support Tickets DB` — Sheet1 + Feedback tab)
- **Notifications:** Gmail (OAuth2)

---

## Repository Structure

```
ai-customer-support-automation/
├── README.md
├── 01_Documentation/
│   ├── Problem_Analysis_and_Documentation.pdf
│   └── Architecture_Diagram.png
├── 02_Workflows_JSON/
│   ├── 1_ticket_collection.json
│   ├── 2_ai_classification.json
│   ├── 3_agent_assignment.json
│   ├── 4_customer_communication.json
│   └── 5_feedback_analytics.json
├── 03_Workflow_Screenshots/
│   ├── Workflow_1_Ticket_Collection.png
│   ├── Workflow_2_AI_Classification.png
│   ├── Workflow_3_Agent_Assignment.png
│   ├── Workflow_4_Customer_Communication.png
│   └── Workflow_5_Feedback_Analytics.png
└── 04_Presentation/
    └── Capstone_Presentation.pptx
```

---

## How to Run This Locally

1. Self-host n8n (Docker):
   ```bash
   docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
   ```
2. Import each JSON file from `02_Workflows_JSON/` into n8n (Workflows → Import from File)
3. Set up credentials: Google Sheets OAuth2, Gmail OAuth2, Google Gemini API key
4. Create a Google Sheet named `Support Tickets DB` with a `Sheet1` tab (columns: `ticket_id, customer_email, subject, description, status, created_at, category, priority, assigned_team`) and a `Feedback` tab (columns: `ticket_id, customer_email, rating, comments, submitted_at`)
5. Publish/activate Workflows 2–5 (they're called by other workflows)
6. Trigger the pipeline:
   ```bash
   curl -X POST http://localhost:5678/webhook-test/ticket-intake \
     -H "Content-Type: application/json" \
     -d '{"email": "test@example.com", "subject": "Cannot login", "description": "Getting error 500"}'
   ```

---

## Author

**Aditya Mane**
B.Tech. Robotics and Artificial Intelligence, COEP Technological University, Pune
