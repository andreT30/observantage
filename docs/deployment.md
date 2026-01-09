# Deployment

## Overview

This application is deployed and upgraded using a **database‑resident Deployment Manager**
implemented in PL/SQL (`DEPLOY_MGR_PKG`).  
The Deployment Manager consumes the **application bundle ZIP as a ZIP (BLOB)** and applies its
contents directly—**no manual extraction is required or expected**.

Deployment is deterministic, logged, and repeatable.

---

## Deployment Components

### 1. Admin prerequisite script (mandatory)
**File:** `ov_run_as_admin.sql`  
**Executed as:** Autonomous Database ADMIN (or equivalent privileged user)

This script performs **one‑time or infrequent** privileged actions, including:

- Grants on system packages (DBMS_CLOUD, DBMS_SCHEDULER, DBMS_METADATA, DBMS_LOB, etc.)
- Scheduler and job‑related privileges
- APEX execution and import privileges
- Any required cross‑schema grants

This script **must be executed before any deployment** using the Deployment Manager.

---

### 2. Deployment Manager installation (application schema)

The Deployment Manager lives in the **application schema** and consists of:

- `deploy_manager_ddl.sql`
- `deploy_manager_pkg.sql`

These create:
- bundle storage tables (ZIP stored as BLOB)
- deployment run / log tables
- the `DEPLOY_MGR_PKG` API

Install or upgrade the Deployment Manager by running, as the application schema owner:

@deploy_manager_ddl.sql  
@deploy_manager_pkg.sql

The package is idempotent and may be reapplied during upgrades.

---


## Deployment Manager API Reference

See: `docs/deploy-manager-api.md`

## Bundle ZIP

The Deployment Manager operates on the **bundle ZIP as a single binary artifact**.

The ZIP typically contains:
- `db/ddl/*.sql` — database objects
- `db/ddl/90_jobs.sql` — scheduler jobs (optional)
- `apex/f1200.sql` — APEX application export
- `manifest.json` — bundle metadata

The ZIP is read directly from a database BLOB and processed internally.

---

## Deployment Procedure (Target ADB)

### Step 0 — Run admin script (once per environment)

Connect as ADMIN and run:

@ov_run_as_admin.sql

---

### Step 1 — Ensure Deployment Manager is installed

Connect as the application schema and run:

@deploy_manager_ddl.sql  
@deploy_manager_pkg.sql

---

### Step 2 — Insert bundle ZIP into the database

Insert the ZIP into the Deployment Manager bundle table
(e.g. `DEPLOY_BUNDLES`) along with identifying metadata
(application id, version tag, manifest JSON).

Capture the generated `BUNDLE_ID`.

---

### Step 3 — Deploy the bundle

Execute:

BEGIN
  deploy_mgr_pkg.deploy_bundle(
    p_bundle_id         => :BUNDLE_ID,
    p_enable_jobs_after => FALSE,
    p_dry_run           => FALSE
  );
END;
/

Notes:
- Jobs should remain **disabled** on first deployment
- `p_dry_run => TRUE` can be used to validate deployment logic and logging

---

### Step 4 — Validate deployment

Verify:
- APEX application opens successfully
- Core pages render without error
- Chatbot page initializes
- Required `APP_CONFIG` entries exist

---

### Step 5 — Enable scheduler jobs

After configuration validation, jobs may be enabled by:
- redeploying with `p_enable_jobs_after => TRUE`, or
- enabling jobs explicitly per operational policy

---

## Logging & Observability

Each deployment execution is recorded with:
- status
- timestamps
- full execution log (CLOB)

Deployment failures are diagnosable directly from deployment run logs.

---

## Upgrade Process

Upgrades follow the **same process** as initial deployment:

1. Run `ov_run_as_admin.sql` if privileges changed
2. Ensure Deployment Manager is current
3. Insert new bundle ZIP
4. Deploy via `DEPLOY_MGR_PKG`
5. Validate
6. Enable jobs if applicable

