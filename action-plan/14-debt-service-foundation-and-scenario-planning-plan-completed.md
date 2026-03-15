# 🎯 Debt Service Foundation and Scenario Planning - Citizens Term Sheet to Supabase Operating Model

> **Version**: v1.2  
> **Status**: ✅ COMPLETED  
> **Date**: 2026-02-27  
> **Completion Date**: 2026-02-27

**Target Components**: `SwainOS_BackEnd/supabase/migrations/`, `SwainOS_BackEnd/src/api/`, `SwainOS_BackEnd/src/services/`, `SwainOS_BackEnd/src/repositories/`, `SwainOS_BackEnd/src/schemas/`, `SwainOS_FrontEnd/apps/web/src/features/debt-service/`, `SwainOS_FrontEnd/apps/web/src/lib/api/`, `SwainOS_FrontEnd/apps/web/src/lib/types/`, `SwianOS_Documentation/docs/`  
**Primary Issues**: Debt Service currently has placeholder UI and starter schema only; there is no production debt domain for schedule generation, payment logging, DSCR monitoring, covenant tracking, or early-payoff simulation.  
**Objective**: Build the canonical debt-servicing data model and API/UI foundation so SwainOS can track scheduled and actual principal/interest payments, auto-update balances in Supabase, and evaluate payoff/prepayment options against broader cash-flow constraints.

## 📋 **QUICK SUMMARY**

**What We're Building/Fixing**: A full debt-service operating layer that starts with the Citizens term-sheet contract and extends to real payment logging, amortization tracking, covenant monitoring, and scenario analysis.

**Critical Issues Being Addressed**:
- Debt data is not operationalized -> define canonical debt facility, schedule, actual payment, covenant, and scenario tables.
- Debt tab is UI-only placeholder -> ship backend contracts and debt-service visuals tied to live Supabase data.
- No payoff decision support -> add deterministic scenario engine for extra principal and payoff timing options.

**Term-Sheet Inputs Locked for Phase 1**:
- **SBA 7A Guaranteed Loan**: `loanAmount = 4,300,000`
- **Structure**: `10-year`, `fully amortizing`
- **Rate**: `Prime + 0.25%`, term sheet states `7.00% fully fixed`
- **Repayment**: equal monthly principal + interest (interest declines as principal pays down)
- **Prepayment**: no prepayment penalty

**Success Metrics**:
- Debt facility + amortization schedule can be generated from fixed-rate loan inputs beginning May 1 baseline planning window.
- Logged payments automatically update outstanding balance, principal paid, and interest paid.
- Debt Service tab renders live KPI cards, schedule table, and scenario comparison visuals.
- Scenario runs produce payoff-date and interest-savings deltas versus base plan.
- API contracts and docs are aligned to envelope standards and naming conventions.

---

## 🎯 **EXECUTION STATUS**

**Progress**: 6 of 6 sections completed  
**Current Status**: Execution complete. Debt schema, APIs, UX, integration hooks, and documentation are aligned to current implementation.

| Phase | Status | Priority | Notes |
|-------|---------|----------|-------|
| 1️⃣ Debt Contract and Scope Freeze | ✅ COMPLETED | HIGH | Formulas, debt semantics, and baseline date rules locked and implemented |
| 2️⃣ Supabase Debt Domain Buildout | ✅ COMPLETED | HIGH | Canonical debt tables, constraints, RLS, and refinement migrations in place |
| 3️⃣ Debt Engine + Backend API Surface | ✅ COMPLETED | HIGH | Schedule generation, payment logging, scenarios, and covenant endpoints live |
| 4️⃣ Debt Service Visuals and UX | ✅ COMPLETED | HIGH | Operational dashboard shipped with creditor-level visibility and manual payment prompt |
| 5️⃣ Cross-Module Decision Integration | ✅ COMPLETED | MEDIUM | Debt obligations surfaced in command-center reads and decision context |
| 6️⃣ QA, Validation, and Documentation Closeout | ✅ COMPLETED | HIGH | Lint/contract checks and documentation synchronization completed |

---

## 🚨 **CRITICAL REQUIREMENTS**

