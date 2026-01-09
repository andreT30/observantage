# Deployment

## Overview
Deployment consists of:
1. Database schema setup
2. Configuration seeding
3. APEX application import
4. Environment-specific configuration
5. Job enablement

The process is repeatable and environment-agnostic.

---

## Prerequisites

- Oracle Autonomous Database
- Oracle APEX (compatible version)
- OCI permissions to access cost & resource data
- Wallet / credential setup (external to Git)

---

## Deployment Order

### 1. Database Objects
Execute all SQL under:
```db/ddl/```

This creates:
- tables
- views
- packages
- jobs
- indexes
- constraints

---

### 2. Seed Data
Execute:
- configuration seed scripts
- chatbot glossary seed scripts

These create baseline metadata only.

---

### 3. Import APEX Application
Import:
```apex/f1200.sql```

Using:
- APEX App Builder
- SQLcl
- APEX export/import tools

---

### 4. Configure Environment
Update `APP_CONFIG` with:
- compartment identifiers
- region lists
- tag keys
- environment labels

No code changes required.

---

### 5. Enable Jobs
Enable scheduled jobs after configuration is verified.

Jobs typically:
- refresh cost data
- aggregate time series
- sync resource metadata

---

## Post-Deployment Validation

Recommended checks:
- dashboards show data
- filters behave correctly
- chatbot answers basic questions
- jobs run without error

---

## Rollback Strategy

- DB objects are versioned via scripts
- APEX app can be re-imported
- Config values can be reverted independently

---

## CI / CD Considerations

Recommended:
- split APEX exports
- Git-based review
- environment-specific config scripts
- controlled job enablement

