# Chroma DB — Unauthorized Information Disclosure

**Date:** 2026-01-08
**Reporter:** Shay Ben Tikva
**Severity:** High

---

## Executive summary 🔍
- **Target:** `https://hakc.com/` (example host tested).
- **Summary:** Public-facing Chroma endpoints (notably the collections endpoints) return collection metadata and vector configuration details without authentication. The application also exposes a `/docs` Swagger UI and an `openapi.json` which reveal additional API surfaces. Public trace headers (e.g., `Chroma-Trace-Id`) were also observed and can be used to locate additional instances.
- **Impact:** High — sensitive dataset metadata and vector configuration are exposed and Swagger/OpenAPI exposure increases the attack surface.

---

## Affected hosts & endpoints
- Example host: `https://hakc.com/`
- Observed endpoints:
  - `GET /api/v1/collections?tenant=default_tenant&database=default_database`
  - `GET /api/v2/tenants/default_tenant/databases/default_database/collections`
  - `GET /docs` (Swagger UI)
  - `GET /openapi.json` (OpenAPI spec)
- Observed header: `Chroma-Trace-Id: 0` (trace header seen in responses)

---

## Evidence & observations 📋
- Sample response (sanitized):

```json
[{
  "id": "7c807fbe-9717-4b44-8a06-046106554183",
  "name": "knowledge_base",
  "configuration_json": {"hnsw_configuration":{"space":"l2","ef_construction":100}},
  "dimension": 1536,
  "tenant": "default_tenant",
  "database": "default_database"
}]
```

- Running a simple `curl` against the collections endpoints returned non-empty arrays without authentication.
- Searching your scan logs for the header `chroma-trace-id` returned ~5,744 hits (indicates widespread header exposure in your environment).
- Visiting `/docs` served a Swagger UI; fetching `/openapi.json` revealed endpoint definitions (some endpoints returned errors such as `Internal Server Error`, but it was possible to access usable API surface in other cases).

> Note: No destructive actions were taken. Only minimal, unauthenticated GET requests and directory/db enumeration used for the report.

---
