# Deployment Manager API (DEPLOY_MGR_PKG)

This document describes the public PL/SQL API exposed by `DEPLOY_MGR_PKG` used to export and deploy application bundles.

The Deployment Manager is designed to:
- store bundles as ZIP **BLOBs**
- deploy directly from the ZIP (no extraction required)
- log all activity into deployment run tables
- optionally enqueue deployments as scheduler jobs

---

## Package: `DEPLOY_MGR_PKG`

### `FUNCTION export_bundle(...) RETURN NUMBER`

Exports the current application state into a new bundle record (ZIP + manifest), stored in the deployment tables.

**Signature**
```sql
FUNCTION export_bundle(
  p_app_id       IN NUMBER,
  p_version_tag  IN VARCHAR2,
  p_include_jobs IN BOOLEAN DEFAULT TRUE
) RETURN NUMBER;
```

**Parameters**
- `p_app_id` – APEX application id to export (e.g. `1200`)
- `p_version_tag` – version label stored with the bundle (e.g. `2026.01.09`)
- `p_include_jobs` – include scheduler job export content in the bundle

**Returns**
- `BUNDLE_ID` (NUMBER) identifying the stored bundle.

---

### `PROCEDURE deploy_bundle(...)`

Deploys a bundle **synchronously** in the current session.

**Signature**
```sql
PROCEDURE deploy_bundle(
  p_bundle_id         IN NUMBER,
  p_enable_jobs_after IN BOOLEAN DEFAULT FALSE,
  p_dry_run           IN BOOLEAN DEFAULT FALSE
);
```

**Parameters**
- `p_bundle_id` – bundle identifier to deploy
- `p_enable_jobs_after` – if `TRUE`, jobs created by the bundle are enabled at the end
- `p_dry_run` – if `TRUE`, runs through the deployment flow under logging without making persistent changes where supported

**Behavior**
- Starts a deployment run record
- Reads the ZIP BLOB for `p_bundle_id`
- Applies DDL/scripts and imports the APEX app included in the bundle
- Logs progress and failures into deployment run logs

---

### `PROCEDURE enqueue_deploy(...)`

Enqueues a deployment as a **DBMS_SCHEDULER job** and returns the created run id and job name.

**Signature** (from implementation)
```sql
PROCEDURE enqueue_deploy(
  p_bundle_id         IN NUMBER,
  p_enable_jobs_after IN BOOLEAN DEFAULT FALSE,
  p_dry_run           IN BOOLEAN DEFAULT FALSE,
  o_run_id            OUT NUMBER,
  o_job_name          OUT VARCHAR2
);
```

**Parameters**
- `p_bundle_id` – bundle identifier to deploy
- `p_enable_jobs_after` – whether to enable jobs at the end of deployment
- `p_dry_run` – dry-run mode
- `o_run_id` – OUT: deployment run identifier created by the manager
- `o_job_name` – OUT: scheduler job name created to perform the deployment

**Behavior**
- Creates a run record (action `DEPLOY`)
- Creates a scheduler job of type `STORED_PROCEDURE`
- Job action points to `DEPLOY_MGR_PKG.DEPLOY_WORKER`
- Passes arguments as strings (`'Y'/'N'`) for scheduler compatibility
- Enables the job (auto-drop)

Use this for non-interactive deployments, long-running installs, or when you want deployment to continue outside a client session.

---

### `PROCEDURE deploy_worker(...)`

Scheduler-compatible deployment entry point (no BOOLEAN parameters).

**Signature**
```sql
PROCEDURE deploy_worker(
  p_run_id            IN NUMBER,
  p_bundle_id         IN NUMBER,
  p_enable_jobs_after IN VARCHAR2, -- 'Y'/'N'
  p_dry_run           IN VARCHAR2  -- 'Y'/'N'
);
```

**Parameters**
- `p_run_id` – deployment run id (created by `enqueue_deploy`)
- `p_bundle_id` – bundle identifier to deploy
- `p_enable_jobs_after` – `'Y'` or `'N'`
- `p_dry_run` – `'Y'` or `'N'`

---

### `PROCEDURE exec_script_clob(...)`

Executes an arbitrary SQL/PLSQL script under deployment logging.

**Signature**
```sql
PROCEDURE exec_script_clob(
  p_script      IN CLOB,
  p_script_name IN VARCHAR2,
  p_dry_run     IN BOOLEAN DEFAULT FALSE
);
```

**Use cases**
- Apply one-off fixes under the same run logging model
- Execute migrations or patch scripts with consistent logging

---

### Utility Functions

#### `FUNCTION get_latest_bundle_id(p_app_id IN NUMBER) RETURN NUMBER;`
Returns the most recent bundle id for an APEX application id.

#### `FUNCTION blob_to_clob(p_blob IN BLOB) RETURN CLOB;`
Converts BLOB to CLOB (used for reading ZIP members / manifest text where needed).

#### `FUNCTION clob_to_blob(p_clob IN CLOB) RETURN BLOB;`
Converts CLOB to BLOB (used for writing text into ZIP members / storing content).

---

## Logging Model

Deployments are tracked in deployment tables created by the Deployment Manager DDL.
The run record typically includes:
- `RUN_ID`
- action (e.g. `DEPLOY`)
- status
- timestamps
- `LOG_CLOB` with step-by-step output and errors

Operational guidance:
- Always capture the `RUN_ID` when deploying
- Use the run log as the primary source of truth for failures

---

## Recommended Usage Patterns

### Synchronous deploy (interactive)
```sql
BEGIN
  deploy_mgr_pkg.deploy_bundle(
    p_bundle_id         => :BUNDLE_ID,
    p_enable_jobs_after => FALSE,
    p_dry_run           => FALSE
  );
END;
/
```

### Enqueued deploy (recommended for long runs)
```sql
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
```

---

## Notes

- Always run `ov_run_as_admin.sql` before using the Deployment Manager on a new target ADB.
- Enable scheduler jobs only after configuration (`APP_CONFIG`) has been verified.
