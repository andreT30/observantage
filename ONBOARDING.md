# Architecture

Home: [README](../README.md) · **Docs** · **Architecture**

## Overview
The application follows a **three-layer architecture**:

1. **Presentation Layer (Oracle APEX)**
2. **Application & Analytics Layer (PL/SQL)**
3. **Data & Integration Layer (OCI Cost & Resource Data)**

The design emphasizes:
- database-centric logic
- configuration-driven behavior
- environment portability
- traceability and logging

## Architecture Diagram

The following diagram shows the high-level system architecture and separation of concerns.

===mermaid
flowchart TB
  U[End Users] -->|Browser| APEX[Oracle APEX App]

  subgraph Presentation
    APEX --> P1[Dashboards & Reports]
    APEX --> P2[Admin UI]
    APEX --> P3[NL2SQL Chatbot UI]
  end

  subgraph Database
    PL[PL/SQL Packages]
    V[Views]
    CFG[APP_CONFIG]
    LOG[Logging Tables]
    META[Chatbot Metadata]
  end

  APEX -->|SQL / AJAX| V
  APEX -->|Procedures| PL

  PL --> CFG
  PL --> META
  PL --> LOG

  subgraph Data
    COST[OCI Cost Usage]
    RES[OCI Resources]
    REL[Resource Relationships]
  end

  PL --> COST
  PL --> RES
  PL --> REL
===

- Oracle APEX provides the presentation layer
- All business logic resides in the database
- Configuration and chatbot behavior are data-driven

---

## 1. Presentation Layer (Oracle APEX)

### Responsibilities
- Dashboards and charts for cost analytics
- Interactive reports and drilldowns
- Administration and configuration UI
- Chatbot user interface

### Characteristics
- No business logic embedded in pages
- SQL queries primarily reference views or packaged functions
- Centralized theming and shared components
- Logging and execution delegated to DB packages

### Key Page Groups
- Cost overview dashboards
- Resource and workload analytics
- Tag and filter-driven reports
- Chatbot page (NL input → results + explanation)
- Admin/configuration pages

---

## 2. Application & Analytics Layer (PL/SQL)

This layer contains the **core intelligence** of the system.

### Main Responsibilities
- Cost aggregation and normalization
- Resource-to-cost correlation
- Tag extraction and mapping
- NL2SQL chatbot pipeline
- Job orchestration and logging

### Major Package Categories
- **Cost ingestion & refresh**
  - Time-series refresh procedures
  - Monthly/daily aggregation
- **Analytics & reporting**
  - Pivoting, trend analysis, comparisons
- **Configuration & utilities**
  - APP_CONFIG accessors
  - JSON helpers, logging helpers
- **Chatbot (NL2SQL)**
  - Intent routing
  - Glossary-based SQL generation
  - Execution and summarization

All logic is implemented in PL/SQL and exposed to APEX via views and packaged APIs.

---

## 3. Data & Integration Layer

### Core Data Domains
- **Cost usage time series**
  - Daily and monthly OCI cost data
- **OCI resources**
  - Resource identifiers, regions, compartments
- **Relationships**
  - Parent–child resource mappings
- **Tags**
  - Defined and freeform tags
  - Normalized via configuration
- **Configuration**
  - Application behavior driven by data
- **Logging**
  - Job runs, chatbot requests, execution traces

### External Inputs
- OCI Cost & Usage reports
- OCI resource inventory
- OCI tags (non-uniform across tenancies)

---

## NL2SQL Chatbot Architecture (High Level)

1. User enters natural language question in APEX
2. Intent is inferred using glossary and rules
3. Relevant tables, metrics, and filters are selected
4. SQL is generated dynamically
5. SQL is executed safely
6. Results are summarized and returned
7. Full trace is logged for debugging and auditing

Details: [chatbot.md](chatbot.md)

---