### **⚠️ MANDATORY CHECKLIST - Must Complete Before Implementation**

- [x] **Canonical Debt Formulas**: Principal/interest allocation, balance rollforward, and payoff deltas are deterministic and documented.
- [x] **Schema Integrity**: Debt facility, schedule, actual payments, and scenarios are separated cleanly (no mixed-purpose rows).
- [x] **Auto-Adjustment Logic**: Logged payments update balance and downstream metrics immediately in database-backed outputs.
- [x] **Envelope Consistency**: All debt endpoints return `{ data, pagination, meta }`.
- [x] **Naming Compliance**: DB `snake_case`, API params `snake_case`, JSON `camelCase`, frontend files in `kebab-case.tsx`.
- [x] **No Dead Code**: Remove placeholder-only debt logic once live debt data flow is active.
- [x] **Docs Sync**: Backend/frontend docs and debt contract references are updated with final schema and endpoint behavior.

### **Documentation Update Requirement**

> **Required docs for this rollout**:
> - `docs/swainos-code-documentation-backend.md`
> - `docs/swainos-code-documentation-frontend.md`
> - `docs/frontend-data-queries.md`
> - `docs/sample-payloads.md`
> - `docs/swainos-terminology-glossary.md` (if new debt terms are introduced)

---

## 📐 **NAMING CONVENTION ALIGNMENT**

This plan follows existing SwainOS conventions:
- Backend modules/files: `snake_case.py`
- Frontend component files: `kebab-case.tsx`
- Frontend service files: `camelCaseService.ts`
- Supabase objects: `snake_case` with index naming `idx_*`
- API query params: `snake_case`
- API JSON fields: `camelCase`

Debt-specific naming lock:
- Display labels: `Outstanding Balance`, `Scheduled Payment`, `Principal Paid`, `Interest Paid`, `Extra Principal`, `Remaining Balance`, `Payoff Date`
- Contract keys: `outstandingBalanceAmount`, `scheduledPaymentAmount`, `principalPaidAmount`, `interestPaidAmount`, `extraPrincipalAmount`, `remainingBalanceAmount`

---

## 🔧 **STRATEGIC APPROACH**

### **Implementation Philosophy**
Treat debt servicing as a first-class financial ledger domain. Build a deterministic debt core first (facility terms, schedule, actual payments, rollforward), then layer scenario analysis and visuals, then connect debt decisions to cash-flow and AI recommendation systems.

### **Key Architecture Decisions**
- **Facility-first model**: each debt instrument is modeled explicitly with immutable origination terms and controlled term revisions.
- **Schedule vs actual separation**: projected schedule rows remain distinct from actual posted payments to preserve auditability.
- **Scenario isolation**: simulated payoff plans do not overwrite base debt records; scenario outputs are versioned and comparable.
- **Deterministic before AI**: payoff recommendations are computed from rules/math first; AI narratives only summarize and prioritize.

### **Data Flow**

```
Citizens term-sheet terms + manual debt setup
  -> debt_facilities + debt_terms_history
  -> amortization engine (fixed-rate monthly PI schedule)
  -> debt_payment_schedule (projected monthly rows)
  -> payment logging (actual principal/interest/extra principal)
  -> debt_balance_snapshots + covenant snapshots + DSCR metrics
  -> debt scenarios (base vs accelerated payoff options)
  -> API endpoints (/api/v1/debt-service/*)
  -> Debt Service UI visuals (KPIs, schedule, trend, scenario comparisons)
```

---

## 1️⃣ **DEBT CONTRACT AND SCOPE FREEZE**
*Priority: High - lock debt semantics before schema and API build*

### **🎯 Objective**
Freeze v1 debt scope, formulas, and date semantics for reproducible schedule generation and payment tracking.

### **🔍 Analysis / Discovery**
- Lock start operating assumptions: planning period begins from `May 1` baseline window for dashboarding and forward schedule views.
- Confirm fixed-rate amortization behavior for primary term loan:
  - equal monthly payment amount
  - declining interest component
  - increasing principal component over time
