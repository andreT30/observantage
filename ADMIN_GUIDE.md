# Administration & Operations Guide

Home: [README](../README.md) · **Docs** · **Administration & Operations**

This guide is for **application administrators and operators**.
It covers configuration, monitoring, and operational control **after initial deployment**.

> Initial installation is documented in `docs/deployment.md`.  
> Application updates are handled **inside the application UI** and documented in `docs/update.md`.

---

## Administrator Responsibilities

Administrators are responsible for:

- validating post-deployment configuration
- managing workloads and subscriptions
- controlling scheduler jobs
- maintaining chatbot metadata
- monitoring runs, logs, and data freshness

Administrators **do not** deploy or update the application manually via SQL.

---

## Access Control

Administrative capabilities are protected using:

- Oracle APEX authorization schemes
- application-level roles

Only authorized users can access:
- configuration pages
- workload and subscription management
- job control
- chatbot metadata editors
- update workflows

---

## Configuration Management

### Central Configuration Table
- `APP_CONFIG`

All runtime behavior is driven from configuration values:
- OCI scope (compartments, regions)
- tagging conventions
- feature toggles
- chatbot behavior

Best practices:
- treat configuration as code
- document every change
- validate after each change

---

## Workload Administration

### Purpose
Workloads define logical groupings used by:
- dashboards
- reports
- cost attribution
- chatbot queries

### Admin Pages
- **Create Workload**
- **My Reports & Workloads**

### Best Practices
- avoid overlapping workload definitions
- keep naming consistent
- validate workloads against real resources

Incorrect workload definitions lead to misleading analytics.

---

## Subscription Management

### Purpose
Manage OCI subscription metadata used by:
- credit tracking
- projections
- executive dashboards

### Admin Pages
- **Create Subscription Detail**

Changes here directly affect financial reporting.

---

## Scheduler Jobs

### Source of Jobs
All application scheduler jobs are defined in the bundle under:

===
db/ddl/90_jobs.sql
===

During initial deployment:
- jobs are created **disabled**
- no jobs are enabled automatically

---

### Job Lifecycle

Administrators control job lifecycle entirely from the application:

1. Review job definitions
2. Enable jobs after configuration validation
3. Monitor execution and duration
4. Disable jobs during maintenance or incidents

Jobs should **never** be enabled before configuration is complete.

---

### Job Monitoring
Use admin pages and logs to:
- verify successful execution
- identify failures
- observe execution time trends

---

## Data Load & Initialization

### Initial Load
Performed once after deployment to bootstrap data.

### Data Load
Used for controlled reprocessing or corrections.

These operations are exposed via the admin UI and logged.

---

## Chatbot Administration

### What Admins Control
- business glossary rules
- keywords and synonyms
- dataset metadata
- summaries used in responses

### Admin Pages
- Business Glossary editors
- Chatbot parameter pages
- Debug and inspection pages

---

### Extending Chatbot Coverage
To support new business language:
1. Add or update glossary rules
2. Add keywords/synonyms
3. Test via chatbot UI
4. Monitor logs

No deployment or update is required.

---

## Application Updates

Application updates are **not performed via SQL scripts**.

Updates are:
- triggered from the application UI
- executed using the internal Deployment Manager
- logged and auditable

See: `docs/update.md`

---

## Monitoring & Logging

### What Is Logged
- job executions
- data load runs
- chatbot requests
- deployment/update runs
- errors and warnings

### Operator Guidance
- always capture run IDs
- use logs as the primary troubleshooting source
- never modify data directly to fix issues

---

## Operational Checklists

### After Initial Deployment
- [ ] configuration completed (`APP_CONFIG`)
- [ ] initial data load executed
- [ ] dashboards render correctly
- [ ] chatbot initializes
- [ ] jobs reviewed but still disabled

---

### Before Enabling Jobs
- [ ] configuration validated
- [ ] data load successful
- [ ] admin access verified
- [ ] monitoring in place

---

## Common Admin Mistakes

- enabling jobs too early
- modifying data outside the app
- overlapping workload definitions
- changing tag keys without validation
- attempting updates via SQL

---

## Governance & Audit

- all operations are logged
- chatbot SQL is traceable
- configuration changes are auditable
- deployment and updates are separated

This separation is intentional and reduces operational risk.

**See also**
- [Deployment Guide](deployment.md)
- [Update Guide](update.md)
- [Configuration Reference](configuration.md)
- [Troubleshooting](troubleshooting.md)
- [Security Model](security.md)


# Configuration

Home: [README](../README.md) · **Docs** · **Configuration**

## Overview
The application is configured entirely through **database tables**, primarily `APP_CONFIG`.

There are:
- no hardcoded OCIDs
- no hardcoded tag names
- no environment-specific values in code

This makes the system portable across:
- DEV / TEST / PROD
- different OCI tenancies
- different tagging conventions

---

## Configuration Principles

- Configuration is **read at runtime**
- Keys are stable; values vary by environment
- Missing or incorrect config causes visible, logged failures
- Secrets must be injected externally (never committed)

---

## APP_CONFIG Table

### Purpose
Centralized key-value configuration store.

### Typical Columns
- `CONFIG_KEY`
- `CONFIG_VALUE`
- optional description / category

---

## Configuration Categories

### 1. OCI Environment

Controls how the application connects to OCI data.

**Examples**
- root compartment identifiers
- region lists
- tenancy scope

These define **what data is visible** to the app.

---

### 2. Tag Mapping & Attribution

Defines how OCI tags are interpreted.

**Examples**
- cost center tag key
- environment tag key
- implementor / owner tag key

These values are used dynamically to extract tag data from JSON.

---

### 3. Analytics Behavior

Controls how analytics behave.

**Examples**
- default date ranges
- normalization rules
- feature toggles for dashboards

---

### 4. Chatbot Configuration

Controls NL2SQL behavior.

**Examples**
- model identifiers
- routing behavior
- execution limits
- verbosity of summaries

This allows chatbot tuning **without code changes**.

---

### 5. Application Behavior

General application settings.

**Examples**
- UI defaults
- feature flags
- environment labels

---

## Environment-Specific Values

Typical environment-specific values:
- OCI compartment OCIDs
- region lists
- tagging conventions
- enabled jobs

These should be:
- set post-deployment
- excluded or anonymized in GitHub

---

## Configuration Deployment Strategy

Recommended:
1. Deploy schema objects
2. Insert baseline config keys (no secrets)
3. Override values per environment
4. Validate via health checks

---

## Validation & Troubleshooting

Misconfiguration usually manifests as:
- empty dashboards
- chatbot returning no results
- job failures

Check:
- APP_CONFIG values
- logging tables
- job run logs

Details: [troubleshooting.md](troubleshooting.md)

**See also**
- [Admin Guide](admin-guide.md)
- [Deployment Guide](deployment.md)


# Application Configuration Reference (APP_CONFIG)

Home: README · Docs · Application Configuration

---

## Purpose

This document describes all application configuration parameters stored in the APP_CONFIG table.

APP_CONFIG is the single source of truth for runtime behavior. No configuration values are hardcoded in application logic.

---

## APP_CONFIG Table Model

- CONFIG_KEY: Unique identifier for the configuration parameter.
- CONFIG_VALUE: String-based value interpreted by usage context.

---

## Non-Modifiable Configuration Keys

Structural and system-owned keys. These must not be modified during normal operation.