## Design Principles
- **Database-first**: logic close to data
- **Config over code**: behavior driven by tables
- **Traceability**: everything logged
- **Environment portability**: no hardcoded OCIDs or secrets
- **Extensibility**: new datasets and chatbot rules without code changes

---

## Operational View
- Scheduled jobs refresh and aggregate data
- APEX pages remain read-only consumers
- Failures are diagnosable via run IDs and logs
- Deployment is repeatable via SQL + APEX import

---

## Cost Data Flow

This diagram illustrates how OCI cost data flows from ingestion to dashboards.

===mermaid
flowchart LR
  OCI[OCI Cost & Usage Reports] --> J1[Scheduled Jobs]

  J1 --> RAW[Raw / Staging Tables]
  RAW --> AGG[Aggregation & Normalization]
  AGG --> FACT[Cost Time-Series Tables]

  FACT --> V[Analytics Views]
  V --> APEX[Dashboards & Reports]

  CFG[APP_CONFIG] --> J1
  CFG --> AGG
===

Key points:
- Data is refreshed via scheduled jobs
- Aggregations and normalization occur centrally
- Dashboards and chatbot read from the same facts

**See also**
- [Data Model](data-model.md)
- [Deployment Guide](deployment.md)
- [Security Model](security.md)


# Infrastructure Requirements

Home: [README](../README.md) · Docs · Infrastructure Requirements

---

## Purpose

This document defines the mandatory infrastructure prerequisites that must exist
before deploying the application.

It is intentionally separated from the deployment procedure to:
- reduce operational risk
- support security and audit reviews
- clearly define infrastructure responsibility boundaries

No deployment or update steps are described here.

---

## High-Level Requirements

Before deployment, the following must already exist:

- An existing Oracle Cloud Infrastructure (OCI) tenancy
- An existing OCI Administrator account
- A dedicated OCI compartment for the deployment
- An Autonomous Database suitable for analytics and batch processing
- OCI IAM configuration (Dynamic Groups and Policies)
- OCI GenAI service availability (for chatbot functionality)

---

## OCI Tenancy

- The application is deployed into an existing OCI tenancy
- All access control is enforced via OCI IAM
- The application uses OCI Resource Principal only
- No credentials, API keys, passwords, or secrets are used or stored

---

## OCI Administrator Account

An OCI user with administrator privileges is required to:

- create compartments
- create Autonomous Databases
- manage IAM Dynamic Groups
- create and manage IAM policies

This account is required only for initial infrastructure setup.

---

## Deployment Compartment

Create or select a dedicated OCI compartment for the deployment.

This compartment will contain:
- the Autonomous Database
- Object Storage buckets
- all application-managed resources

Make a note of the Compartment OCID.
This value is required in multiple IAM policy definitions.

---

## Autonomous Database (ADB)

### Required Database Type

Autonomous AI Lakehouse (preferred)
(formerly Autonomous Data Warehouse)

This application is designed for analytics-heavy and batch-oriented workloads.

---

### Database Version Preference

For **new Autonomous Database deployments**, **Oracle Autonomous Database 26ai**
(or the latest AI-enabled version available) is **strongly recommended**.

Reasons include:
- native JSON data type and enhanced JSON querying
- improved SQL and PL/SQL support for JSON-centric workloads
- built-in vector capabilities, enabling future enhancements
  (e.g. semantic search, embeddings, and AI-assisted analytics)
- continued alignment with Oracle’s AI-enabled database roadmap

Earlier Autonomous Database versions remain supported, but newer AI-enabled
versions provide a more future-proof foundation.

---

### Why Autonomous Database (ADB)

The application is intentionally designed to run on **Oracle Autonomous Database**.

Key reasons include:

- **Fully managed by Oracle**
  - No operating system or database patching
  - No manual backups
  - No infrastructure maintenance overhead
  - Allows teams to focus on analytics and application logic

- **DBMS_CLOUD is pre-installed and pre-configured**
  - Native access to Object Storage
  - Secure, policy-driven access to OCI services
  - No external credentials or custom integrations required