---

## Common Issues

- Admin script not executed → missing privileges
- Jobs enabled before configuration
- Missing `APP_CONFIG` values
- Scheduler permission errors

See also: `docs/troubleshooting.md`

## Appendix: Admin Grants (ov_run_as_admin.sql)

This appendix lists the privileged grants applied by `ov_run_as_admin.sql` (executed as ADMIN) to the application schema.
These are required for bundle deployment, job management, OCI data access, and related platform features.

### Legacy roles

- `GRANT "CONNECT" TO OCI_FOCUS_REPORTS;`
- `GRANT "RESOURCE" TO OCI_FOCUS_REPORTS;`

### ADB built-in roles

- `GRANT "DWROLE" TO OCI_FOCUS_REPORTS;`
- `GRANT "CONSOLE_DEVELOPER" TO OCI_FOCUS_REPORTS;`
- `GRANT "OML_DEVELOPER" TO OCI_FOCUS_REPORTS;`

### Read access

- `GRANT SELECT ANY TABLE TO OCI_FOCUS_REPORTS;`

### Scheduler / jobs

- `GRANT CREATE ANY JOB TO OCI_FOCUS_REPORTS;`
- `grant create job to oci_focus_reports;`

### Application context

- `GRANT CREATE ANY CONTEXT TO oci_focus_reports;`

### DDL privileges

- `grant create table to oci_focus_reports;`
- `grant create sequence to oci_focus_reports;`
- `grant create view to oci_focus_reports;`
- `grant create materialized view to oci_focus_reports;`
- `grant create procedure to oci_focus_reports;`
- `grant create trigger to oci_focus_reports;`

### DBMS packages