- Lock treatment of extra principal:
  - reduces future interest
  - shortens payoff date unless explicit recast option selected
- Confirm prepayment penalty rule: none for the primary SBA 7A term loan.

### **⚙️ Implementation**

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `docs/swainos-code-documentation-backend.md` | Modify | Add debt domain formulas and date semantics |
| `docs/swainos-code-documentation-frontend.md` | Modify | Add debt KPI/visual interpretation contract |
| `docs/sample-payloads.md` | Modify | Add debt-service endpoint samples |

**Implementation Steps:**
1. Freeze debt formula dictionary (scheduled payment split, rollforward, scenario deltas).
2. Freeze data semantics (`asOfDate`, schedule period basis, posted payment rules).
3. Freeze v1 product scope to primary fixed-rate term loan plus payment logging and simulation.

### **✅ Validation Checklist**
- [x] Debt formulas are documented and unambiguous.
- [x] Date semantics are explicit and shared across backend/frontend.
- [x] Scope boundaries are confirmed (v1 deterministic debt core).

---

## 2️⃣ **SUPABASE DEBT DOMAIN BUILDOUT**
*Priority: High - implement canonical debt data architecture*

### **🎯 Objective**
Create normalized Supabase schema that supports debt terms, generated schedules, posted payments, covenants, and scenarios.

### **🔄 Implementation**

**Schema Objects (v1):**
- `debt_facilities`
  - one row per debt instrument (loan metadata, lender, type, origination, maturity, status)
- `debt_facility_terms`
  - effective-dated term parameters (`rate_mode`, `annual_rate`, `payment_frequency`, `amortization_months`, `is_fixed`)
- `debt_payment_schedule`
  - projected per-period due rows (`due_date`, `scheduled_payment_amount`, `scheduled_principal_amount`, `scheduled_interest_amount`, `remaining_balance_amount`)
- `debt_payments_actual`
  - posted payment events (`payment_date`, `principal_paid_amount`, `interest_paid_amount`, `extra_principal_amount`, `source_account`, `reference`)
- `debt_balance_snapshots`
  - as-of balances for fast KPI reads and audit trails
- `debt_covenants` and `debt_covenant_snapshots`
  - threshold definitions and measured pass/fail outputs
- `debt_scenarios` and `debt_scenario_events`
  - scenario header + simulated extra-principal event timeline

**Indexes and constraints:**
- Unique schedule row per facility per due date
- FK integrity from schedule/payments/scenarios to facilities
- Numeric non-negative checks for principal, interest, and balances
- `idx_debt_payment_schedule_facility_due_date`
- `idx_debt_payments_actual_facility_payment_date`
- `idx_debt_balance_snapshots_facility_as_of_date`

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `SwainOS_BackEnd/supabase/migrations/00xx_create_debt_service_domain_v1.sql` | Create | Canonical debt tables/constraints/indexes |
| `SwainOS_BackEnd/supabase/migrations/00xx_create_debt_service_views_v1.sql` | Create | Debt KPI and schedule summary views |
| `SwainOS_BackEnd/supabase/migrations/00xx_debt_rls_policies_v1.sql` | Create/Modify | Debt domain RLS controls |

### **✅ Validation Checklist**
- [x] Migrations apply cleanly and are idempotent-safe where practical.
- [x] Fixed-rate schedule rows generate correctly for test facility.
- [x] Payment-posting writes preserve FK and numeric constraints.
- [x] RLS behavior is aligned with admin/service and authenticated read patterns.

---

## 3️⃣ **DEBT ENGINE + BACKEND API SURFACE**
*Priority: High - operationalize schedule generation and payment logging*

### **🎯 Objective**
Provide deterministic backend logic and APIs for debt overview, schedules, payment posting, DSCR/covenants, and scenario comparisons.

### **⚙️ Implementation**

**Backend capabilities:**
- Fixed-rate amortization generator for equal monthly PI schedule
- Payment posting workflow:
  - log principal + interest + optional extra principal
  - recompute remaining balance and status
  - preserve immutable event history
- Scenario runner:
  - apply extra-principal events
  - return payoff date delta and total-interest delta vs base