- **Seamless OCI integration**
  - Native support for OCI Resource Principal
  - Direct interaction with OCI services governed by IAM policies
  - Required by the application for:
    - cost and usage report ingestion
    - resource discovery
    - object storage access
    - Generative AI integration

This tight integration with OCI services is a **core architectural requirement**
and is not achievable with self-managed or non-autonomous databases.


---

### Database Requirements

- APEX must be enabled
- The database must run in the selected deployment compartment
- The database must support OCI Resource Principal

Make a note of the Autonomous Database OCID.
This value is required for Dynamic Group configuration.

---

## OCI IAM – Dynamic Group

Create a Dynamic Group in the Default identity domain with the following properties.

Name:
===
focus-reports-ADW-DG
===

Rule:
===
resource.id = 'ocid1.autonomousdatabase.oc1.eu-frankfurt-1....'
===

Replace the OCID above with the Autonomous Database OCID created earlier.

This Dynamic Group represents the Autonomous Database identity
when using OCI Resource Principal.

---

## OCI IAM – Root Compartment Policy

Create a tenancy-level (root compartment) IAM policy.

Name:
===
focus-reports-root-policy
===

---

Policy Rules:

===
define tenancy usage-report as ocid1.tenancy.oc1..aaaaaaaaned4fkpkisbwjlr56u7cj63lf3wffbilvqknstgtvzub7vhqkggq

endorse dynamic-group focus-reports-DG to read objects in tenancy usage-report
endorse dynamic-group focus-reports-ADW-DG to read objects in tenancy usage-report

Allow dynamic-group focus-reports-ADW-DG to inspect compartments in tenancy
Allow dynamic-group focus-reports-ADW-DG to inspect tenancies in tenancy

Allow dynamic-group focus-reports-ADW-DG to read autonomous-databases in compartment id ocid1.compartment.oc1....
Allow dynamic-group focus-reports-ADW-DG to read secret-bundles in compartment id ocid1.compartment.oc1....

Allow dynamic-group focus-reports-ADW-DG to read all-resources in tenancy
Allow dynamic-group focus-reports-ADW-DG to manage object-family in compartment id ocid1.compartment.oc1....
===

Replace ocid1.compartment.oc1.... with the deployment compartment OCID.

Reference (Usage Reports Tenancy Definition):
https://docs.oracle.com/en-us/iaas/Content/Billing/Concepts/costusagereportsoverview.htm

---

## OCI IAM – Compartment-Level Policy

Create an IAM policy at the deployment compartment level.

Name:
===
focus-reports-compartment-policy
===

Policy Rules:

===
Allow dynamic-group focus-reports-ADW-DG to manage generative-ai-family in compartment id ocid1.compartment.oc1....
Allow dynamic-group focus-reports-ADW-DG to manage genai-agent-family in compartment id ocid1.compartment.oc1....

Allow dynamic-group focus-reports-ADW-DG to use autonomous-databases in compartment id ocid1.compartment.oc1....
Allow dynamic-group focus-reports-ADW-DG to manage object-family in compartment id ocid1.compartment.oc1....
===

Replace ocid1.compartment.oc1.... with the deployment compartment OCID.

---

## OCI GenAI Requirements (Chatbot)

For the AI chatbot functionality to work:

- The OCI tenancy must be subscribed to a region where OCI Generative AI is available
- The Autonomous Database must be able to access GenAI services via Resource Principal

---

### Tested Models

The following GenAI models have been tested:
- Cohere Command-A
- Grok-4

If GenAI is not available:
- chatbot functionality will not operate
- all other application features remain functional

---

## Explicit Non-Requirements

The application does not require:
- OCI API keys
- OCI Vault secrets
- passwords or shared credentials
- external secret stores
- cross-tenancy access
- cross-environment access

---

## Environment Model

