# 🎯 Budget, Forecast, and QuickBooks Operating Model - Lean v1 Planning Foundation

> **Version**: v1.2  
> **Status**: ⏸️ PARKED (NOT STARTED)  
> **Date**: 2026-03-06

**Target Components**: `SwianOS_Documentation/supabase/migrations/`, `SwainOS_BackEnd/src/api/`, `SwainOS_BackEnd/src/services/`, `SwainOS_BackEnd/src/repositories/`, `SwainOS_BackEnd/src/schemas/`, `SwainOS_BackEnd/src/integrations/`, `SwainOS_BackEnd/scripts/`, `SwainOS_FrontEnd/apps/web/src/features/`, `SwainOS_FrontEnd/apps/web/src/lib/api/`, `SwainOS_FrontEnd/apps/web/src/lib/types/`, `SwianOS_Documentation/docs/`  
**Primary Issues**: Current budgeting schema is too flat for versioned planning, QuickBooks actuals are not yet operationalized for budget variance, and there is no AI-ready budget context layer linked to broader platform decisions.  
**Objective**: Ship a fast, production-usable budget vs actual flow in SwainOS with versioned planning, deterministic QuickBooks mapping, and platform-ready data contracts.

## 📋 **QUICK SUMMARY**

**What We're Building/Fixing**: A lean budget and forecast system in SwainOS where QuickBooks supplies actuals and SwainOS owns planning, variance, and AI decision context.

**Critical Issues Being Addressed**:
- Budgeting cannot scale with single-table design -> introduce versioned budget domain with line-level monthly grain.
- QuickBooks connection lacks planning utility -> normalize and map actuals into internal categories.
- AI and command-center cannot reason on budget drift -> publish concise budget/actual/variance context views.

**Success Metrics**:
- At least one baseline budget version and one reforecast version can be created and locked.
- Daily QuickBooks sync runs successfully with auditable run metadata and idempotent behavior.
- Variance APIs match fixture calculations and power frontend/AI surfaces.
- Category mapping coverage reaches 100% for material P&L accounts before close.
- Architecture cleanly supports future joins to travel, AP, debt, FX, and cash-flow.

---

## 🎯 **EXECUTION STATUS**

**Progress**: 0 of 7 sections completed  
**Current Status**: Feature parked and intentionally deferred; treat this plan as a fresh restart point.

| Phase | Status | Priority | Notes |
|-------|---------|----------|-------|
| 1️⃣ Scope and Contract Freeze | 📋 NOT STARTED | HIGH | Deferred |
| 2️⃣ Supabase Planning Schema | 📋 NOT STARTED | HIGH | Deferred |
| 3️⃣ QuickBooks Ingestion + Mapping | 📋 NOT STARTED | HIGH | Deferred |
| 4️⃣ Backend Finance APIs | 📋 NOT STARTED | HIGH | Deferred |
| 5️⃣ Frontend Finance Workflow | 📋 NOT STARTED | HIGH | Deferred |
| 6️⃣ AI and Command-Center Bridge | 📋 NOT STARTED | MEDIUM | Deferred |
| 7️⃣ QA, Runbook, and Docs | 📋 NOT STARTED | HIGH | Deferred |

---

## 🚨 **CRITICAL REQUIREMENTS**

### **⚠️ MANDATORY CHECKLIST - Must Complete Before Implementation**

- [ ] **Type Safety**: Strict typing in backend schemas and frontend contracts; no `any`.
- [ ] **Naming Conventions**: All DB, API, backend, and frontend naming follows SwainOS rules.
- [ ] **Performance First**: Heavy variance aggregation performed in SQL/views, not frontend loops.
- [ ] **ESLint + Quality Clean**: Zero warnings/errors on touched frontend modules.
- [ ] **Documentation Update**: Backend/frontend/query/payload docs updated with finance domain changes.
- [ ] **No Dead Code**: Remove stale budget assumptions once new domain is live.
- [ ] **Type Safety**: Strict typing in new backend schemas/services/repositories; no `any` introduced.
- [ ] **Naming Conventions**: New DB/backend/API contracts follow `snake_case` and JSON `camelCase`.
- [ ] **Performance First**: Variance and AI context are SQL view-driven (`variance_monthly_v1` + AI views).
- [ ] **ESLint + Quality Clean**: Frontend module not started yet (deferred by plan).
- [ ] **Documentation Update**: Backend/query/payload docs updated to current implemented contract.
- [ ] **No Dead Code**: Audit pass removed/avoided new dead code in implemented backend scope.

