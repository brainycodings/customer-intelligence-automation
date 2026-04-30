# Customer Intelligence Automation System
> A fully automated churn detection and lifecycle pipeline built with n8n, HubSpot, and AI — inspired by real-world B2B SaaS customer success workflows.

---

##  Overview

This project simulates the customer data automation infrastructure of a B2B SaaS company. It automatically ingests customer data, scores engagement behavior, triggers personalized re-engagement actions, and generates weekly reports — all without manual intervention.

Built as a portfolio project to demonstrate end-to-end automation skills using modern no-code/low-code tools.

---

##  Architecture

```
Google Sheets (200 fictional customers)
        │
        ▼
[Workflow 1 — Ingestion]
n8n reads rows → transforms data → upserts contacts into HubSpot (nightly)
        │
        ▼
[Workflow 2 — Scoring]
Reads HubSpot contacts → calculates engagement score (0–100) → updates segments
        │
        ▼
[Workflow 3 — Lifecycle Actions]
Filters churn_risk contacts → generates re-engagement email → sends via Gmail
                                                            → posts alert to Slack
                                                            → creates ticket in HubSpot
        │
        ▼
[Workflow 4 — Weekly Reporting]
Aggregates all metrics → appends row to Google Sheets dashboard → sends email summary
```

---

##  Tech Stack

| Tool | Role |
|------|------|
| **n8n** | Workflow automation engine |
| **HubSpot** | CRM — contact storage & ticketing |
| **Google Sheets** | Data source + reporting dashboard |
| **Gmail** | Transactional emails |
| **Slack** | CS team alerts |
| **Groq API** | AI-generated re-engagement emails |


---

## 📊 Scoring Logic (Workflow 2)

Each contact receives an engagement score between 0 and 100 based on behavioral signals:

| Signal | Impact |
|--------|--------|
| Login in last 7 days | +30 pts |
| No login for 8–20 days | −20 pts |
| No login for 20+ days | −40 pts |
| 10+ actions in last 7 days | +20 pts |
| 0 actions in last 7 days | −20 pts |
| Pro or Enterprise plan | +15 pts |
| 3+ support tickets in 30 days | −15 pts |

**Segments:**
- `churn_risk` — score < 40
- `at_risk` — score 40–70
- `healthy` — score > 70

---

## Workflow Details

### Workflow 1 — Ingestion (runs at midnight)
- Reads 200 rows from Google Sheets
- Transforms column names and data types via JavaScript
- Upserts each contact into HubSpot with custom properties: `plan`, `mrr`, `days_inactive`, `actions_7d`

### Workflow 2 — Scoring (runs at 01:00am)
- Fetches all HubSpot contacts with custom properties
- Calculates engagement score using behavioral rules
- Updates `engagement_score` and `churn_segment` in HubSpot for each contact

### Workflow 3 — Lifecycle Actions (runs at 02:00am)
- Filters contacts where `churn_segment = churn_risk`
- Generates a personalized re-engagement email via AI (Groq)
- Sends email via Gmail
- Posts an alert to Slack `#churn-alerts` channel
- Creates a support ticket in HubSpot CRM

### Workflow 4 — Weekly Reporting (runs every Monday at 08:00am)
- Aggregates metrics across all contacts
- Appends a new row to the Google Sheets dashboard with: total contacts, segment breakdown, average score, total MRR, MRR at risk
- Sends a formatted weekly summary via Gmail

---

## Sample Weekly Report Output

```
📊 Weekly Report — 2026-05-05

👥 Total contacts : 200
🔴 Churn risk     : 47
🟡 At risk        : 68
🟢 Healthy        : 85

📈 Average score  : 54/100
💰 Total MRR      : 18 650€
⚠️  MRR at risk   : 4 200€
```

---

## Data Model

The dataset simulates 200 B2B SaaS customers with the following fields:

| Field | Type | Description |
|-------|------|-------------|
| `customer_id` | String | Unique identifier |
| `email` | String | Contact email |
| `plan` | Enum | free / starter / pro / enterprise |
| `mrr_eur` | Integer | Monthly recurring revenue |
| `last_login_date` | Date | Last platform login |
| `days_since_last_login` | Integer | Days since last login |
| `actions_last_7d` | Integer | In-app actions in last 7 days |
| `features_used_count` | Integer | Number of distinct features used |
| `support_tickets_30d` | Integer | Support tickets opened in 30 days |
| `nps_score` | Integer | NPS score (0–10, optional) |

---

##  How to Run

1. Clone this repo and import the workflow JSON files into your n8n instance
2. Set up credentials: HubSpot API key, Google Sheets OAuth2, Gmail OAuth2, Slack OAuth2, Groq API key
3. Create the following custom properties in HubSpot: `plan`, `mrr`, `days_inactive`, `actions_7d`, `engagement_score`, `churn_segment`
4. Import `saas_customers.csv` into Google Sheets
5. Activate workflows in order: W1 → W2 → W3 → W4
6. Execute manually to test, then let the schedules run automatically

---

##  Repository Structure

```
├── README.md
├── data/
│   └── saas_customers.csv          # 200 fictional SaaS customers
├── workflows/
│   ├── workflow_1_ingestion.json
│   ├── workflow_2_scoring.json
│   ├── workflow_3_lifecycle.json
│   └── workflow_4_reporting.json
└── screenshots/
    ├── workflow_1_overview.png
    ├── workflow_2_scoring_output.png
    ├── workflow_3_slack_alert.png
    └── workflow_4_dashboard.png
```

---

*This project is fictional and uses simulated data. No real customer data was used.*
    