- DSCR and covenant evaluation snapshots

**Endpoint family (`/api/v1/debt-service/*`):**
- `GET /overview`
- `GET /facilities`
- `GET /schedule`
- `GET /payments`
- `POST /payments`
- `GET /covenants`
- `GET /scenarios`
- `POST /scenarios/run`

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `src/schemas/debt_service.py` | Create | Request/response contracts |
| `src/repositories/debt_service_repository.py` | Create | Debt domain DB access |
| `src/services/debt_service_service.py` | Create | Amortization, posting, scenario logic |
| `src/api/debt_service.py` | Create | Debt API route handlers |
| `src/api/router.py` | Modify | Register debt-service routes |

### **✅ Validation Checklist**
- [x] Payment posting updates debt outputs deterministically.
- [x] Schedule endpoint returns declining interest profile correctly.
- [x] Scenario endpoint returns stable base-vs-scenario deltas.
- [x] API envelopes and error contracts match platform standards.

---

## 4️⃣ **DEBT SERVICE VISUALS AND UX**
*Priority: High - replace placeholder debt tab with executive-operational views*

### **🎯 Objective**
Ship Debt Service visuals that make payment status, servicing trajectory, and payoff options immediately understandable.

### **🔄 Implementation**

**Debt tab visual modules (v1):**
- KPI strip:
  - Outstanding Balance
  - Next Due Payment
  - Principal Paid YTD
  - Interest Paid YTD
  - DSCR and Covenant status
- Amortization timeline chart:
  - stacked monthly principal vs interest bars
  - remaining balance trend line
- Payment ledger table:
  - posted payment date, principal, interest, extra principal, running balance
- Scenario comparison panel:
  - Base vs selected scenario
  - payoff date difference
  - total interest saved
  - monthly cash impact
- Risk/watchlist panel:
  - upcoming due spikes, covenant headroom warnings, missed-payment flags

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `apps/web/src/lib/types/debt-service.ts` | Create | Debt contract types |
| `apps/web/src/lib/api/debtService.ts` | Create | Debt API client |
| `apps/web/src/features/debt-service/debt-service-server-loader.ts` | Create | Server-first data loader |
| `apps/web/src/features/debt-service/debt-service-page.tsx` | Modify | Replace placeholder with live visuals |
| `apps/web/src/features/debt-service/*` | Create | Chart/table/scenario subcomponents |

### **✅ Validation Checklist**
- [x] Debt tab renders live data without placeholder blocks.
- [x] Visuals clearly show declining interest and principal shift over time.
- [x] Scenario compare UX is deterministic and readable.
- [x] Loading/empty/error states are complete and polished.

---

## 5️⃣ **CROSS-MODULE DECISION INTEGRATION**
*Priority: Medium - tie debt decisions into cash and AI operating context*

### **🎯 Objective**
Integrate debt-service outputs with cash-flow and AI context so payoff decisions are evaluated against liquidity and operating risk.

### **🔄 Implementation**
- Feed debt service due amounts into cash-flow forecasts with shared date semantics.
- Add debt metrics to command-center and AI context surfaces:
  - next 30/60/90 day debt obligations
  - covenant proximity indicators
  - scenario impact on projected cash runway
- Define deterministic recommendation gate:
  - do not suggest extra principal if liquidity guardrail fails.

### **✅ Validation Checklist**
- [x] Cash-flow views consume updated debt obligations accurately.
- [x] AI/debt narratives align with deterministic debt math.
- [x] No contradictory debt values across modules.

---

## 6️⃣ **QA, VALIDATION, AND DOCUMENTATION CLOSEOUT**
*Priority: High - ensure debt-service reliability before broader rollout*

### **🎯 Objective**
Validate debt-domain correctness, protect contract quality, and close documentation/action-plan updates.

### **🧪 Testing**
- Backend:
  - unit tests for amortization math and payment-posting rollforward
  - integration tests for debt-service endpoints and scenario runs
- Frontend:
  - lint/type checks
  - visual QA for KPI, chart, ledger, and scenario panels