Each installation is a standalone environment:
- one OCI tenancy context
- one Autonomous Database
- one APEX application
- no shared state with other environments

Isolation is enforced by OCI tenancy boundaries and IAM policies.

---

## Summary

All security, access, and scope control are enforced by:
- OCI IAM
- Dynamic Groups
- OCI policies
- Resource Principal identity

The application itself does not bypass or weaken OCI security controls.




# Security & Trust Model

This document describes how the application enforces security, privacy, and trust.
It is written for:
- security reviewers
- architects
- auditors
- administrators

The security model is **cloud-native, policy-driven, and auditable**.

---

## Security Principles

The application is built on the following principles:

- **No secrets**
- **Least privilege**
- **Policy-driven access**
- **No PII exposure to AI services**
- **Database-enforced controls**
- **Full auditability**

Security is enforced primarily by **OCI IAM policies**, not application logic.

---

## Authentication

Authentication is handled by **Oracle APEX** and supports:

- **Oracle APEX Accounts**
- **OCI Single Sign-On (SSO)**, including:
  - OCI IAM
  - Federated identity providers
  - Enterprise SSO integrations

The application does **not** implement custom authentication mechanisms.

---

## Authorization Model

### Roles

The application currently defines **two roles only**:

- **End Users**
  - dashboards
  - reports
  - chatbot usage

- **Administrators**
  - configuration
  - job control
  - chatbot metadata
  - update workflows

Authorization is enforced via:
- APEX authorization schemes
- role-based access at page and component level

---

## OCI Resource Principal (No Credentials)

### Identity Model

The database uses **OCI Resource Principal** (`OCI$RESOURCE_PRINCIPAL`) exclusively.

- No API keys
- No passwords
- No stored credentials
- No secrets in code or configuration

All OCI access originates from the database **as an OCI resource**.

---

### Dynamic Groups & Policies

Access to OCI services and resources is controlled by:

- **OCI Dynamic Groups**
- **OCI IAM Policies**

This ensures:
- central enforcement
- full auditability in OCI
- zero credential leakage risk

The application itself cannot bypass these policies.

---

## Compartment & Scope Control

OCI data visibility is constrained by:

- Dynamic Group membership
- OCI IAM policies
- configured root compartments

The application can only access:
- compartments
- regions
- services
explicitly allowed by OCI policy.

There is **no application-level override** of OCI scope.

---

## PII Protection and LLM Safety

The application enforces a strict **no-PII-to-LLM** policy.

### Tokenization model

- All PII handling occurs **inside the database**
- Sensitive values are **tokenized before any LLM interaction**
- Tokenization is irreversible outside the database context

### LLM interaction

- LLMs receive **tokenized placeholders only**
- No raw PII, secrets, or personal data are transmitted
- External AI services never see real identifiers or values

### Response rendering

- LLM responses return tokenized placeholders
- De-tokenization occurs **inside the database**
- Only the final UI output contains resolved values

### Security guarantee

- **PII is never sent to any LLM**
- Tokenization is deterministic and auditable
- Privacy policies are enforced by design, not convention

---

## Chatbot Security (NL2SQL)

The chatbot is **deterministic and constrained**:

- SQL is generated only from predefined metadata
- Allowed tables, columns, and filters are whitelisted
- Free-form SQL execution is not possible

Additional safeguards:
- execution limits
- time-range constraints
- full SQL logging

Every chatbot request is traceable end-to-end.

---

## Secrets Management

There are **no secrets** in this system.

Specifically:
- no OCI API keys
- no OAuth secrets
- no passwords
- no tokens stored in tables or config
- no secrets in GitHub

Identity and access are fully delegated to OCI IAM.

---

## Logging & Auditability

The system logs:
- authentication events (via APEX / OCI)
- deployment and update runs
- scheduler job executions
- chatbot requests and generated SQL
- errors and warnings

Logs are:
- queryable
- correlated by run/request IDs
- suitable for audits and incident analysis