| CONFIG_KEY | Default Value | Purpose | Usage | Notes |
| --- | --- | --- | --- | --- |
| AVAILABILITY_DAYS_BACK | 7 | Lookback window for availability metrics. | Constrains availability ingestion and reporting time range. | Affects data volume and processing time. |
| AVAILABILITY_METRICS_TABLE | OCI_AVAILABILITY_METRICS_PY | Target table for persisted availability metrics. | Written by availability jobs; read by dashboards and reports. | Table must exist and match expected schema. |
| AVAILABILITY_METRIC_GROUPS_JSON | [   {     "namespace":"oci_computeagent",     "resource_group":null,     "metrics":["CpuUtilization","MemoryUtilization"],     "resource_display_keys":["resourceDisplayName","instanceId"]   } ] | JSON definition of availability metric groupings. | Drives metric grouping/aggregation in dashboards and queries. | Must be valid JSON. |
| AVAILABILITY_QUERY_SUFFIX | .mean() | Optional SQL suffix applied to availability queries. | Appended to generated availability SQL for advanced filtering/extensions. | -- |
| AVAILABILITY_RESOLUTION | 1h | Aggregation granularity for availability metrics. | Controls hourly vs daily rollups in availability processing. | Higher resolution increases data volume. |
| COMPARTMENTS_TABLE | OCI_COMPARTMENTS_PY | Database table storing discovered OCI compartments. | Populated by discovery jobs; referenced by dashboards, filters, and chatbot context. | Do not modify manually. |
| CREDENTIAL_NAME | OCI$RESOURCE_PRINCIPAL | Legacy DBMS_CLOUD credential name reference. | Used only in credential-based deployments; ignored with Resource Principal. | Must not contain secrets. |
| CSV_FIELD_LIST | AVAILABILITYZONE CHAR(4000), BILLEDCOST DECIMAL(38,12), BILLINGACCOUNTID INTEGER, BILLINGACCOUNTNAME VARCHAR(32767), BILLINGCURRENCY CHAR(4000), BILLINGPERIODEND VARCHAR(64), BILLINGPERIODSTART VARCHAR(64), CHARGECATEGORY CHAR(4000), CHARGEDESCRIPTION CHAR(4000), CHARGEFREQUENCY VARCHAR(32767), CHARGEPERIODEND VARCHAR(64), CHARGEPERIODSTART VARCHAR(64), CHARGESUBCATEGORY VARCHAR(32767), COMMITMENTDISCOUNTCATEGORY VARCHAR(32767), COMMITMENTDISCOUNTID VARCHAR(32767), COMMITMENTDISCOUNTNAME VARCHAR(32767), COMMITMENTDISCOUNTTYPE VARCHAR(32767), EFFECTIVECOST DECIMAL(38,12), INVOICEISSUER CHAR(4000), LISTCOST DECIMAL(38,12), LISTUNITPRICE DECIMAL(38,12), PRICINGCATEGORY VARCHAR(32767), PRICINGQUANTITY DECIMAL(38,12), PRICINGUNIT CHAR(4000), PROVIDER CHAR(4000), PUBLISHER CHAR(4000), REGION CHAR(4000), RESOURCEID CHAR(4000), RESOURCENAME VARCHAR(32767), RESOURCETYPE CHAR(4000), SERVICECATEGORY CHAR(4000), SERVICENAME CHAR(4000), SKUID CHAR(4000), SKUPRICEID VARCHAR(32767), SUBACCOUNTID CHAR(4000), SUBACCOUNTNAME CHAR(4000), TAGS VARCHAR(32767), USAGEQUANTITY DECIMAL(38,12), USAGEUNIT CHAR(4000), OCI_REFERENCENUMBER CHAR(4000), OCI_COMPARTMENTID CHAR(4000), OCI_COMPARTMENTNAME CHAR(4000), OCI_OVERAGEFLAG CHAR(4000), OCI_UNITPRICEOVERAGE VARCHAR(32767), OCI_BILLEDQUANTITYOVERAGE VARCHAR(32767), OCI_COSTOVERAGE VARCHAR(32767), OCI_ATTRIBUTEDUSAGE DECIMAL(38,12), OCI_ATTRIBUTEDCOST DECIMAL(38,12), OCI_BACKREFERENCENUMBER CHAR(4000) | Ordered list of fields used for CSV generation. | Controls column order during CSV export/import pipelines. | Must align with stage/target table structures. |
| DAYS_BACK | 1 | Default historical window for cost and usage processing. | Constrains cost ingestion time range and processing volume. | Larger values increase runtime and data volume. |
| EXA_MAINTENANCE_METRICS_TABLE | OCI_EXA_MAINTENANCE_PY | Target table for Exadata maintenance metrics. | Used by maintenance/availability monitoring logic. | Table must exist if feature is enabled. |
| FILE_SUFFIX | .csv.gz | File extension used for generated files. | Applied when creating export or staging files. | -- |
| FILTER_BY_CREATED_SINCE | Y | Toggle to filter resources by creation date. | Applied during resource selection to limit analysis scope. | Values: true/false. |
| KEY_COLUMN | OCI_REFERENCENUMBER | Primary identifier column used during merges/joins. | Used as join key between stage/target/resources/relationships as applicable. | Must exist in the referenced tables. |
| POST_PROCS | [   "PAGE1_CONS_WRKLD_MONTH_CHART_DATA_PROC",   "PAGE1_CONS_WRKLD_WEEK_CHART_DATA_PROC",   "COST_USAGE_TS_PKG.REFRESH_COST_USAGE_TS",   "REFRESH_CREDIT_USAGE_AGG_PROC",   "REFRESH_CREDIT_CONSUMPTION_STATE_PROC",   "DBMS_MVIEW.REFRESH('FILTER_VALUES_MV', METHOD => 'C')"] | List of post-processing procdures/functions/mviews after cost data load | Executed after successful cost data load | Must be valid JSON. Objects must exist |
| POST_SUBS_PROC_1 | UPDATE_OCI_SUBSCRIPTION_DETAILS | First post-processing procedure after subscription ingestion. | Executed after subscription ingestion to normalize/enrich data. | Procedure must exist and be executable. |
| POST_SUBS_PROC_2 | REFRESH_CREDIT_CONSUMPTION_STATE_PROC | Second post-processing procedure after subscription ingestion. | Executed after POST_SUBS_PROC_1 if configured. | Procedure must exist and be executable. |
| PREFIX_BASE | FOCUS Reports | Base prefix used for object naming in Object Storage. | Default as per Oracle documentation: *https://docs.oracle.com/en-us/iaas/Content/Billing/Concepts/costusagereportsoverview.htm#costreports* | -- |
| PREFIX_FILE | -- | Additional per-file prefix component. | Used to distinguish file types or processing stages. | Combined with PREFIX_BASE and FILE_SUFFIX. |
| RESOURCES_TABLE | OCI_RESOURCES_PY | Target table for discovered OCI resources. | Written by discovery jobs; read by dashboards/reports/chatbot. | Schema must align with discovery procedures. |
| RESOURCE_OKE_RELATIONSHIPS_PROC | POPULATE_OKE_RELATIONSHIPS_PROC | Procedure building OKE-specific resource relationships. | Links OKE clusters, node pools, nodes, and related resources. | Optional if OKE is not used. |
| RESOURCE_RELATIONSHIPS_PROC | POPULATE_RESOURCE_RELATIONSHIPS_PROC | Procedure building generic resource-to-resource relationships. | Builds parent-child and dependency relationships. | Should be idempotent for safe re-execution. |
| STAGE_TABLE | FOCUS_REPORTS_STAGE | Staging table used during file-based processing. | Receives transient intermediate data prior to final load. | Data is transient by design. |
| SUBSCRIPTIONS_TARGET_TABLE | OCI_SUBSCRIPTIONS_PY | Target table for processed subscription data. | Written by subscription ingestion jobs; used for reporting. | Schema must match ingestion logic. |
| SUBSCRIPTION_COMMIT_TARGET_TABLE | OCI_SUBSCRIPTION_COMMITMENTS | Target table for subscription commitment data. | Stores committed amounts/usage for commitment analytics. | Schema must match ingestion logic. |
| TARGET_TABLE | FOCUS_REPORTS_PY | Final target table for processed data. | Receives validated and transformed data from pipelines. | Schema must match load/merge logic. |
| USE_DYNAMIC_PREFIX | Y | Toggle for dynamic prefix generation. | Enables/disables context-based (e.g., time-based) naming to avoid collisions. | Values: true/false. |

---

## Modifiable Configuration Keys

Operational keys that administrators can modify through the application UI.