- `GRANT EXECUTE ON C##CLOUD$SERVICE.DBMS_CLOUD_OCI_OS_ORGANIZATION_SUBSCRIPTION_LIST_ORGANIZATION_SUBSCRIPTIONS_RESPONSE_T TO OCI_FOCUS_REPORTS;`
- `GRANT EXECUTE ON c##cloud$service.dbms_cloud_oci_os_subscription_list_subscriptions_response_t TO OCI_FOCUS_REPORTS;`
- `GRANT EXECUTE ON C##CLOUD$SERVICE.DBMS_CLOUD_OCI_OS_ORGANIZATION_SUBSCRIPTION TO OCI_FOCUS_REPORTS;`
- `GRANT EXECUTE ON C##CLOUD$SERVICE.DBMS_CLOUD_OCI_OS_SUBSCRIPTION TO OCI_FOCUS_REPORTS;`
- `GRANT EXECUTE ON C##CLOUD$SERVICE.DBMS_CLOUD_OCI_ONESUBSCRIPTION_SUBSCRIPTION_CURRENCY_T TO OCI_FOCUS_REPORTS;`
- `GRANT EXECUTE ON C##CLOUD$SERVICE.DBMS_CLOUD_OCI_ONESUBSCRIPTION_SUBSCRIPTION_SUBSCRIBED_SERVICE_TBL TO OCI_FOCUS_REPORTS;`
- `GRANT EXECUTE ON C##CLOUD$SERVICE.DBMS_CLOUD_OCI_ONESUBSCRIPTION_SUBSCRIPTION_PRODUCT_T TO OCI_FOCUS_REPORTS;`
- `GRANT EXECUTE ON C##CLOUD$SERVICE.DBMS_CLOUD_OCI_ONESUBSCRIPTION_COMMITMENT_SERVICE_TBL TO OCI_FOCUS_REPORTS;`
- `grant execute on C##CLOUD$SERVICE.DBMS_CLOUD_OCI_MN_MONITORING to OCI_FOCUS_REPORTS;`
- `grant execute on DBMS_CLOUD_OCI_ID_IDENTITY to OCI_FOCUS_REPORTS;`
- `grant execute on DBMS_CLOUD_OCI_RS_RESOURCE_SEARCH to OCI_FOCUS_REPORTS;`
- `grant execute on DBMS_CLOUD_OCI_DB_DATABASE_LIST_MAINTENANCE_RUN_HISTORY_RESPONSE_T to OCI_FOCUS_REPORTS;`
- `grant execute on DBMS_CLOUD_OCI_DB_DATABASE to OCI_FOCUS_REPORTS;`
- `grant execute on DBMS_CLOUD_OCI_DATABASE_MAINTENANCE_RUN_HISTORY_SUMMARY_T to OCI_FOCUS_REPORTS;`
- `grant execute on DBMS_CLOUD_OCI_DATABASE_MAINTENANCE_RUN_HISTORY_SUMMARY_TBL to OCI_FOCUS_REPORTS;`
- `grant execute on DBMS_CLOUD_OCI_DATABASE_UPDATE_AUTONOMOUS_DATABASE_DETAILS_T to OCI_FOCUS_REPORTS;`
- `grant execute on dbms_cloud_oci_monitoring_list_metrics_details_t to OCI_FOCUS_REPORTS;`
- `grant execute on dbms_cloud_oci_monitoring_summarize_metrics_data_details_t to OCI_FOCUS_REPORTS;`
- `grant execute on c##cloud$service.dbms_cloud_oci_mn_monitoring_list_metrics_response_t to OCI_FOCUS_REPORTS;`
- `grant execute on c##cloud$service.dbms_cloud_oci_mn_monitoring_summarize_metrics_data_response_t to OCI_FOCUS_REPORTS;`
- `grant execute on DBMS_CLOUD_OCI_DB_DATABASE_UPDATE_AUTONOMOUS_DATABASE_RESPONSE_T to OCI_FOCUS_REPORTS;`
- `grant execute on C##CLOUD$SERVICE.DBMS_CLOUD_OCI_DATABASE_MAINTENANCE_RUN_SUMMARY_T to OCI_FOCUS_REPORTS;`
- `grant execute on C##CLOUD$SERVICE.DBMS_CLOUD_OCI_DATABASE_ESTIMATED_PATCHING_TIME_T to OCI_FOCUS_REPORTS;`
- `grant execute on DBMS_CLOUD_OCI_ID_IDENTITY_LIST_COMPARTMENTS_RESPONSE_T to OCI_FOCUS_REPORTS;`
- `grant execute on DBMS_CLOUD_OCI_DB_DATABASE_LIST_EXADATA_INFRASTRUCTURES_RESPONSE_T to OCI_FOCUS_REPORTS;`
- `grant execute on DBMS_CLOUD_OCI_DB_DATABASE_LIST_CLOUD_EXADATA_INFRASTRUCTURES_RESPONSE_T to OCI_FOCUS_REPORTS;`
- `grant execute on c##cloud$service.dbms_cloud_oci_rs_resource_search_search_resources_response_t to OCI_FOCUS_REPORTS;`
- `grant execute on c##cloud$service.dbms_cloud_oci_resource_search_structured_search_details_t to OCI_FOCUS_REPORTS;`
- `grant execute on dbms_cloud_oci_identity_compartment_tbl to OCI_FOCUS_REPORTS;`
- `GRANT EXECUTE ON DBMS_RESULT_CACHE TO OCI_FOCUS_REPORTS;`
- `GRANT EXECUTE ON DBMS_CLOUD TO OCI_FOCUS_REPORTS;`
- `GRANT EXECUTE ON DBMS_CRYPTO TO OCI_FOCUS_REPORTS;`
- `GRANT EXECUTE ON DBMS_CLOUD_PIPELINE TO OCI_FOCUS_REPORTS;`
- `GRANT EXECUTE ON DBMS_CLOUD_AI TO OCI_FOCUS_REPORTS;`

### Object grants

- `GRANT EXECUTE ON CS_SESSION TO OCI_FOCUS_REPORTS;`
- `GRANT EXECUTE ON OCI$RESOURCE_PRINCIPAL TO OCI_FOCUS_REPORTS;`