---

## Environment Model

Each installation is a **standalone environment**:

- one database
- one APEX application
- one OCI tenancy context

There is:
- no shared state
- no cross-environment access
- no implicit trust between environments

Isolation is inherent by deployment model.

---

## Common Security Questions

### “Does the app store or transmit OCI credentials?”
No. It uses OCI Resource Principal exclusively.

---

### “Can the LLM see personal data?”
No. All PII is tokenized before any LLM call.

---

### “Can admins bypass OCI policies?”
No. OCI IAM policies are authoritative.

---

### “Is this auditable?”
Yes. All actions and data paths are logged and explainable.

---

## Final Statement

This application uses **OCI-native security controls**, avoids secret-based access,
and enforces privacy by design.

Trust is not assumed — it is **provable**.

**See also**
- [Admin Guide](admin-guide.md)
- [Deployment Guide](deployment.md)


# Deployment

Home: [README](../README.md) · **Docs** · **Deployment**

## Overview

This application is deployed using a **database-resident Deployment Manager**
implemented in PL/SQL (`DEPLOY_MGR_PKG`).

The Deployment Manager consumes the **application bundle ZIP as a ZIP (stored as a BLOB)** and applies
its contents directly—**no manual extraction is required or expected**.

This document covers **initial deployment only**.
Application updates are handled **inside the application itself** and are documented separately.

---

## Deployment Components

### 1. Admin prerequisite script (mandatory)
**File:** `ov_run_as_admin.sql`  
**Executed as:** Autonomous Database ADMIN (or equivalent privileged user)

Purpose:
- grant required system and object privileges
- enable scheduler, APEX import, and DBMS package access

This script must be executed **before the first deployment** in each environment.

---

### 2. Deployment Manager installation (application schema)

The Deployment Manager is installed in the **application schema** using:

- `deploy_manager_ddl.sql`
- `deploy_manager_pkg.sql`

Run as the application schema owner:

@deploy_manager_ddl.sql  
@deploy_manager_pkg.sql

These scripts create:
- bundle storage tables (ZIP stored as BLOB)
- deployment run and log tables
- the `DEPLOY_MGR_PKG` API

---

## Bundle ZIP

The bundle ZIP is the **single deployment artifact**.

Typical contents:
- `db/ddl/*.sql` — database objects (tables, views, packages, procedures)
- `db/ddl/90_jobs.sql` — scheduler jobs created as part of the application
- `apex/f1200.sql` — Oracle APEX application export
- `manifest.json` — bundle metadata

The Deployment Manager reads and processes these entries **directly from the ZIP**.

---

## Deployment Procedure (Target ADB)

### Step 0 — Run admin script
Connect as ADMIN and run:

@ov_run_as_admin.sql

---

### Step 1 — Install Deployment Manager
Connect as the application schema owner and run:

@deploy_manager_ddl.sql  
@deploy_manager_pkg.sql

---

### Step 2 — Upload bundle ZIP

MANIFSET_JSON example:
=== json
{
  "app_id": 1200,
  "version": "v20260109_134227",
  "schema": "ORDS_PLSQL_GATEWAY",
  "created_at": "2026-01-09T13:42:30.742+00:00",
  "notes": "Credentials are not exported; ensure required credentials exist per environment."
}
===

#### - SQL Developer 
Insert the bundle ZIP into the Deployment Manager bundle table
(`DEPLOY_BUNDLES`) and capture the generated `BUNDLE_ID`.

#### - SQLcl (either installed locally or as part of SQL Developer in VSCode)
Create the below script locally:
===sql
INSERT INTO deploy_bundles (
    app_id,
    version_tag,
    manifest_json,
    bundle_zip,
    sha256
)
VALUES (
    :app_id,
    :version_tag,
    :manifest_json,
    EMPTY_BLOB(),
    :sha256
)
RETURNING bundle_zip INTO :blob;
===

