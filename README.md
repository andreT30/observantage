# OCI Cost Analytics & NL2SQL Chatbot  
**Oracle APEX • Oracle Autonomous Database • OCI FinOps**

---

## What this is

This repository contains a **production-grade Oracle APEX application** for **OCI cost analytics, reporting, and governance**, including an **explainable NL2SQL AI chatbot** that allows users to query cost and usage data using natural language.

The application is designed to be:
- accurate and auditable
- configuration-driven
- secure by design
- usable by both technical and non-technical users

It runs on **Oracle Autonomous Database (ADB)** with **Oracle APEX**, and integrates OCI Cost & Usage data, OCI resource metadata, and tagging.

---

## Who this is for

- **FinOps / Finance teams**  
  Understand spend, trends, credits, and chargeback.
- **Engineering & Operations**  
  Analyze workloads, services, resources, and usage.
- **Platform owners & Architects**  
  Govern OCI usage with transparency and control.
- **Executives & Stakeholders**  
  High-level visibility without technical complexity.

No SQL or Oracle expertise is required for end users.

---

## Key capabilities

### Cost & Usage Analytics
- Daily and monthly cost analysis
- Workload-level and resource-level attribution
- Trend analysis (MoM, QoQ, projections)
- PAYG and non-PAYG credit tracking

### Dashboards & Reports
- Executive overview dashboards
- Flexible cost and usage reports
- Drill-down from high level → workload → resource
- Saved and reusable reports

### OCI Resource Visibility
- Resource inventory across compartments and regions
- Parent–child resource relationships
- Tag-based attribution and filtering

### NL2SQL AI Chatbot
- Ask questions in natural language (e.g. *“cost last month by service”*)
- Deterministic, metadata-driven SQL generation
- Fully logged and explainable (no black box)
- Safe execution with guardrails

### Enterprise-Ready Design
- Database-centric logic
- Configuration over hardcoding
- No secrets in source control
- Full auditability

---

## High-level architecture

The application follows a **database-first, three-layer architecture**:

- **Presentation**: Oracle APEX (dashboards, reports, chatbot UI)
- **Logic**: PL/SQL packages (analytics, chatbot, jobs)
- **Data**: OCI cost usage, resources, relationships, configuration

📐 See the full architecture diagram in  
[`docs/architecture.md`](docs/architecture.md)

---

## How users interact with the app

Typical flow:
1. Start at the **Home dashboard** for an overview
2. Drill down via **Workloads**, **Cost Report**, or **Usage Report**
3. Use the **AI chatbot** when you don’t know where to start
4. Explore resources and tags via **Resource Explorer**

🧭 Page-by-page walkthrough:  
[`docs/usage-guide.md`](docs/usage-guide.md)

---

## NL2SQL chatbot (why it’s different)

This is **not** a free-form LLM guessing SQL.

The chatbot:
- uses explicit glossary rules and metadata
- only generates SQL against known datasets
- applies validation and execution limits
- logs every step (input → SQL → result)

🔍 See the full pipeline and diagrams:  
[`docs/chatbot.md`](docs/chatbot.md)

---

## Security & trust

- Authentication handled by Oracle APEX / IAM
- Role-based authorization for users and admins
- No OCI credentials or secrets in GitHub
- Deterministic SQL generation (no injection risk)
- Full logging and audit trail

🔐 Security model and trust boundaries:  
[`docs/security.md`](docs/security.md)

---

## Repository structure

```
apex/
f1200.sql             # Oracle APEX application export

db/
ddl/                  # Tables, views, packages, jobs
migrations/           # Bundle migration metadata

docs/
architecture.md
apex-pages.md
usage-guide.md
admin-guide.md
chatbot.md
configuration.md
data-model.md
deployment.md
troubleshooting.md
security.md
diagrams/
```


---

## Deployment overview

1. Deploy database objects (`db/ddl`)
2. Seed baseline configuration and chatbot metadata
3. Import the APEX application
4. Configure environment-specific values
5. Enable scheduled jobs

📦 Full instructions:  
[`docs/deployment.md`](docs/deployment.md)

---

## Configuration model

All behavior is driven by configuration tables (primarily `APP_CONFIG`):

- no hardcoded OCIDs
- no hardcoded tag keys
- no environment-specific logic in code

📘 Configuration reference:  
[`docs/configuration.md`](docs/configuration.md)

---

## Administration & operations

Admins can:
- manage workloads and subscriptions
- control data loads and refresh jobs
- extend chatbot vocabulary
- monitor logs and executions

🛠 Admin guide:  
[`docs/admin-guide.md`](docs/admin-guide.md)

---

## Why this application stands out

- **Explainable analytics** (no hidden logic)
- **Explainable AI** (NL2SQL with full traceability)
- **Enterprise governance** without sacrificing usability
- **Designed for real FinOps workflows**, not demos

---

## Status

This repository contains:
- production APEX application export
- full database schema and logic
- complete end-user, admin, and security documentation

It is ready for:
- onboarding new users
- audits and reviews
- long-term maintenance
- Git-based collaboration

---

## Next steps (optional)

- Add screenshots to the usage guide
- Add Mermaid diagrams to README sections
- Split public vs internal documentation
- Add `CHANGELOG.md` and `CONTRIBUTING.md`

---

**This documentation is part of the product.**  
If something is unclear, it should be documented—not hidden.