### **Documentation Update Requirement**

> **⚠️ IMPORTANT**: This rollout must update:
> - `docs/swainos-code-documentation-backend.md`
> - `docs/swainos-code-documentation-frontend.md`
> - `docs/frontend-data-queries.md`
> - `docs/sample-payloads.md`
> - `docs/swainos-terminology-glossary.md` (if needed)

---

## 📐 **NAMING CONVENTION ALIGNMENT**

All code in this plan follows current SwainOS conventions:

| Element | Convention | Example |
|---------|------------|---------|
| Backend modules | `snake_case.py` | `finance_budget_service.py` |
| Frontend components | `kebab-case.tsx` | `budget-variance-panel.tsx` |
| Frontend services | `camelCaseService.ts` | `financeBudgetService.ts` |
| Database tables | `snake_case`, plural | `budget_versions` |
| Database columns | `snake_case` | `month_start`, `budget_amount` |
| API query params | `snake_case` | `fiscal_year`, `version_id` |
| JSON properties | `camelCase` | `budgetAmount`, `variancePct` |

---

## 🔧 **STRATEGIC APPROACH**

### **Implementation Philosophy**
Start with the smallest production-usable planning core, optimize query paths early, and preserve extensibility so budget and actuals can later unify with travel operations, debt, AP, FX, and AI without contract churn.

### **Key Architecture Decisions**
- **SwainOS plans, QuickBooks actuals**: SwainOS is system-of-record for budget versions and reforecasts; QBO is read-only source for actuals.
- **Monthly grain first**: fast, stable, and easy to reason about for finance and AI.
- **Deterministic category mapping**: one mapping boundary to avoid metric drift across modules.
- **View-based variance outputs**: precomputed variance for low-latency frontend and AI access.

### **Data Flow**

```
SwainOS budget input
  -> budget_versions + budget_lines

QuickBooks daily pull
  -> raw/staging payloads
  -> normalized actuals_monthly
  -> account mapping -> financial_categories

Variance computation
  -> variance_monthly_v1
  -> finance APIs
  -> frontend budget/variance pages + AI context views
```

---

## 1️⃣ **SCOPE AND CONTRACT FREEZE**
*Priority: High - define decisions before schema and API implementation*

### **🎯 Objective**
Lock v1 business semantics so implementation stays lean, predictable, and fast.

### **🔍 Analysis / Discovery**
- Confirm v1 supports operating P&L planning by month.
- Freeze lifecycle: `draft`, `approved`, `locked`, `superseded`.
- Lock variance semantics:
  - `varianceAmount = actualAmount - budgetAmount`
  - `variancePct = varianceAmount / nullif(abs(budgetAmount), 0)`
- Freeze non-goals:
  - no advanced ML forecast engine in v1
  - no direct bank feed dependency for initial release
  - no complex multi-entity consolidation unless required

### **⚙️ Implementation**

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `docs/swainos-code-documentation-backend.md` | Modify | Add planning domain contract definitions |
| `docs/swainos-code-documentation-frontend.md` | Modify | Add budget and variance UI surface |
| `docs/sample-payloads.md` | Modify | Add finance endpoint payload examples |

**Implementation Steps:**
1. Finalize metric dictionary and lifecycle states with finance owners.
2. Lock v1 scope and explicit non-goals.
3. Publish contract definitions in docs before coding.

### **✅ Validation Checklist**
- [ ] Scope lock approved.
- [ ] Metric dictionary published.
- [ ] Lifecycle semantics documented.

---

## 2️⃣ **SUPABASE PLANNING SCHEMA**
*Priority: High - create fast and scalable planning data foundation*

### **🎯 Objective**
Implement normalized planning tables with strong constraints and index coverage.

### **🔄 Implementation**

**Schema objects (v1):**
- `financial_categories`
- `budget_versions`
- `budget_lines`
- `budget_assumptions` (lightweight notes/drivers)
- `actuals_monthly`
- `account_mapping`
- `variance_monthly_v1` (view/materialized view)