Launch SQLcl inside the same directory as the script above .
Version Tag, check github version update.
Manifest JSON, check example above and update accordingly.

Sample execution:
===sql
sql demo_user/demo_pwd@db_high <<EOF
VAR blob BLOB
EXEC :app_id := 1200
EXEC :version_tag := 'v20260109_134227'
EXEC :manifest_json := '{"app":"focus-reports","version":"v20260109_134227"}'

@insert_bundle.sql
LOAD BLOB :blob FROM FILE 'bundle_app1200.zip'
COMMIT
EOF
===


---

### Step 3 — Deploy (scheduled job only)
Initial deployment is performed **via scheduler job**, not synchronously. 
- $\color{#ff7b72}{\textsf{BUNDLE ID}}$ - Is the chosen ID during upload in Step 2
- $\color{#ff7b72}{\textsf{INITIAL}}$ - for new installations
- $\color{#ff7b72}{\textsf{YOUR WORKSPACE}}$ - APEX target workspace name
- $\color{#ff7b72}{\textsf{APP ID}}$ - target APEX id (default is 1200)
- $\color{#ff7b72}{\textsf{USE ID OFFSET}}$ - in cases where this APP is already installed at least once under the same APEX Instance, you must set it to 'Y' to avoid conflict with existing APEX app components. If this is the first and only app installation, set it to 'N'.
- $\color{#ff7b72}{\textsf{ALLOW OVERWRITE}}$ - If target APP_ID already exists in target APEX <b><u>Instance</u></b> it will overwrite it. <u>Make sure this is correct!</u>
- $\color{#ff7b72}{\textsf{AUTH SCHEME NAME}}$ - target APP's authentication scheme. APEX's default is $\color{#f7ee78}{\textsf{'Oracle APEX Accounts'}}$ . A 2nd option is $\color{#a5d6ff}{\textsf{'OCI SSO'}}$ but that requires configuring the workspace/app with OAuth for external authentication. Check below for an example of integrating Oracle APEX with OCI IAM domains:
https://docs.oracle.com/en/learn/apex-identitydomains-sso/index.html


===sql
DECLARE
  l_run_id  NUMBER;
  l_job     VARCHAR2(128);
BEGIN
  deploy_mgr_pkg.enqueue_deploy(
    p_bundle_id        => :bundle_id,
    p_install_mode     => 'INITIAL',
    p_workspace_name   => 'YOUR_WORKSPACE',
    p_app_id           => APP_ID,
    p_use_id_offset    => 'N',
    p_auth_scheme_name => 'Oracle APEX Accounts', -- or "OCI SSO" for federated login
    p_allow_overwrite  => 'N',
    o_run_id           => l_run_id,
    o_job_name         => l_job
  );
END;
/
===

---

### Step 4 — Monitor
Connect as the application schema owner and run the below queries:
- ===SELECT * FROM deploy_runs ORDER BY run_id DESC;===
- ===SELECT * FROM deploy_applied ORDER BY applied_at DESC;===

---

### Step 5 — Validate
After the job completes:
- open the APEX application
- verify core pages load
- verify chatbot initializes
- verify required `APP_CONFIG` entries exist

---

### Step 6 — Enable jobs
Jobs defined in `db/ddl/90_jobs.sql` are created **disabled**.

Enable them only after:
- configuration is verified
- initial validation is complete

---

## Logging & Observability

Each deployment is tracked via deployment run tables:
- run id
- status
- timestamps
- detailed execution log (CLOB)

Deployment issues should always be investigated via these logs.

---

## What this document does NOT cover

- Application updates
- In-app self-update flows
- Dry-run update behavior
- Bundle export

These topics are documented separately in [Update Guide](update.md).

**See also**
- [Update Guide](update.md)
- [Admin Guide](admin-guide.md)
- [Deployment Manager API](deploy-manager-api.md)
- [Troubleshooting](troubleshooting.md)
