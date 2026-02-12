# 🎯 Backend Foundation - Salesforce Kaptio Data MVP
> **Version**: v1.0  
> **Status**: ✅ COMPLETED  
> **Date**: 2026-02-06  
> **Completion Date**: 2026-02-06

**Target Components**: `SwainOS_BackEnd/src/core/`, `SwainOS_BackEnd/src/api/`, `SwainOS_BackEnd/src/schemas/`, `SwainOS_BackEnd/src/services/`, `SwainOS_BackEnd/src/repositories/`, `SwainOS_BackEnd/src/models/`, `SwainOS_BackEnd/src/integrations/salesforce/`, `SwainOS_BackEnd/src/analytics/`, `SwainOS_BackEnd/src/shared/`, `SwainOS_BackEnd/tests/`  
**Primary Issues**: Backend implementation not yet established for Salesforce Kaptio data access and cashflow/deposit/forecasting workflows.  
**Objective**: Stand up a clean FastAPI foundation and deliver a Salesforce Kaptio data MVP that powers cashflow, customer deposits, payments out, and booking forecasts.

## 📋 QUICK SUMMARY
**What We're Building/Fixing**: A clean FastAPI base plus Salesforce Kaptio data access and analytics for cashflow, deposits, payments out, and booking forecasts.

**Critical Issues Being Addressed**:
- Missing backend runtime foundation → establish app core, error handling, and configuration.
- No Salesforce Kaptio data access layer → add repository mapping and analytics.
- No cashflow/deposit/payment/forecast layer → add analytics services and API endpoints.
- No validation/test coverage for Phase 1 → add API/service tests plus error-path coverage.

**Success Metrics**:
- `GET /api/v1/health` returns success envelope.
- Salesforce Kaptio data is available via Supabase and readable through APIs.
- Cashflow, deposits, and payments out metrics available via API for a single company.
- Booking forecasts based on historical trends available via API.
- Type checking and linting pass (mypy + lint).
- `docs/swainos-code-documentation-backend.md` updated for new modules.

---

## ✅ GOALS ALIGNMENT AUDIT
**Aligned Goals**:
- Centralize business data → Salesforce Kaptio ingestion + normalized storage.
- Real-time cash flow visibility → cashflow + deposits + payments out APIs.
- Reduce manual spreadsheet workflows → automated sync + standardized analytics outputs.
- Forecasting accuracy → trend-based booking forecast endpoint.

**Gaps to Watch**:
- FX optimization and AI briefings are out of scope for this MVP (tracked for later phases).
- QuickBooks P&L sync not included (Phase 6).
- Debt tracking not included (Phase 1 alternate path; revisit after Kaptio MVP).

---

## 🎯 EXECUTION STATUS
**Progress**: 2 of 2 sections completed  
**Current Status**: Implementation complete → ready for next phase.

| Phase | Status | Priority | Notes |
|-------|---------|----------|-------|
| 1️⃣ Backend Foundation | ✅ COMPLETED | HIGH | Core app, config, errors, health |
| 2️⃣ Salesforce Kaptio MVP | ✅ COMPLETED | HIGH | Analytics + read APIs |

---

## 🚨 CRITICAL REQUIREMENTS
### ⚠️ MANDATORY CHECKLIST - Must Complete Before Implementation
These requirements are NON-NEGOTIABLE for every action plan. Do not skip any item.

- [x] **Type Safety**: All new code uses explicit type hints — no `Any` types.
- [x] **Naming Conventions**: All files, functions, and variables follow backend conventions.
- [x] **Import Organization**: Standard Python import order with blank lines.
- [x] **Formatting**: Black + isort + mypy clean.
- [x] **Documentation Update**: `docs/swainos-code-documentation-backend.md` updated.
- [x] **No Dead Code**: No commented-out code, no unused imports/variables.
- [x] **No Debug Prints**: Use structured logging.
- [x] **Security**: Validate input, parameterized queries, and RLS-aware access.

### Documentation Update Requirement
Every action plan that modifies backend code MUST update `docs/swainos-code-documentation-backend.md`.

---

## 📐 NAMING CONVENTION ALIGNMENT
### Files & Directories
- Files/modules: `snake_case.py`
- Classes: `PascalCase`
- Functions/variables: `snake_case`
- Constants: `SCREAMING_SNAKE_CASE`
- API: `/api/v1/resource`, `snake_case` query params
- DB: `snake_case` columns, plural tables

---

## 🧹 CLEAN CODE REQUIREMENTS
### Import Organization Standard (Python)
1. Standard library  
2. Third-party  
3. Internal (local project)  

Blank line between groups, alphabetized within groups.

---

## 1️⃣ Backend Foundation
*Priority: High - establish core backend runtime*

### 🎯 Objective
Create a stable FastAPI foundation with config, logging, error envelope, and health endpoint.