**Core constraints/indexes:**
- unique `budget_lines(version_id, month_start, category_id)`
- unique source-idempotency key for actual rows
- `idx_budget_lines_version_month`
- `idx_actuals_monthly_month_category`
- `idx_account_mapping_source_account`

### **✅ Validation Checklist**
- [ ] Migrations implemented cleanly (`0080`-`0083`).
- [ ] Versioned lines enforce uniqueness.
- [ ] Variance view returns deterministic outputs with tested month-boundary behavior.
- [ ] Index paths align to endpoint access patterns.

---

## 3️⃣ **QUICKBOOKS INGESTION + MAPPING**
*Priority: High - make actuals reliable and audit-ready*

### **🎯 Objective**
Create resilient QuickBooks sync workflow with explicit mapping governance.

### **🔄 Implementation**

**QuickBooks inputs required from operations:**
- Production realm connection and app credentials.
- Accounting basis confirmation (`accrual` or `cash`).
- Fiscal-year boundaries and close process timing.
- Final account inclusion/exclusion and category mapping owner.
- Optional class/location usage decision for dimensional planning.

**Cadence and schedules:**
- Daily scheduled sync in early morning local time.
- Optional intraday refresh for in-month monitoring.
- Post-close reconciliation sync after adjustments.

**Known limitations to plan for:**
- API throttling or transient failures -> retry/backoff + run checkpointing required.
- Report totals can shift after reclassification/reconciliation.
- QBO bank balances are accounting-grade, not treasury real-time.
- Custom report layouts are not guaranteed API-equivalent; rely on internal mapping.

### **✅ Validation Checklist**
- [ ] Sync jobs are idempotent and restart-safe.
- [ ] Unmapped accounts are surfaced via run warnings.
- [ ] Run logs capture status/counts/errors in `quickbooks_sync_runs`.
- [ ] Source freshness metadata is exposed in sync result and API envelope meta.

---

## 4️⃣ **BACKEND FINANCE API CONTRACTS**
*Priority: High - deliver stable contracts for frontend and AI*

### **🎯 Objective**
Expose fast budget/actual/variance APIs using envelope conventions and strict contracts.

### **🔄 Implementation**

**Planned endpoint family (`/api/v1/finance/*`):**
- `GET /budget/versions`
- `POST /budget/versions`
- `POST /budget/versions/{version_id}/lock`
- `GET /budget/lines`
- `POST /budget/lines/bulk`
- `GET /budget/variance`
- `GET /budget/variance/summary`
- `POST /budget/actuals/sync/run` (token/role gated)

### **✅ Validation Checklist**
- [ ] Envelope contract parity implemented across finance endpoints.
- [ ] Query filters use `snake_case`.
- [ ] JSON fields use `camelCase`.
- [ ] Endpoint behavior validated through focused API/service tests and full suite pass.

---

## 5️⃣ **FRONTEND BUDGET AND VARIANCE WORKFLOW**
*Priority: High - make planning practical and fast to use*

### **🎯 Objective**
Ship a focused finance workflow for budget setup, versioning, and variance review.

### **🔄 Implementation**
- Budget version manager (create, clone, lock).
- Monthly budget grid by category.
- Variance dashboard (top over/under + month trend).
- Reforecast flow from approved baseline.

### **✅ Validation Checklist**
- [ ] Page is server-first data loaded.
- [ ] No unnecessary `useEffect` sync logic.
- [ ] Loading/error/empty states complete.
- [ ] UI uses existing design primitives and Tailwind patterns.

---

## 6️⃣ **AI + PLATFORM BRIDGE**
*Priority: Medium - connect finance to the broader product model*

### **🎯 Objective**
Enable AI and command-center to reason about budget performance and action priorities.

### **🔄 Implementation**
- Add `ai_budget_context_v1`, `ai_budget_changes_v1`, `ai_budget_alerts_v1`.
- Surface top budget risk cards in command-center.
- Keep canonical category keys aligned with travel/AP/debt/FX dimensions for future unified analysis.

### **✅ Validation Checklist**
- [ ] AI context views are compact and deterministic.
- [ ] Recommendation evidence references source periods/categories.
- [ ] No contradictory metric labels across modules.

---