- Data:
  - seeded fixture test for known loan values and month-by-month expected outputs
  - edge-case validation (extra principal, early payoff, zero extra principal)

### **📚 Documentation Updates**

| Document | Section | Change Description |
|----------|---------|-------------------|
| `docs/swainos-code-documentation-backend.md` | Debt Service domain | Debt schema, formulas, endpoints, and services |
| `docs/swainos-code-documentation-frontend.md` | Debt Service module | Visual architecture and data-loading patterns |
| `docs/frontend-data-queries.md` | Debt query inventory | Debt-service endpoint map for route loaders/services |
| `docs/sample-payloads.md` | Debt endpoint samples | Canonical request/response examples |
| `action-plan/action-log` | Milestones | Timestamped debt-service rollout entries |

### **✅ Validation Checklist**
- [x] Backend tests and compile checks pass on touched debt files.
- [x] Frontend lint/type checks pass on touched debt files.
- [x] Docs accurately reflect implemented contracts and visuals.
- [x] Action plan status progresses to ✅ COMPLETED only after full validation.

---

## ⚠️ **RISK MANAGEMENT**

### **High Priority Risks**
- **Term mismatch risk**: final loan docs may differ from term sheet -> **Mitigation**: store versioned debt terms and effective dates.
- **Math integrity risk**: amortization/posting bugs could misstate balances -> **Mitigation**: deterministic unit tests with fixed expected schedules.
- **Cross-module drift risk**: debt values differ from cash-flow module -> **Mitigation**: shared rollup/view contracts and parity checks.

### **Medium Priority Risks**
- **Scenario misuse risk**: aggressive prepayment suggestions without liquidity context -> **Mitigation**: liquidity guardrails and decision warnings.
- **Data freshness risk**: manual payment entry delay causes stale debt KPIs -> **Mitigation**: explicit `asOfDate` and overdue-entry prompts.

### **Rollback Strategy**
1. Disable debt-service write routes while preserving read endpoints.
2. Revert debt-service UI to read-only schedule mode.
3. Re-run schedule generation from facility terms and reconcile payment events.

---

## 📊 **SUCCESS CRITERIA**

### **Technical Success Metrics**

| Metric | Target | Verification Method |
|--------|--------|---------------------|
| Debt schedule correctness | 100% parity with deterministic amortization fixture | Unit/integration tests |
| Payment logging integrity | Posted payment updates reflected in next debt reads | API tests |
| Scenario correctness | Payoff delta and interest-savings outputs stable | Scenario test fixtures |
| Contract quality | All debt endpoints envelope-compliant | API contract tests |
| Frontend quality | Debt module lint/type clean | `npm run lint` + type checks |

### **User Experience Success**

| Scenario | Expected Outcome |
|----------|------------------|
| Open Debt Service tab | Live KPI, schedule, and ledger visuals render |
| Log principal and interest payment | Balance and next-payment values update automatically |
| Run extra-principal scenario | User sees payoff-date and interest-savings comparison |
| Review risk panel | User sees clear covenant and payment-timing warnings |

---

## 🔗 **RELATED DOCUMENTATION**

- `./action-plan-template.md` - Planning structure baseline
- `./04-data-import-plan-holding.md` - Financial data dependency tracking
- `../docs/SwainOS_Project_Specification.pdf` - Debt-service goals and phase references
- `../docs/swainos-code-documentation-backend.md` - Backend architecture and endpoint catalog
- `../docs/swainos-code-documentation-frontend.md` - Frontend modules and route coverage

---

## 📝 **REVISION HISTORY**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.2 | 2026-02-27 | AI Agent + Ian | Final quality-audit pass: confirmed completion state, tightened runtime resilience notes, and closed UI technical-debt risk by scoping compact KPI typography to Debt Service only |
| v1.1 | 2026-02-27 | AI Agent + Ian | Marked execution complete, closed all validation checklists, and synced plan status to implemented debt-service architecture/UI |
| v1.0 | 2026-02-27 | AI Agent + Ian | Initial debt-service foundation and scenario-planning action plan based on Citizens signed term sheet and SwainOS module architecture |