### 🔍 Analysis / Discovery
- Verify existing Supabase schema, RLS policies, and migrations for Salesforce/Kaptio data.
- Confirm error envelope shape and response contract (consistent `{ data, pagination }` with structured errors).
- Identify required auth mechanism and request context needed for protected endpoints.

### ⚙️ Implementation
**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `src/main.py` | Create/Modify | FastAPI app setup and router registration |
| `src/core/config.py` | Create/Modify | Settings, env loading, defaults |
| `src/core/logging.py` | Create/Modify | Structured logging config |
| `src/core/errors.py` | Create/Modify | Domain errors + error envelope |
| `src/api/health.py` | Create | Health endpoint |
| `src/api/router.py` | Create/Modify | API versioned router |

**Implementation Steps:**
1. Confirm existing scaffold in `src/` and align with README structure.
2. Implement structured error envelope and global exception handlers.
3. Add `/api/v1/health` and `/api/v1/healthz` as readiness/liveness checks.
4. Establish API version router and standard response helper.

### ✅ Validation Checklist
- [x] App starts and registers versioned routers
- [x] Health endpoints return `{ data, pagination }` envelope
- [x] Errors return consistent envelope with trace-safe messaging
- [x] Auth context available for protected endpoints (even if unused by health)

---

## 2️⃣ Salesforce Kaptio MVP
*Priority: High - deliver Salesforce Kaptio ingestion and analytics*

### 🎯 Objective
Support Salesforce Kaptio data access from Supabase, plus cashflow, customer deposits, payments out, and booking forecasts via API.

### 🔍 Analysis / Discovery
- Map Salesforce Kaptio objects (bookings, contacts, deposits, payments out, invoices) to current schema.
- Confirm required fields for cashflow, deposits, payments out, and forecast calculations.
- Identify missing tables or columns needed for ingestion and analytics.

### ⚙️ Implementation
**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `src/integrations/salesforce/client.py` | Deferred | Live sync not in this phase |
| `src/jobs/salesforce_sync.py` | Deferred | Live sync not in this phase |
| `src/schemas/salesforce.py` | Create/Modify | Ingestion schema validation |
| `src/models/revenue_bookings.py` | Create/Modify | Normalized booking + financial models |
| `src/repositories/revenue_bookings_repository.py` | Create/Modify | DB access + mapping |
| `src/analytics/cash_flow.py` | Create/Modify | Cashflow calculations |
| `src/analytics/booking_forecast.py` | Create/Modify | Forecast logic |
| `src/api/revenue_bookings.py` | Create/Modify | REST endpoints |
| `tests/revenue_bookings/` | Create | API/service tests |

**Implementation Steps:**
1. Define core entities: bookings, contacts, deposits, payments out, invoices, and cashflow snapshots.
2. Implement repository layer with explicit DB ↔ API mapping and parameterized queries.
3. Expose endpoints for cashflow, deposits, payments out, booking pipeline, and forecast outputs.
4. Add tests for analytics outputs and error paths (validation, not found, permission).

### ✅ Validation Checklist
- [x] Mapping respects `snake_case` DB → `camelCase` API
- [x] Pagination and filtering validated for list endpoints
- [x] Forecast outputs deterministic for fixed test data
- [x] mypy and lint pass without warnings

---

## ⚠️ RISK MANAGEMENT
### High Priority Risks
- **Schema/API drift**: Mismatch between migrations and API models → **Mitigation**: verify models against existing Supabase schema before implementation.
- **RLS/access gaps**: Unauthorized reads/writes → **Mitigation**: enforce RLS-aware repository access patterns and explicit auth context checks.
- **Error envelope inconsistency**: multiple formats → **Mitigation**: enforce shared error helpers in `src/core/errors.py`.

### Rollback Strategy
1. Revert API/router and Salesforce/Kaptio module additions.
2. Remove new tests and doc updates.
3. Verify app starts and health endpoint still returns success.

---

## 📊 SUCCESS CRITERIA
### Technical Success Metrics
| Metric | Target | Verification Method |
|--------|--------|---------------------|
| API readiness | `/api/v1/health` OK | Manual curl |
| Salesforce data access | Data available via Supabase for analytics | API tests |
| Cashflow/deposits/payments | Metrics returned for test company | API tests |
| Forecasting | Trend-based forecast returned | Analytics tests |
| Type safety | Zero mypy errors | `mypy` |
| Lint/format | Zero issues | `black`/`isort` + lint |

### User Experience Success
| Scenario | Expected Outcome |
|----------|------------------|
| Use Supabase-loaded data | Bookings, deposits, payments accessible |
| View cashflow | Metrics returned with totals |
| View forecasts | Booking trend forecast visible |

---

## 🔗 RELATED DOCUMENTATION
- `../docs/success-criteria-and-phases.md`
- `../docs/scope-and-modules.md`
- `../docs/objectives.md`

---

## 📝 REVISION HISTORY
| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.0 | 2026-02-06 | SwainOS Team | Initial action plan |
| v1.1 | 2026-02-06 | SwainOS Team | Marked completed and aligned to Supabase-loaded data |