## 7️⃣ **QA, RUNBOOK, AND DOCUMENTATION CLOSEOUT**
*Priority: High - ensure stable operation after launch*

### **🎯 Objective**
Validate correctness, performance, and operating readiness.

### **🧪 Testing**
- Unit tests: mapping logic, variance math, version lifecycle.
- Integration tests: sync pipeline and finance APIs.
- Frontend checks: lint/type/build and manual workflow QA.
- Data QA: fixture comparisons using 2025 P&L categories.

### **📚 Documentation Updates**

| Document | Section | Change Description |
|----------|---------|-------------------|
| `docs/swainos-code-documentation-backend.md` | Finance domain | Schema, ingestion, and endpoint behavior |
| `docs/swainos-code-documentation-frontend.md` | Finance module | Route composition and UX behaviors |
| `docs/frontend-data-queries.md` | Finance query inventory | Endpoint list and filter semantics |
| `docs/sample-payloads.md` | Finance payloads | Canonical request/response envelopes |
| `action-plan/action-log` | Milestones | Timestamped rollout updates |

### **✅ Validation Checklist**
- [ ] Backend compile/tests pass for touched modules.
- [ ] Frontend lint/type/build pass for touched modules.
- [ ] Runbook covers daily sync and month-end close.
- [ ] Docs are aligned to shipped contracts for implemented backend scope.

---

## ⚠️ **RISK MANAGEMENT**

### **High Priority Risks**
- **Category mapping drift**: new QBO accounts create silent misclassification -> **Mitigation**: mapping exception queue + mandatory review.
- **Scope bloat**: adding advanced forecasting too early -> **Mitigation**: strict v1 non-goals and phased rollout.
- **Cross-module label drift**: inconsistent metric names across finance/cash-flow/AI -> **Mitigation**: glossary and contract governance checks.

### **Medium Priority Risks**
- **Data freshness confusion**: users act on stale actuals -> **Mitigation**: expose `asOfDate` and source freshness status prominently.
- **API throughput spikes**: sync and dashboard load collide -> **Mitigation**: schedule sync windows and precomputed variance views.

### **Rollback Strategy**
1. Freeze budget writes by status gate while keeping reads active.
2. Revert active planning version to latest locked baseline.
3. Rebuild variance view from verified source snapshots and rerun validation checks.

---

## 📊 **SUCCESS CRITERIA**

### **Technical Success Metrics**

| Metric | Target | Verification Method |
|--------|--------|---------------------|
| Version lifecycle integrity | 100% lock behavior | API integration tests |
| Sync reliability | Daily successful runs | Sync logs and alerts |
| Variance correctness | Deterministic fixture parity | Unit + integration tests |
| Mapping completeness | 100% material account mapping | Mapping QA report |
| Performance | No noticeable UI lag | Route timing and query profiling |

### **User Experience Success**

| Scenario | Expected Outcome |
|----------|------------------|
| Finance creates baseline version | Budget lines save and version locks correctly |
| Daily sync runs | Actuals refresh with visible as-of timestamp |
| Exec reviews variance | Top misses and trends are immediately clear |
| Team runs reforecast | New version created without overwriting history |

---

## 🔗 **RELATED DOCUMENTATION**

- **[Action Plan Template](./action-plan-template.md)** - Required planning structure
- **[FX Core Currency Plan](./13-fx-core-currency-buy-timing-framework-plan-completed.md)** - Relevant finance-risk architecture patterns
- **[Debt Service Plan](./14-debt-service-foundation-and-scenario-planning-plan-completed.md)** - Debt domain integration reference
- **[Backend Code Documentation](../docs/swainos-code-documentation-backend.md)** - Backend architecture and active endpoints
- **[Frontend Code Documentation](../docs/swainos-code-documentation-frontend.md)** - Frontend architecture and routes

---

## 📝 **REVISION HISTORY**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.0 | 2026-03-06 | AI Agent + Ian | Initial execution-ready plan for budget + QuickBooks integration using action-plan template |
| v1.1 | 2026-03-06 | AI Agent + Ian | Updated to current implementation state: backend/supabase complete, QBO runtime hardened, tests/docs/audits complete, frontend deferred |
| v1.2 | 2026-03-06 | AI Agent + Ian | Feature parked and reset to not-started state; execution checklists/statuses reset for future restart |