| CONFIG_KEY | Purpose | Usage | Notes (examples) |
| --- | --- | --- | --- |
| ADB_COMPUTE_COUNT_DOWN | Minimum compute value for Autonomous Database scaling. | Used by scaling logic to reduce compute allocation. | Integer value (2). |
| ADB_COMPUTE_COUNT_UP | Maximum compute value for Autonomous Database scaling. | Used by scaling logic to increase compute allocation. | Integer value (8). |
| ADB_OCID | OCID of the Autonomous Database. | Used for scheduled OCPU scaling. | OCID format (ocid1.autonomousdatabase.oc1.eu-frankfurt-1.....). |
| AVAILABILITY_REGIONS | Subscribed regions in tenancy. | Used to discover cost/resources across tenancy's subscribed regions. |  Comma-separated list (eu-frankfurt-1,eu-amsterdam-1). |
| COMPARTMENTS_PATH_PREFIX_CHILD1 | Logical path prefix for compartment hierarchy of child tenancies in a multi-org OCI tenancy. | Used to distinguish tenancy's root and other compartments from children tenancies. | Organization-specific convention (myothertenancy). |
| COMPARTMENT_ID | Application's compartment OCID. | Typically this is ADB's compartment OCID. | Must be an OCID (ocid1.compartment.oc1....). |
| COST_TAG_COSTCENTER_KEY | Tag key used to extract cost center value. | Used when parsing OCI tags JSON to derive cost center. Usually a tag that would define a workload/application | Must of format: tag-namespace.tag-key (Oracle-Standard.CostCenter). |
| COST_TAG_CREATED_BY_KEY | Tag key used to extract created-by value. | Used when parsing OCI tags JSON to derive creator. | Must of format: tag-namespace.tag-key (Oracle-Tags.CreatedBy). |
| COST_TAG_ENVIRONMENT_KEY | Tag key used to extract environment label. | Used when parsing OCI tags JSON to derive environment label. | Must of format: tag-namespace.tag-key (Oracle-Standard.Environment). |
| COST_TAG_IMPLEMENTOR_KEY | Tag key used to extract implementor value. | Used when parsing OCI tags JSON to derive implementor. | Must of format: tag-namespace.tag-key (Oracle-Standard.Implementor). |
| FC_ADB_REGION | OCI region of the application's ADB placement. | Used for region-aware OCI endpoint selection. | Example: (eu-frankfurt-1). |
| OBJECT_BASE_URI | Base URI for OCI Object Storage FOCUS REPORTS access. | Used by DBMS_CLOUD when retrieving FOCUS reports objects of the tenancy. The URL is constructed in two parts -> Fixed part: https://objectstorage.eu-frankfurt-1.oraclecloud.com/n/bling/b/ Tenant specific part: ocid1.tenancy.oc1..../o/ .  | Tenant specific (https://objectstorage.eu-frankfurt-1.oraclecloud.com/n/bling/b/ocid1.tenancy.oc1...../o/). |
| OCI_HOME_REGION | Tenancy home region. | Used for tenancy-level and global OCI interactions. | May differ from FC_ADB_REGION (eu-frankfurt-1). |
| OCI_ROOT_CHILD1_COMPARTMENT_OCID | Primary child tenancy OCID. | In a multi-org tenancy, this would be the 1st child. If more need to be added, manual APP_CONFIG addtions and Scheduled jobs have to be added | OCID format (ocid1.tenancy.oc1..). |
| OCI_ROOT_COMPARTMENT_OCID | Tenancy root compartment OCID. | Used for compartment discovery and tenancy-wide scope operations. | OCID format (ocid1.tenancy.oc1..). |
| OIDC_DISCOVERY_URL | OIDC discovery endpoint used for authentication configuration. | Consumed by authentication flow to resolve OIDC metadata. | Only used when APEX is configured for External Authentication (OCI IAM Domains example: https://idcs-....identity.oraclecloud.com:443/.well-known/openid-configuration). |
| P2_COMPARTMENT_ID | Same as COMPARTMENT_ID. | Same as COMPARTMENT_ID. | Must be an OCID (ocid1.compartment.oc1....). |
| l_auth_scheme_name | Name of the Oracle APEX authentication scheme. | Used to select the authentication mechanism used by the application. | Must match an existing APEX authentication scheme name. Usually "Oracle APEX Accounts" or Application's default "OCI SSO" (OCI SSO)|
| model_id | Generative AI model identifier used by the chatbot. | Passed to the AI execution layer for interpretation and summarization. | Must be an OCID of the OCI's GenAI model chosen and must be available/enabled for the tenancy/region (ocid1.generativeaimodel.oc1.eu-frankfurt-1.amaaaaaask7dceyaaypm2hg4db3evqkmjfdli5mggcxrhp2i4qmhvggyb4ja). |


# NL2SQL Chatbot

Home: [README](../README.md) · **Docs** · **NL2SQL Chatbot**

## Purpose
The NL2SQL chatbot allows users to query OCI cost and resource data using **natural language**, without writing SQL.

Example questions:
- “Show total cost last month by service”
- “Which workloads increased cost this quarter?”
- “Cost per cluster including child resources”
- “Show monthly cost trend with MoM percentage change”

The chatbot is **metadata-driven**, explainable, and fully logged.

---

## Design Principles

- Deterministic SQL generation (not free-form LLM guessing)
- Business language mapped explicitly to schema
- No hardcoded table or column names
- Fully auditable execution
- Safe execution boundaries
- Declarative rule-based behavior via glossary metadata

---

## High-Level Flow

1. User submits a question via APEX  
2. Input is normalized and tokenized  
3. Intent and entities are inferred  
4. Glossary rules determine:
   - metrics
   - dimensions
   - filters
   - time ranges
   - comparison logic (MoM, WoW, trends)
5. SQL is generated  
6. SQL is validated and executed  
7. Results are summarized  
8. Full trace is logged  

---

## Chatbot Processing Pipeline

===mermaid
flowchart TD
  Q[User Question] --> NORM[Normalize & Tokenize]

  NORM --> ROUTER[Intent Router]

  ROUTER --> GLOSS[Glossary Rules]
  ROUTER --> META[Dataset Metadata]

  GLOSS --> SQLGEN[SQL Generator/Reasoner]
  META --> SQLGEN

  SQLGEN --> VALID[Validate & Guardrails]
  VALID --> EXEC[Execute SQL]

  EXEC --> RES[Result Set]
  RES --> TOK[Tokenize PII]
  TOK --> SUM[Summarization]

  SUM --> DETOK[Detokenize PII]
  DETOK --> UI[Chatbot Response]

  SQLGEN --> LOG[Execution Log]
  EXEC --> LOG
===

---

## Core Components

### 1. Glossary Rules (Primary Control Layer)

Glossary rules define how **business language maps to SQL semantics**.

They control:
- metrics
- filter dimensions
- grouping dimensions
- time logic
- period comparison semantics

Rules are stored in database tables and managed via APEX UI.

No application redeployment is required to extend vocabulary or behavior.

---

## Business Glossary Rules — Operator Guide


# Business Glossary Rules — Operator Guide (APEX UI Aligned)

This guide documents the **Business Glossary Rules module** exactly as exposed in the APEX UI.

Use this document to safely create, modify, and validate glossary rules that control NL2SQL behavior.

---

## Main UI Actions

### Refresh Rules
Reloads all glossary rules from the database.

---

### Create New Rule
Creates a new generic glossary rule (metrics, filters, time rules).

---

### Create Workload Rule
Creates workload-bound rules using existing WORKLOAD_NAME values.

Automatically generates keyword mappings.

---

## Business Glossary Grid Columns

These columns appear in the **Business Glossary Rules** report.

---

### Rule ID → System Identifier
Unique identifier of the rule group.

Example → 22

---

### Active → Enable / Disable Rule

Controls whether the rule is evaluated.

Values:
TRUE  
FALSE  

---

### Priority → Execution Order

Higher values execute first.

Recommended ranges:

Time rules → 150–300  
Workloads → 20–50  
Metrics → 1–10  

Example → 200

---

### Keyword → Trigger Token

Text matched from user input.

Example → cost  
Example → storage  
Example → MoM  

---

### Ord → Keyword Order

Order of evaluation inside the same Rule ID.

Use incremental values:

1, 2, 3, ...

---

### Match Mode → Matching Strategy

Controls keyword detection.

Values:

any → substring match  
exact → full token match  

Example → any

---

### Role → Rule Behavior Type

Defines SQL semantic behavior.

Supported values:

metric  
filter_dimension  
group_dimension  
generic_time_filter  

---

### Description → Operator Documentation

Human readable explanation of rule purpose.

Example → Numeric cost metric for OCI spend.

---

## Target Mapping Fields

Used for SQL binding.

---

### Target Table → SQL Source View

View used by the rule.

Examples:

COST_USAGE_MONTHLY_WKLD_V  
COST_USAGE_DAILY_WKLD_V  

---

### Target Column → SQL Column

Column affected by the rule.

Examples:

COST  
WORKLOAD_NAME  
SERVICECATEGORY  

---

### Target Role → SQL Semantic Role

Controls how SQL builder treats column.

Values:

metric  
filter_dimension  
group_dimension  

---

## Target Filter (JSON Editor)

Defines exact SQL predicate behavior.

Click **Format JSON** to validate syntax.

---

## Common Filter Templates

### Text Matching Filter

Field → Example

JSON Example:

===json
{
  "operator": "like",
  "value": "%{matched_keyword}%",
  "case": "upper"
}
===

Resulting SQL:

UPPER(column) LIKE '%STORAGE%'

---

### Exact Value Filter

Field → Example

JSON Example:

===json
{
  "operator": "=",
  "value": "123",
  "case": "none"
}
===

---

### Default Daily Snapshot Filter

Used on COST_USAGE_DAILY_WKLD_V

Field → Example

operator → =  
value → TRUNC(SYSDATE-3,'DD')  
case → none  

---

## Generic Filter Rule (Top JSON Section)

Used for time logic and global filters.

---

### Month Range Example (Summer)

Field → Example

JSON example:

===json
{
  "applies_to": "MONTH",
  "operator": "IN",
  "value": [
    "06",
    "07",
    "08",
    "6",
    "7",
    "8",
    "JUN",
    "JUL",
    "AUG",
    "JUNE",
    "JULY",
    "AUGUST"
  ],
  "case": "upper"
}
===

---

### Period Comparison Example (MoM)

Field → Example

JSON example: 

===json
{
  "applies_to": "PERIOD_COMPARISON",
  "comparison": {
    "type": "MOM",
    "basis": "PCT_CHANGE"
  },
  "time_grain": "MONTH",
  "requires_time_series": true,
  "include_outputs": [
    "CURRENT",
    "PREVIOUS",
    "DELTA",
    "PCT_CHANGE"
  ],
  "notes": "Compute base_value by month bucket then LAG(base_value) over bucket; pct_change=(base-prev)/NULLIF(prev,0)*100."
}
===

---

## Create Keyword Action

Available inside Update Rule dialog.

Allows adding multiple keywords to same Rule ID.

---

### Example

Rule ID → 41  
Keyword → spend  
Keyword → cost  

Both map to same metric behavior.

---

## Delete Keyword Button

Removes selected keyword mapping only.

Does NOT delete entire rule group.

---

## Delete Rule Button

Deletes full rule group including all keywords.

---

## Create Workload Rule Workflow

1. Click **Create Workload Rule**
2. Press **Create Key from Workloads**
3. Select workload values
4. Save

System auto-generates:

Keyword → workload name  
Target Column → WORKLOAD_NAME  
Filter Template → LIKE pattern  

---

## BusinessGlossaryTester

Use for validation.

Test:

Single keyword  
Combined phrases  
Time + metric + workload  

Verify:

Detected keywords  
Applied filters  
Generated SQL  

---

## Save Button Behavior

Validates:

JSON syntax  
Required fields  
Target bindings  

Only valid rules are stored.

---

## Cancel Button

Closes editor without saving.

---

## Best Practices

Always:

Use UPPER matching  
Prefer aggregated views  
Keep priorities separated  
Avoid overlapping keywords  
Test every rule  

---

## Production Safety Rules

Never:

Modify PROD without testing  
Use base tables  
Hardcode schema names  
Mix metric + filter roles  
Use ambiguous keywords  

---

## Recommended Rule Patterns

Metric rules → COST, USAGE  
Workload rules → WORKLOAD_NAME  
Service filters → SERVICECATEGORY  
Time rules → generic_time_filter  

---



---

## Rule Decision Matrix (What To Create)

Use this table to decide **which rule type to create** based on user intent.

| User Wants To Ask | Create Rule Role | Target Table | Target Column | JSON Needed |
|-------------------|----------------|-------------|---------------|-------------|
| "total cost", "spend" | metric | COST_USAGE_MONTHLY_WKLD_V | COST | No |
| "usage hours", "GB used" | metric | COST_USAGE_MONTHLY_WKLD_V | USAGE | No |
| "storage cost", "network cost" | filter_dimension | COST_USAGE_MONTHLY_WKLD_V | SERVICECATEGORY | LIKE filter |
| "by service", "per region" | group_dimension | COST_USAGE_MONTHLY_WKLD_V | SERVICECATEGORY / REGION | No |
| "Workload1 cost" | filter_dimension (Workload Rule) | COST_USAGE_DAILY_WKLD_V | WORKLOAD_NAME | Auto-generated |
| "today usage" | filter_dimension | COST_USAGE_DAILY_WKLD_V | DATE_BUCKET | Equals filter |
| "summer cost" | generic_time_filter | — | — | Month IN filter |
| "last 7 days" | generic_time_filter | — | — | Rolling window |
| "MoM trend" | generic_time_filter | — | — | PERIOD_COMPARISON |
| "monthly trend" | group_dimension | COST_USAGE_MONTHLY_WKLD_V | DATE_BUCKET | No |

---

## Quick Creation Recipes

### Recipe — Add New Cost Metric

Use when user asks about a numeric value.

Steps:

Create New Rule  
Role → metric  
Target Table → COST_USAGE_MONTHLY_WKLD_V  
Target Column → COST  
Priority → 5  
Keyword → spend  

---

### Recipe — Add Service Category Filter

Use when user mentions OCI services.

Steps:

Create New Rule  
Role → filter_dimension  
Target Column → SERVICECATEGORY  
Priority → 20  

Target Filter JSON:

{
  "operator": "like",
  "value": "%{matched_keyword}%",
  "case": "upper"
}

Keyword Examples:

storage  
network  
compute  

---

### Recipe — Add New Workload

Use when onboarding a new workload name.

Steps:

Click Create Workload Rule  
Press Create Key from Workloads  
Select workload  
Save  

System auto-generates filter rules.

---

### Recipe — Add Seasonal Time Logic

Use for business calendar concepts.

Steps:

Create New Rule  
Role → generic_time_filter  

Top JSON:

{
  "applies_to": "MONTH",
  "operator": "IN",
  "value": ["12","DEC","DECEMBER"],
  "case": "upper"
}

Keyword:

winter  
december  

---

### Recipe — Enable Period Comparison

Use for trends and deltas.

Steps:

Create New Rule  
Role → generic_time_filter  

Top JSON:

{
  "applies_to": "PERIOD_COMPARISON",
  "comparison": { "type": "MOM", "basis": "PCT_CHANGE" },
  "time_grain": "MONTH",
  "requires_time_series": true
}

Keyword:

MoM  
month over month  

---

## Admin Validation Checklist

Before deploying to PROD:

- Rule tested in BusinessGlossaryTester  
- SQL preview reviewed  
- Priority conflicts avoided  
- No duplicate keywords  
- Time logic validated  
- Dataset = LATEST  
- Environment = PROD  

---



---

### Rule Types (by ROLE)

#### metric
Defines measurable values.

Examples:
- cost → COST
- usage → USAGE

---

#### filter_dimension
Defines WHERE clause behavior.

Examples:
- “storage” → SERVICECATEGORY LIKE '%STORAGE%'
- Workload1 → WORKLOAD_NAME LIKE '%WORKLOAD1%'

---

#### group_dimension
Defines GROUP BY behavior.

Examples:
- “by service”
- “per workload”
- “per region”

---

#### generic_time_filter

Used for:

- Fixed ranges  
  - “summer”
  - “December”

- Rolling windows  
  - “last 7 days”
  - “past 3 months”

- Period comparisons  
  - MoM
  - WoW
  - DoD
  - QoQ

These rules may:

- inject WHERE clauses
- define time bucketing
- enforce time-series output
- trigger window functions

---

## Time Logic Examples

### Fixed period mapping (Summer)

===json
{
  "applies_to": "EXPR_FILTER",
  "expr": "EXTRACT(MONTH FROM DATE_BUCKET)",
  "operator": "IN",
  "value": [6,7,8]
}
===

---

### Default daily snapshot behavior

If user omits a date:

===sql
DATE_BUCKET = TRUNC(SYSDATE-3,'DD')
===

Ensures consistent reporting snapshots.

---

## Period Comparison Logic

Example glossary rule:

===json
{
  "applies_to": "PERIOD_COMPARISON",
  "comparison": { "type": "MOM", "basis": "PCT_CHANGE" },
  "time_grain": "MONTH",
  "requires_time_series": true
}
===

This automatically triggers:

- compare intent
- monthly bucketing
- LAG window functions
- percentage change calculation

---

## SQL Generation

SQL is generated from the reasoning plan.

Characteristics:

- No dynamic guessing
- No hidden joins
- No silent filters
- No unsafe clauses

---

## APEX Rule Management UI

Rules are maintained using APEX admin pages.

Screenshot placeholders:

Glossary Rule Editor  
![Glossary Rule Editor](/screenshots/chatbot_params1.png)

Create Glossary Rule  
![Create Glossary Rule](/screenshots/chatbot_params2.png)

Update Glossary Rule  
![Update Glossary Rule](/screenshots/chatbot_params4.png)

Create Glossary Keyword   
![Create Glossary Keyword](/screenshots/chatbot_params3.png)

Create Workload Rule  
![Create Workload Rule](/screenshots/chatbot_params5.png)

Keyword Tester
![Keyword Tester](/screenshots/chatbot_params6.png)

Examples
![Example1](/screenshots/chatbot_params7.png)
![Example2](/screenshots/chatbot_params8.png)

---

## Guardrails

===mermaid
flowchart LR
  USER[Authenticated User] --> APEX[APEX App]

  APEX -->|Authorized| VIEW[Views]
  APEX -->|Controlled| PKG[PL/SQL APIs]

  PKG --> DATA[Cost & Resource Data]

  subgraph Controls
    AUTH[Auth Schemes]
    CFG[APP_CONFIG]
    LOG[Audit Logs]
  end

  AUTH --> APEX
  CFG --> PKG
  PKG --> LOG
===

---

## Summarization

Results are rendered as:

- tables
- charts
- natural language explanations

Summaries describe:

- filters
- time window
- applied comparison logic
- metric interpretation

---

## Logging & Traceability

Every request logs:

- user input
- glossary hits
- reasoning JSON
- generated SQL
- execution statistics

---

## Extending the Chatbot

To add new logic:

1. Add glossary rule
2. Add keywords
3. Adjust priority
4. Test in UI

No PL/SQL changes required.

---

## Failure Modes

Typical causes:

- missing glossary coverage
- overlapping rules
- ambiguous time logic
- insufficient time range

All failures are logged.

---

See also:
- Usage Guide
- Admin Guide
- Security Model


---

## Recommended Authoring Guidelines

When creating glossary rules:

### Metrics
- Always map metrics to aggregated fact views
- Avoid raw base tables

### Filters
- Use `LIKE` with `UPPER()` for robustness
- Prefer semantic business terms over column names

### Time Rules
- Use `generic_time_filter` only for reusable patterns
- Use `PERIOD_COMPARISON` for trends and deltas

### Workloads
- Always use workload abstraction instead of hardcoded compartments

---

---
## Configuration Parameters (CHATBOT_PARAMETERS)

This section documents the parameters consumed by the NL2SQL engine (the attached PL/SQL packages) and the provided parameter export.

### Parameter reference

| Parameter | Type | Status | Used by (code) | What it does | One-sentence example |
|---|---|---|---|---|---|
| `CHART_SYSTEM` | CLOB prompt/component | Used | CHATBOT_CORE_CHART_PKG.txt (literal), CHATBOT_CORE_CHART_PKG.txt (load_component) | System prompt used by the chart suggestion phase to turn result rows into an ECharts-compatible chart specification JSON. | Update `CHART_SYSTEM` to adjust how the model behaves (e.g., enforce SQL-only output or change answer tone) without changing code. |
| `CONTEXT_TURNS` | Number | Used | CHATBOT_CORE_RUNNER_PKG.txt (literal) | Number of prior turns to include when building the conversation history block. | Set `CONTEXT_TURNS=4` to include the last 4 turns when constructing the relevant history context. |
| `DEBUG_FLAG` | Scalar (string/flag) | Used | CHATBOT_CORE_CTX_PARAMS_PKG.txt (literal) | Enables verbose logging (request/response payloads, intermediate CLOBs) when set to TRUE for the current dataset/environment. | Set `DEBUG_FLAG=TRUE` to enable the corresponding feature for a dataset/environment. |
| `FOLLOWUP_CONTEXT_TURNS` | Number | Referenced in code (missing from CSV) | CHATBOT_CORE_SQL_PKG.txt (get_num_param) | Number of previous turns included when building the follow-up decider history block. | Set `FOLLOWUP_CONTEXT_TURNS=4` to include the last 4 turns when constructing the relevant history context. |
| `FOLLOWUP_DECIDER_SYSTEM` | CLOB prompt/component | Used | CHATBOT_CORE_SQL_PKG.txt (literal), CHATBOT_CORE_SQL_PKG.txt (load_component), CHATBOT_FOLLOWUP_PKG.txt (literal), CHATBOT_FOLLOWUP_PKG.txt (load_component) | System prompt used to classify follow-up messages and decide whether to answer directly, ask a question, or rewrite SQL. | Update `FOLLOWUP_DECIDER_SYSTEM` to adjust how the model behaves (e.g., enforce SQL-only output or change answer tone) without changing code. |
| `FOLLOWUP_HISTORY_MAX_CHARS` | Number | Referenced in code (missing from CSV) | CHATBOT_CORE_SQL_PKG.txt (get_num_param) | Character budget for the follow-up decider history block. | Set `FOLLOWUP_HISTORY_MAX_CHARS=2000` to cap the associated prompt/input block at 2,000 characters. |
| `FOLLOWUP_SQL_SYSTEM` | CLOB prompt/component | Used | CHATBOT_CORE_RUNNER_PKG.txt (literal) | Prompt template used to generate a revised SQL statement from previous SQL plus a follow-up request. | Update `FOLLOWUP_SQL_SYSTEM` to adjust how the model behaves (e.g., enforce SQL-only output or change answer tone) without changing code. |
| `GENERAL_CHAT_SYSTEM` | CLOB prompt/component | Used | CHATBOT_CORE_RUNNER_PKG.txt (literal) | System prompt used when the resolved intent is GENERAL_CHAT (no SQL execution). | Update `GENERAL_CHAT_SYSTEM` to adjust how the model behaves (e.g., enforce SQL-only output or change answer tone) without changing code. |
| `GLOSSARY_EXTRACT_SYSTEM` | CLOB prompt/component | Used | CHATBOT_CORE_RUNNER_PKG.txt (literal) | System prompt used to extract the most relevant glossary rules/keywords for the current question. | Update `GLOSSARY_EXTRACT_SYSTEM` to adjust how the model behaves (e.g., enforce SQL-only output or change answer tone) without changing code. |
| `GLOSSARY_TEXT` | CLOB prompt/component | Used | CHATBOT_CORE_SQL_PKG.txt (literal) | Export of glossary rules/keywords in compact text form used to ground the NL2SQL generation. | Update `GLOSSARY_TEXT` to add new business terms (e.g., map “credit burn” to the CREDIT_CONSUMPTION_STATE view) and the bot will pick them up. |
| `GLOSSARY_THRESHOLD_SCORE10` | Number | Used | CHATBOT_CORE_RUNNER_PKG.txt (literal) | Numeric threshold controlling when the 'glossary quick match' gate is considered confident enough to reuse/return its excerpt. | Set `GLOSSARY_THRESHOLD_SCORE10=7.0` so only intent scores ≥7/10 are treated as NL2SQL. |
| `HISTORY_MAX_CHARS` | Number | Used | CHATBOT_CORE_RUNNER_PKG.txt (literal) | Character budget for the history block included in prompts. | Set `HISTORY_MAX_CHARS=2000` to cap the associated prompt/input block at 2,000 characters. |
| `INTENT_NL2SQL_THRESHOLD_SCORE10` | Number | Used | CHATBOT_CORE_SQL_PKG.txt (literal) | Threshold (0–10 scale) above which the ensemble intent resolver treats a message as NL2SQL (otherwise GENERAL_CHAT / CLARIFY). | Set `INTENT_NL2SQL_THRESHOLD_SCORE10=7.0` so only intent scores ≥7/10 are treated as NL2SQL. |
| `INTENT_SYSTEM` | CLOB prompt/component | Used | CHATBOT_CORE_SQL_PKG.txt (literal), CHATBOT_CORE_SQL_PKG.txt (load_component) | System prompt used by the intent classifier LLM (returns NL2SQL vs GENERAL_CHAT vs FOLLOWUP_CLARIFY, etc.). | Update `INTENT_SYSTEM` to adjust how the model behaves (e.g., enforce SQL-only output or change answer tone) without changing code. |
| `MAX_EXAMPLES_CHARS` | Number | Unknown | — | Maximum characters allowed for any example/few-shot block after clamping. | Set `MAX_EXAMPLES_CHARS=2000` to cap the associated prompt/input block at 2,000 characters. |
| `MAX_FULL_JSON_CHARS` | Number | Unknown | — | Maximum characters allowed when returning full result JSON (safety cap). | Set `MAX_FULL_JSON_CHARS=2000` to cap the associated prompt/input block at 2,000 characters. |
| `MAX_HISTORY_CHARS` | Number | Unknown | — | Maximum characters allowed for history included in prompts after clamping. | Set `MAX_HISTORY_CHARS=2000` to cap the associated prompt/input block at 2,000 characters. |
| `MAX_INSTRUCTION_CHARS` | Number | Unknown | — | Maximum characters allowed for the NL2SQL instruction block after clamping. | Set `MAX_INSTRUCTION_CHARS=2000` to cap the associated prompt/input block at 2,000 characters. |
| `MAX_REPHRASE_CHARS` | Number | Unknown | — | Maximum characters allowed for the rephrased question (or rephrase prompt inputs) after clamping. | Set `MAX_REPHRASE_CHARS=2000` to cap the associated prompt/input block at 2,000 characters. |
| `MAX_SCHEMA_CHARS` | Number | Referenced in code (missing from CSV) | CHATBOT_CORE_SQL_PKG.txt (get_num_param) | Maximum characters allowed for the schema block (SCHEMA slice or full) after clamping. | Set `MAX_SCHEMA_CHARS=2000` to cap the associated prompt/input block at 2,000 characters. |
| `MAX_TABLEDESC_CHARS` | Number | Referenced in code (missing from CSV) | CHATBOT_CORE_SQL_PKG.txt (get_num_param) | Maximum characters allowed for the table descriptions block after clamping. | Set `MAX_TABLEDESC_CHARS=2000` to cap the associated prompt/input block at 2,000 characters. |
| `NL2SQL_INSTRUCTION` | CLOB prompt/component | Used | CHATBOT_CORE_RUNNER_PKG.txt (literal) | Primary instruction/prompt template for the NL2SQL generation phase (may fall back to INSTRUCTION). | Update `NL2SQL_INSTRUCTION` to adjust how the model behaves (e.g., enforce SQL-only output or change answer tone) without changing code. |
| `REASONING_ENABLED` | Scalar (string/flag) | Used | CHATBOT_CORE_RUNNER_PKG.txt (literal) | Flag (TRUE/FALSE) controlling whether the REASONING phase runs. | Set `REASONING_ENABLED=TRUE` to enable the corresponding feature for a dataset/environment. |
| `REASONING_SYSTEM` | CLOB prompt/component | Used | CHATBOT_CORE_RUNNER_PKG.txt (literal) | System prompt for the optional REASONING phase that produces a structured plan JSON before SQL generation. | Update `REASONING_SYSTEM` to adjust how the model behaves (e.g., enforce SQL-only output or change answer tone) without changing code. |
| `REDACTION_ENABLED` | Scalar (string/flag) | Referenced in code (missing from CSV) | CHATBOT_CORE_CHART_PKG.txt (get_param_value), CHATBOT_CORE_SUMMARY_PKG.txt (get_param_value) | Flag controlling whether preview/result JSON is tokenized/redacted before being sent to chart/summary LLMs. | Set `REDACTION_ENABLED=TRUE` to enable the corresponding feature for a dataset/environment. |
| `ROUTER_ENABLED` | Scalar (string/flag) | Used | CHATBOT_CORE_CTX_PARAMS_PKG.txt (get_param_value), CHATBOT_CORE_CTX_PARAMS_PKG.txt (literal) | Flag controlling whether table routing is enabled (routes to a subset of SCHEMA/TABLE_DESCRIPTIONS before NL2SQL). | Set `ROUTER_ENABLED=TRUE` to enable the corresponding feature for a dataset/environment. |
| `ROUTER_SYSTEM` | CLOB (system prompt) | Used | `CHATBOT_ROUTER_PKG.route_tables` | System prompt for the **table router** LLM call; if NULL, a built-in fallback prompt is used.  | Set `ROUTER_SYSTEM` to: “Choose the minimal set of tables and joins; output strict JSON only.” |
| `SCHEMA` | CLOB prompt/component | Used | CHATBOT_CORE_RUNNER_PKG.txt (literal), CHATBOT_CORE_SQL_PKG.txt (literal), CHATBOT_CORE_SQL_PKG.txt (load_component) | Authored schema JSON blocks ("tables":[...]) loaded into prompts; can have multiple rows and is concatenated/deduped upstream. | Provide a JSON document in `SCHEMA` (e.g., a `{"tables":[...]}` structure) so the model can ground joins and columns. |
| `STRICT_SQL_OUTPUT` | CLOB prompt/component | Used | CHATBOT_CORE_RUNNER_PKG.txt (literal) | Additional guardrail prompt forcing the model to output only executable SQL (no markdown/plans/explanations). | Update `STRICT_SQL_OUTPUT` to adjust how the model behaves (e.g., enforce SQL-only output or change answer tone) without changing code. |
| `SUMMARY_SYSTEM` | CLOB prompt/component | Used | CHATBOT_CORE_RUNNER_PKG.txt (literal), CHATBOT_CORE_SUMMARY_PKG.txt (literal), CHATBOT_CORE_SUMMARY_PKG.txt (load_component) | System prompt used to summarize query results (rows/plan/meta) into the final user-facing answer. | Update `SUMMARY_SYSTEM` to adjust how the model behaves (e.g., enforce SQL-only output or change answer tone) without changing code. |
| `TABLE_DESCRIPTIONS` | CLOB prompt/component | Used | CHATBOT_CORE_RUNNER_PKG.txt (literal), CHATBOT_CORE_SQL_PKG.txt (literal), CHATBOT_CORE_SQL_PKG.txt (load_component), CHATBOT_CORE_SUMMARY_PKG.txt (literal) | Authored table description JSON blocks ("tables":[...]) loaded into prompts; can have multiple rows and is concatenated/deduped upstream. | Provide a JSON document in `TABLE_DESCRIPTIONS` (e.g., a `{"tables":[...]}` structure) so the model can ground joins and columns. |
| `TABLE_JOINS` | CLOB prompt/component | Used | CHATBOT_CORE_RUNNER_PKG.txt (literal) | Join metadata JSON describing how tables relate (used by reasoning/router/NL2SQL to avoid invented joins). | Provide a JSON document in `TABLE_JOINS` (e.g., a `{"tables":[...]}` structure) so the model can ground joins and columns. |


# Deployment Manager API (DEPLOY_MGR_PKG)

Home: [README](../README.md) · **Docs** · **Deployment Manager API**

This document describes the **supported Deployment Manager API surface**
intended for operational use.

Only the APIs required for **export** and **import (deployment)** are documented here.

---

## Export Bundle

Exports the current application state into a new bundle ZIP stored in the database.

===sql
FUNCTION export_bundle(
  p_app_id       IN NUMBER,
  p_version_tag  IN VARCHAR2,
  p_include_jobs IN BOOLEAN DEFAULT TRUE
) RETURN NUMBER;
===

### Example
===sql
DECLARE
  l_bundle_id NUMBER;
BEGIN
  l_bundle_id := deploy_mgr_pkg.export_bundle(
                   p_app_id       => 1200,
                   p_version_tag  => 'v1.0.0',
                   p_include_jobs => TRUE
                 );

  DBMS_OUTPUT.PUT_LINE('BUNDLE_ID='||l_bundle_id);
END;
/
===

---

## Import / Deploy Bundle (scheduled job only)

Deployment is performed **as a scheduler job**.

===sql
PROCEDURE enqueue_deploy(
  p_bundle_id         IN NUMBER,
  p_enable_jobs_after IN BOOLEAN DEFAULT FALSE,
  p_dry_run           IN BOOLEAN DEFAULT FALSE,
  o_run_id            OUT NUMBER,
  o_job_name          OUT VARCHAR2
);
===

### Example
===sql
DECLARE
  l_run_id   NUMBER;
  l_job_name VARCHAR2(128);
BEGIN
  deploy_mgr_pkg.enqueue_deploy(
    p_bundle_id         => :BUNDLE_ID,
    p_enable_jobs_after => FALSE,
    p_dry_run           => FALSE,
    o_run_id            => l_run_id,
    o_job_name          => l_job_name
  );

  DBMS_OUTPUT.PUT_LINE('RUN_ID='||l_run_id||' JOB='||l_job_name);
END;
/
===

---

## Notes

- Synchronous deploy APIs are intentionally not documented.
- `p_dry_run` is intended for **update scenarios only**.
- Scheduler jobs defined in `db/ddl/90_jobs.sql` are created disabled by default.
- Deployment status and logs must be checked via deployment run tables or admin UI.

**See also**
- [Deployment Guide](deployment.md)
- [Update Guide](update.md)


# Data Model

Home: [README](../README.md) · **Docs** · **Data Model**

## Overview
The application is built around a **cost-centric analytical data model**, enriched with OCI resource metadata, tagging, relationships, configuration, and logging.

The model is optimized for:
- time-series analytics
- drill-down reporting
- NL2SQL query generation
- traceability and auditability

This document focuses on **logical domains**, not every physical column.

---

## Core Data Domains

### 1. Cost Usage Time Series

These tables store normalized OCI cost and usage data.

**Purpose**
- Daily and monthly cost analysis
- Trend comparison (MoM, QoQ, YoY)
- Normalization for unequal month lengths

**Typical Attributes**
- date / month
- service category
- service name
- charge description
- resource identifier
- cost amounts
- currency

These tables are the **primary fact tables** consumed by dashboards and the chatbot.

---

### 2. OCI Resources

These tables represent OCI resources independently of cost.

**Purpose**
- Display resource inventory
- Correlate cost to logical resources
- Enable resource-centric analytics

**Typical Attributes**
- resource identifier (OCID)
- display name
- resource type
- region
- compartment id
- creation timestamp
- defined tags
- freeform tags

Resources may exist **without cost** for a given period.

---

### 3. Resource Relationships

This domain models **parent–child relationships** between OCI resources.

**Purpose**
- Represent hierarchical services (e.g. clusters → nodes)
- Enable roll-up analytics
- Support chatbot reasoning across resource hierarchies

**Typical Attributes**
- parent identifier
- child identifier
- relationship type

This allows dashboards and NL queries such as:
> “Show cost per cluster including all child resources”

---

### 4. Tag Normalization & Attribution

OCI tagging is **not uniform across environments**.  
This application normalizes tags via configuration.

**Purpose**
- Extract standardized values (e.g. cost center, environment)
- Support consistent filtering and grouping
- Avoid hardcoded tag names

**Typical Flow**
1. Raw tags stored as JSON
2. Tag keys resolved from configuration
3. Values extracted dynamically at query time

This avoids breaking analytics when tag conventions change.

---

### 5. Configuration (APP_CONFIG)

Configuration is entirely **data-driven**.

**Purpose**
- Control application behavior
- Store environment-specific values
- Decouple logic from deployment

**Examples**
- OCI compartment roots
- Tag keys for cost attribution
- Region settings
- Feature toggles
- Chatbot model and routing parameters

No secrets should be committed to GitHub.

Details: [configuration.md](configuration.md)

---

### 6. Chatbot Metadata (NL2SQL)

These tables support the NL2SQL chatbot.

**Purpose**
- Map business language to data model
- Control SQL generation without code changes
- Enable explainability and traceability

**Logical Concepts**
- Glossary rules
- Keywords and synonyms
- Metric definitions
- Filter dimensions
- Grouping dimensions

The chatbot operates on **metadata, not heuristics**.

Details: [chatbot.md](chatbot.md)

---

### 7. Logging & Execution Tracing

All major operations are logged.

**Purpose**
- Debug failures
- Audit chatbot behavior
- Trace background jobs

**Typical Logged Data**
- run / request id
- execution timestamp
- status
- generated SQL
- error messages
- partial outputs

This is critical for operating the system in production.

---

## Data Model Characteristics

- Star-like analytics model (facts + dimensions)
- JSON used for flexible metadata (tags, chatbot rules)
- No hardcoded environment assumptions
- Designed for extensibility

---

## How Dashboards Use the Model

- Dashboards read from:
  - views
  - packaged functions
- No direct writes from APEX pages
- All transformations occur in PL/SQL

---

## How the Chatbot Uses the Model

- Discovers available metrics and dimensions dynamically
- Generates SQL against known fact tables
- Applies filters based on glossary rules
- Logs every step

**See also**
- [Architecture](architecture.md)
- [Chatbot](chatbot.md)


# APEX Pages Map

Home: [README](../README.md) · **Docs** · **APEX Pages Map**

This document maps Oracle APEX pages to functional documentation so users (and admins) can understand where to go in the app and what each page does.

APEX App: **1200**

---

## How to read this page map

- **Core user pages** are the main dashboards, reports, and the chatbot.
- **Admin / maintenance pages** are for configuration, data loading, monitoring, and APEX built-in admin utilities.
- For each page you get:
  - **What it’s for**
  - **Key UI regions** (major page regions)
  - **Server-side processes** (AJAX/on-demand processes and main actions)

---

## Core user pages

| Page | Name | Alias | Purpose | Primary doc |
|---:|---|---|---|---|
| 1 | Home | HOME | Landing dashboard: subscriptions, credits, workload summaries & trends | docs/usage-guide.md#home |
| 2 | OVBot | OVBOT | NL2SQL chatbot UI (chat, dataset/model selection, results dialog) | docs/chatbot.md |
| 3 | Workloads | WORKLOADS | Workload-centric analytics, trends, drilldowns | docs/usage-guide.md#workloads |
| 4 | Usage Report | USAGE-REPORT | Usage-focused reporting (filters → usage views) | docs/usage-guide.md#usage-report |
| 5 | Cost Report | COST-REPORT | Cost-focused reporting (filters → cost views) | docs/usage-guide.md#cost-report |
| 6 | Resource Explorer | RESOURCE-EXPLORER | Resource inventory + attributes + filtering | docs/usage-guide.md#resource-explorer |
| 7 | My Reports | MY-REPORTS | Saved/custom reports area | docs/usage-guide.md#my-reports |
| 8 | OCI Calculator | OCI-CALCULATOR | Calculator utilities for OCI-related calculations | docs/usage-guide.md#oci-calculator |
| 300 | Availability Metrics | AVAILABILITYMETRICS | Availability KPIs/metrics view | docs/usage-guide.md#availability-metrics |

---

## Chatbot & glossary administration (power users)

| Page | Name | Alias | Purpose | Primary doc |
|---:|---|---|---|---|
| 30 | NL2SQL Bot Debug | NL2SQL-BOT-DEBUG | Debug UI for NL2SQL pipeline and generated SQL | docs/chatbot.md#debugging |
| 31 | Chatbot Parameters | CHATBOT-PARAMETERS | Parameter management for chatbot behavior | docs/chatbot.md#configuration |
| 32 | UpdateChatBotParams | UPDATECHATBOTPARAMS | Edit/update chatbot parameters | docs/chatbot.md#configuration |
| 33 | NL2SQL Table Definition Tool | NL2SQL-TABLE-DEFINITION-TOOL1 | Tooling to define/maintain NL2SQL schema metadata | docs/chatbot.md#schema-metadata |
| 34 | UpdateSummaries | UPDATESUMMARIES | Maintain summaries used by chatbot/reporting | docs/chatbot.md#summaries |
| 35 | UpdateBusinessGlossary | UPDATEBUSINESSGLOSSARY | Edit business glossary rules/keywords | docs/chatbot.md#business-glossary |
| 36 | CreateBusinessGlossary | CREATEBUSINESSGLOSSARY | Create new glossary entries | docs/chatbot.md#business-glossary |

---

## Configuration, data loading, and operational tooling (admins)

| Page | Name | Alias | Purpose | Primary doc |
|---:|---|---|---|---|
| 501 | Application Tables | APP-DETAILS | Admin view into core application tables | docs/admin-guide.md#application-tables |
| 502 | Create Subscription Detail | CREATE-SUBSCRIPTION-DETAIL | Manage subscription detail records | docs/admin-guide.md#subscriptions |
| 503 | Create Workload | CREATE-WORKLOAD | Create workloads (metadata/config used in analytics) | docs/admin-guide.md#workloads-admin |
| 504 | UpdateMyReports | UPDATEMYREPORTS | Admin/maintenance for saved reports | docs/admin-guide.md#my-reports-admin |
| 505 | My Reports & Workloads | MY-REPORTS-WORKLOADS | Mapping reports to workloads / combined admin view | docs/admin-guide.md#my-reports-workloads |
| 506 | Data Load | DATA-LOAD | Data load operations (staging/import) | docs/admin-guide.md#data-load |
| 508 | Initial Load | INITIAL-LOAD | First-time initialization and baseline ingestion | docs/admin-guide.md#initial-load |
| 507 | Edit Scheduler Job | EDIT-SCHEDULER-JOB | Edit/inspect scheduler job definitions | docs/admin-guide.md#scheduler-jobs |

---

## App shell / authentication / theme

| Page | Name | Alias | Purpose | Primary doc |
|---:|---|---|---|---|
| 0 | Global Page | — | Shared regions/items for all pages (header, JS/CSS hooks, shared UI) | docs/architecture.md#presentation-layer-oracle-apex |
| 9999 | Login Page | LOGIN | Authentication entry point | docs/security.md#authentication |
| 1004 | Theme Switch | THEME-SWITCH | Theme switching support | docs/usage-guide.md#themes |

---

## Built-in / system administration pages (APEX utilities)

These are standard APEX productivity/admin pages included with the app.

| Page | Name | Alias | Purpose |
|---:|---|---|---|
| 10000 | Administration | ADMINISTRATION | Admin landing |
| 10010 | Email Reporting | EMAIL-REPORTING | Email reporting utilities |
| 10030 | About this Application | ABOUT-THIS-APPLICATION | App info/about |
| 10040 | Activity Dashboard | ACTIVITY-DASHBOARD | Usage/activity metrics |
| 10041 | Top Users | TOP-USERS | Top users analytics |
| 10042 | Application Error Log | APPLICATION-ERROR-LOG | Error log viewer |
| 10043 | Page Performance | PAGE-PERFORMANCE | Performance analytics |
| 10044 | Page Views | PAGE-VIEWS | Page view analytics |
| 10045 | Automations Log | AUTOMATIONS-LOG | Automations log |
| 10046 | Log Messages | LOG-MESSAGES | Application log messages |
| 10050 | Configuration Options | CONFIGURATION-OPTIONS | APEX configuration options |
| 10060 | Feedback | FEEDBACK | Feedback entry |
| 10061 | Feedback Submitted | FEEDBACK-SUBMITTED | Feedback confirmation |
| 10063 | Manage Feedback | MANAGE-FEEDBACK | Feedback admin |
| 10064 | Feedback | FEEDBACK1 | Additional feedback page |
| 10070 | Theme Style Selection | THEME-STYLE-SELECTION | Theme style selection |
| 10080 | App Export | APP-EXPORT | Export utilities |
| 10081 | App Import | APP-IMPORT | Import utilities |
| 10082 | App Bundle Logs | APP-IMPORT-LOGS | Bundle import logs |
| 10083 | Download App | DOWNLOAD-APP | Download utilities |

---

# Per-page details (from export)

Below is the extracted detail for the most important pages. (Regions are the major page regions; processes are key server-side actions.)

## Page 1 — Home (HOME)

**What it’s for**
- Primary landing dashboard: subscription overview, credits, projections, workload costs, monthly/weekly trends.

**Key UI regions**
- Focus Cost Reporting
- Region Selector
- Subscriptions
- Subscription Data (No-PAYG)
- Credit Summary
- Projection Filters
- Subscription Data (PAYG)
- Credit Consumption (PAYG)
- Credit Consumption Chart (PAYG)
- Workload Costs
- Workload Monthly
- Workload 8 weeks
- Consolidated Workload Costs
- Consolidated Cost Per Workload Monthly

**Key server-side processes (AJAX/on-demand)**
- Update Credit Summary Cards
- Update Credit Summary Cards 30 days
- GET_WRKLD_COSTS_MONTHLY
- GET_WRKLD_COSTS_BY_RN
- GET_WRKLD_WORKLOADS_LIST
- GET_WRKLD_WEEKLY_BY_RN

---

## Page 2 — OVBot (OVBOT)

**What it’s for**
- Chat interface for NL2SQL: manage conversations, select dataset/model, run NL question → SQL → results.

**Key UI regions**
- Conversation Management
- Select Chat
- ChatBot
- Chat Input Bar
- Select Dataset
- DataReportDialog

**Key server-side processes**
- Init Chat ID
- RUN_AI_PROCESS
- RESET_AI_STATE
- SET_OR_CLEAR_P2_SQL
- SAVE_CHAT_TITLE

---

## Page 3 — Workloads (WORKLOADS)

**What it’s for**
- Workload-based cost analytics with interactive charts and drill-downs.

**Key UI regions** (high-level)
- (Extracted regions exist; expanded functional walkthrough is in docs/usage-guide.md#workloads)

**Key server-side processes**
- (On-demand processes present; documented in the workload guide section)

---

## Page 4 — Usage Report (USAGE-REPORT)

**What it’s for**
- Usage-centric reporting with filterable dimensions and time ranges.

---

## Page 5 — Cost Report (COST-REPORT)

**What it’s for**
- Cost-centric reporting with filterable dimensions and time ranges.

---

## Page 6 — Resource Explorer (RESOURCE-EXPLORER)

**What it’s for**
- Browse/search resources, attributes, and tags; supports discovery and drilldowns.

---

## Page 7 — My Reports (MY-REPORTS)

**What it’s for**
- Access saved reports and report sets.

---

## Page 8 — OCI Calculator (OCI-CALCULATOR)

**What it’s for**
- Calculator-style utilities supporting OCI cost analysis workflows.

---

## Page 30 — NL2SQL Bot Debug (NL2SQL-BOT-DEBUG)

**What it’s for**
- Debugging and transparency for NL2SQL: inspect generated SQL, pipeline decisions, and logs.

---

## Pages 31–36 — Chatbot metadata management

**What it’s for**
- Manage chatbot parameters, glossary rules/keywords, summaries, and schema metadata.

---

## Admin pages 501–508

**What it’s for**
- Manage workloads/subscriptions/reports, data loading, job editing, and initial setup operations.

---

**See also**
- [Usage Guide](usage-guide.md)
- [Admin Guide](admin-guide.md)


# Troubleshooting

Home: [README](../README.md) · **Docs** · **Troubleshooting**

## General Approach
All components log their behavior.
Always start by identifying:
- run id
- request id
- timestamp

---

## Common Issues

### 1. Empty Dashboards
Possible causes:
- missing cost data
- incorrect compartment config
- wrong date range defaults

Check:
- APP_CONFIG values
- cost refresh job logs

---

### 2. Job Failures
Possible causes:
- OCI permission changes
- malformed input data
- schema changes

Check:
- job run logs
- error stack in logging tables

---

### 3. Chatbot Returns No Results
Possible causes:
- missing glossary coverage
- ambiguous input
- no data for period

Check:
- chatbot execution logs
- generated SQL
- applied filters

---

### 4. Incorrect Cost Attribution
Possible causes:
- tag key mismatch
- incomplete tagging
- resource relationship gaps

Check:
- tag configuration
- raw tag JSON
- relationship tables

---

## Logging Tables

The system logs:
- job execution
- chatbot requests
- SQL generation
- errors and warnings

Logs are designed to be:
- queryable
- correlated via IDs
- retained for audit

---

## Debug Strategy

1. Reproduce the issue
2. Capture run/request id
3. Inspect generated SQL
4. Validate configuration
5. Adjust glossary or config if needed

---

## Support Checklist

Before raising an issue:
- capture timestamps
- capture request id
- export logs
- note environment

**See also**
- [Admin Guide](admin-guide.md)
- [Deployment Guide](deployment.md)
