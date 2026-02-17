# 🎯 Travel Agency and Travel Agent Analytics Plan - Backend-First Full Product Rollout

> **Version**: v1.0  
> **Status**: ✅ COMPLETED  
> **Date**: 2026-02-17  
> **Completion Date**: 2026-02-17

**Target Components**: `SwainOS_BackEnd/src/api/**`, `SwainOS_BackEnd/src/services/**`, `SwainOS_BackEnd/src/repositories/**`, `SwainOS_BackEnd/src/schemas/**`, `SwianOS_Documentation/supabase/migrations/**`, `SwainOS_FrontEnd/apps/web/src/features/sales/**`, `SwainOS_FrontEnd/apps/web/src/features/travel-consultant/**`, `SwainOS_FrontEnd/apps/web/src/lib/api/**`, `SwainOS_FrontEnd/apps/web/src/lib/types/**`, `SwianOS_Documentation/docs/**`  
**Primary Issues**: SwainOS has strong travel consultant analytics but does not yet provide first-class Travel Agency and Travel Agent analytics, search, profile drill-down, and relationship insight parity.  
**Objective**: Deliver complete Travel Agency + Travel Agent analytics with backend-first execution (Supabase rollups, API surface, services, schemas), followed by full frontend rollout with consultant-parity UX and canonical naming, with no unresolved data-contract unknowns at build start.

## 📋 **QUICK SUMMARY**

**What We're Building/Fixing**: A full Travel Agencies sales surface where users can search, rank, and drill into both agencies and agents, including production metrics, consultant affinity, and year-over-year performance.

**Critical Issues Being Addressed**:
- No first-class Travel Agencies/Agents analytics model -> introduce canonical backend entities and relationship rollups.
- No drill-down profiles for agency/agent production quality -> ship consultant-parity API/profile contracts.
- No top-N ranking and fuzzy unified search -> provide optimized leaderboard/search endpoints with one search input contract.
- Trade attribution is implicit and inconsistent -> formalize deterministic trade classification and eligibility rules in rollups.

**Success Metrics**:
- Backend supports `leaderboard`, `profile`, and `search` for both Travel Agents and Travel Agencies with stable IDs.
- Top-5 and Top-10 rank queries return within existing consultant endpoint performance envelope.
- YoY metrics delivered for leads, traveled itineraries, gross revenue, and gross profit (no margin/pax expansion in v1).
- Frontend Sales tab includes a new `Travel Agencies` module with searchable leaderboards and drill-down pages for both entity types.

---

## 🎯 **EXECUTION STATUS**

**Progress**: 6 of 6 sections completed  
**Current Status**: Backend and frontend rollout completed; migrations, APIs, UI modules, and documentation aligned to current contracts.

| Phase | Status | Priority | Notes |
|-------|---------|----------|-------|
| 1️⃣ Data Contract and Trade Classification Lock | ✅ COMPLETE | HIGH | Trade classification and metric naming locked to canonical glossary and contracts |
| 2️⃣ Supabase Schema and Rollup Foundation | ✅ COMPLETE | HIGH | Canonical travel-agent/agency schema, split lead/booked rollups, and deterministic refresh function implemented |
| 3️⃣ Backend API Surface and Service Layer | ✅ COMPLETE | HIGH | Travel-agent/agency/search endpoints and service/repository layers implemented with consultant-parity envelopes |
| 4️⃣ Frontend Travel Agencies Module Buildout | ✅ COMPLETE | HIGH | Travel Agencies landing + drill-down pages implemented with search, top-N ranking, and responsive tables |
| 5️⃣ QA, Performance Hardening, and Regression Gate | ✅ COMPLETE | HIGH | Migration/runtime fixes, profile data-path correction, and responsive UI pass completed |
| 6️⃣ Documentation and Closeout | ✅ COMPLETE | HIGH | Backend/frontend docs, frontend queries, sample payloads, glossary, and action log updated |

---

## 🚨 **CRITICAL REQUIREMENTS**

### **⚠️ MANDATORY CHECKLIST - Must Complete Before Implementation**

- [x] **Deterministic Trade Rule**: Itinerary is trade only when `consortia` is present/non-blank and not a not-applicable/null-like marker, and a primary contact linkage exists.
- [x] **Entity Stability**: Stable external IDs are mandatory for agency and agent contracts; unknown mappings blocked from analytics output.
- [x] **Future Mobility Support**: Agent-to-agency assignment is single-active in v1 but modeled with effective dating for future moves.
- [x] **Deterministic Ranking**: Tie-breakers for top lists and primary-consultant affinity are fixed and documented.
- [x] **Canonical Naming**: User-facing metric names follow glossary (`Gross Profit`, `Gross Revenue`, `Conversion Rate`, `Close Rate`, `Booked Itineraries`).
- [x] **No Dirty Analytics Buckets**: Missing/dirty contact-account link records are excluded from analytics datasets.
- [x] **Contract Consistency**: Endpoint shape and envelopes mirror Travel Consultant contract conventions.
- [x] **No Feature Flags**: Ship full backend, then full frontend rollout directly.

### **Documentation Update Requirement**

> **Required docs for this rollout**:
> - `docs/swainos-code-documentation-backend.md`
> - `docs/swainos-code-documentation-frontend.md`
> - `docs/frontend-data-queries.md`
> - `docs/sample-payloads.md`
> - `docs/swainos-terminology-glossary.md` (if any new cross-module term is introduced)

---

## 📐 **NAMING CONVENTION ALIGNMENT**

This rollout follows existing project naming conventions and glossary standards:
- Backend modules/files: `snake_case.py`
- Frontend components/files: `kebab-case.tsx`
- Frontend services: `camelCaseService.ts`
- API query params: `snake_case`
- JSON payload fields: `camelCase`
- Canonical financial metric key: `grossProfitAmount` with display label **Gross Profit**

---

## 🔧 **STRATEGIC APPROACH**

### **Implementation Philosophy**
Build from deterministic attribution and pre-aggregated analytics first, then expose stable API contracts, then deliver frontend parity with Travel Consultant experiences. Avoid bespoke paths by reusing existing service/repository and UI module patterns wherever possible.

### **Key Architecture Decisions**
- **Two first-class entities**: `travelAgency` (organization) and `travelAgent` (person/primary contact), each with stable ID and profile endpoint.
- **Single-active + effective dating**: v1 behavior is one agent tied to one agency; schema tracks `effective_from`/`effective_to` so agent moves are supported without redesign.
- **Pre-aggregated rollups for speed**: monthly and yearly rollups for agency and agent drive leaderboard/profile reads, minimizing runtime joins.
- **Physical rollup tables + refresh function**: use explicit rollup tables (not only materialized views) for predictable indexes, simpler refresh sequencing, and safer future incremental refresh.
- **Unified fuzzy search contract**: one backend search endpoint accepts one query string and searches agent name, agency name, email, and supported identifiers.
- **Strict exclusion policy**: records with unresolved contact-account linkage are excluded, not bucketed.
- **Deterministic tie-break hierarchy**:
  - Top lists: `gross_profit_amount DESC`, then `traveled_itineraries DESC`, then `gross_amount DESC`, then `entity_name ASC`.
  - Primary consultant affinity: `converted_leads DESC`, then `closed_won_itineraries DESC`, then `consultant_name ASC`.

### **Data Flow**

```
┌─────────────────────────────────────────────────────────────┐
│ Sales UI: Travel Agencies module                            │
│   ├── Travel Agent leaderboard/profile pages                │
│   └── Travel Agency leaderboard/profile pages               │
├─────────────────────────────────────────────────────────────┤
│ Frontend service layer (typed)                              │
│   └── GET /travel-agents/* and /travel-agencies/*           │
├─────────────────────────────────────────────────────────────┤
│ FastAPI routes -> services -> repositories                  │
│   └── Consultant-parity envelopes and snake_case params     │
├─────────────────────────────────────────────────────────────┤
│ Supabase rollups + entity mapping                           │
│   ├── Agent/Agency identity + relationship table            │
│   ├── Monthly/yearly production + YoY rollups               │
│   └── Lead conversion and consultant affinity rollups       │
└─────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ **DATA CONTRACT AND TRADE CLASSIFICATION LOCK**
*Priority: High - Freeze deterministic business rules before schema/migration work*

### **🎯 Objective**
Finalize immutable v1 classification, eligibility, and metric definitions so backend and frontend implementation remains aligned.

### **🔍 Analysis / Discovery**
- Confirmed rules from stakeholder direction:
  - Trade classification has no exceptions.
  - Default top-list ranking metric is **Gross Profit**.
  - Default period is **current year**.
  - YoY scope: **leads**, **traveled itineraries**, **gross revenue**, **gross profit**.
  - "Primary travel consultant they use" is based on **leads converted**.
- Canonical ID resolution lock (no unknowns at implementation start):
  - `travelAgent.externalId` <- contact external ID from canonical contacts source.
  - `travelAgency.externalId` <- agency/account external ID from canonical agencies source.
  - Itinerary attribution requires both resolvable `primary_contact_external_id` and `agency_external_id`.
  - Rows failing this mapping are excluded from analytics rollups by design.

### **⚙️ Implementation**

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `docs/sample-payloads.md` | Modify | Add canonical request/response examples for new endpoints |
| `docs/frontend-data-queries.md` | Modify | Add frontend query map for agent/agency pages |
| `docs/swainos-code-documentation-backend.md` | Modify | Add new endpoint family and rollup references |

**Implementation Steps:**
1. Define exact trade eligibility predicate in documentation and migration comments.
2. Define canonical metric formulas and period behavior (current year default, YoY comparables).
3. Define consultant affinity formula: highest converted leads with deterministic tie-breakers.
4. Freeze endpoint/query conventions to mirror Travel Consultant APIs.
5. Freeze ranking tie-breakers for all leaderboard and profile "top" sections.

### **✅ Validation Checklist**
- [ ] Trade classification rule is documented exactly once and referenced consistently.
- [ ] KPI definitions are stable across payload docs and backend docs.
- [ ] Query param semantics match existing consultant patterns.

---

## 2️⃣ **SUPABASE SCHEMA AND ROLLUP FOUNDATION**
*Priority: High - Build normalized identity + optimized analytics sources*

### **🎯 Objective**
Create schema and rollups that support fast leaderboard/profile/search reads for both entities without runtime heavy joins.

### **🔄 Implementation**

**Supabase design (recommended for optimization):**
- Add canonical relationship and dimensions:
  - `travel_agencies` (stable agency identity keyed by external ID)
  - `travel_agents` (stable agent identity keyed by contact external ID)
  - `travel_agent_agency_assignments` (`agent_id`, `agency_id`, `effective_from`, `effective_to`, `is_primary`)
- Add rollup tables (indexed, refresh-owned):
  - `travel_agent_leaderboard_monthly_rollup`
  - `travel_agent_profile_monthly_rollup`
  - `travel_agency_leaderboard_monthly_rollup`
  - `travel_agency_profile_monthly_rollup`
  - `travel_agent_consultant_affinity_monthly_rollup` (converted-leads basis)
  - `travel_agency_top_agents_monthly_rollup`
- Add helper search surface:
  - `travel_trade_search_index` with normalized search text and weighted tokens for fuzzy matching.
- Add indexes:
  - compound indexes on `(year, month)`, `(entity_id, year, month)`, `(year, entity_rank)`.
  - search indexes: `GIN (to_tsvector('simple', search_text))` plus `pg_trgm` indexes for fuzzy fallback.
- Add refresh function:
  - `refresh_travel_trade_rollups_v1()` to refresh all new rollups in deterministic sequence.
- Add operational script:
  - `scripts/refresh_travel_trade_rollups.py` to execute and validate refresh in one command.

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `supabase/migrations/0048_create_travel_trade_agent_agency_rollups.sql` | Create | New entities, assignment table, rollups, indexes, refresh function |
| `scripts/refresh_travel_trade_rollups.py` | Create | Rollup refresh runner and basic execution logging |
| `docs/swainos-code-documentation-backend.md` | Modify | Record migration artifacts and rollup catalog |

### **✅ Validation Checklist**
- [ ] Rollups return expected values for current year and YoY windows.
- [ ] Agent move scenario works via effective dating without schema changes.
- [ ] Missing link records are excluded from rollups.
- [ ] Query plans use indexes for top-list and profile lookups.
- [ ] Search latency and relevance pass baseline checks on realistic query terms.

---

## 3️⃣ **BACKEND API SURFACE AND SERVICE LAYER**
*Priority: High - Expose stable, consultant-parity contracts*

### **🎯 Objective**
Ship complete backend APIs for search, leaderboard, and profile across agent and agency entities.

### **⚙️ Implementation**

**Endpoint family (consultant-parity envelope):**
- `GET /api/v1/travel-agents/leaderboard`
- `GET /api/v1/travel-agents/{agent_id}/profile`
- `GET /api/v1/travel-agencies/leaderboard`
- `GET /api/v1/travel-agencies/{agency_id}/profile`
- `GET /api/v1/travel-trade/search`

**Contract behavior:**
- Default sort metric: `gross_profit_amount` (frontend display: `Gross Profit`).
- Default period: current year.
- Supports top-N filtering (`top_n=5|10|...`) and pagination envelopes.
- Supports period selectors aligned to consultant contracts: `period_type`, `year`, `month`.
- Unified fuzzy search endpoint accepts one `query` and returns mixed typed results (`agent`, `agency`) with rank score.
- Profile responses include:
  - production snapshot,
  - YoY metrics (leads, traveled itineraries, gross revenue, gross profit),
  - primary consultant by converted leads,
  - agency profile includes top associated agents.

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `src/api/travel_agents.py` | Create | Travel Agent API routes |
| `src/api/travel_agencies.py` | Create | Travel Agency API routes |
| `src/api/travel_trade_search.py` | Create | Unified fuzzy search route |
| `src/services/travel_agents_service.py` | Create | Agent leaderboard/profile orchestration |
| `src/services/travel_agencies_service.py` | Create | Agency leaderboard/profile orchestration |
| `src/repositories/travel_agents_repository.py` | Create | Agent rollup access |
| `src/repositories/travel_agencies_repository.py` | Create | Agency rollup access |
| `src/repositories/travel_trade_search_repository.py` | Create | Search index access |
| `src/schemas/travel_agents.py` | Create | Agent response schemas |
| `src/schemas/travel_agencies.py` | Create | Agency response schemas |

### **✅ Validation Checklist**
- [ ] Route/service/repository layering follows backend standards.
- [ ] All query params are snake_case; all JSON fields are camelCase.
- [ ] Error envelope and pagination envelope match existing standards.
- [ ] Pydantic schemas are explicit and mypy-friendly.

---

## 4️⃣ **FRONTEND TRAVEL AGENCIES MODULE BUILDOUT**
*Priority: High - Deliver full UX parity after backend completion*

### **🎯 Objective**
Add a new `Travel Agencies` Sales tab module with searchable top lists and drill-down pages for both agents and agencies.

### **🔄 Implementation**

**UX scope:**
- New navigation/tab: `Travel Agencies`.
- One search bar for fuzzy lookup across all supported fields.
- Top lists:
  - top travel agents,
  - top travel agencies,
  - configurable top-N (5/10/etc).
- Drill-down pages:
  - Travel Agent profile page,
  - Travel Agency profile page with top associated agents.
- Metrics and labels aligned to Travel Consultant conventions and glossary terminology.

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `apps/web/src/lib/api/travelAgentsService.ts` | Create | Agent API client |
| `apps/web/src/lib/api/travelAgenciesService.ts` | Create | Agency API client |
| `apps/web/src/lib/api/travelTradeSearchService.ts` | Create | Unified search client |
| `apps/web/src/lib/types/travel-agents.ts` | Create | Agent typed contracts |
| `apps/web/src/lib/types/travel-agencies.ts` | Create | Agency typed contracts |
| `apps/web/src/features/sales/travel-agencies-page.tsx` | Create | Primary page with top lists and search |
| `apps/web/src/features/sales/travel-agent-profile-page.tsx` | Create | Agent deep-dive |
| `apps/web/src/features/sales/travel-agency-profile-page.tsx` | Create | Agency deep-dive with top agents |
| `apps/web/src/lib/constants/navigation.ts` | Modify | Add Travel Agencies navigation entry |

### **✅ Validation Checklist**
- [ ] Travel Agencies module renders from live backend data.
- [ ] Search returns mixed entity results and supports click-through drill-down.
- [ ] Top-N controls function correctly and default behavior is current year.
- [ ] UI labels use glossary canonical terms.

---

## 5️⃣ **QA, PERFORMANCE HARDENING, AND REGRESSION GATE**
*Priority: High - Ensure reliability, speed, and no contract regressions*

### **🎯 Objective**
Guarantee production-safe behavior with optimized query paths and deterministic results.

### **🧪 Testing**
- Backend:
  - repository/service tests for leaderboard/profile/search logic and edge filters.
  - validation tests for YoY math and consultant affinity ranking.
  - endpoint tests for envelope consistency and error handling.
- Supabase:
  - migration replay from clean state.
  - rollup refresh function validation.
  - explain plan checks for leaderboard/search/profile queries.
- Frontend:
  - manual navigation + search + drilldown flow.
  - loading, empty, and error states for all new pages.
  - cross-check values against sample payloads.
- Data correctness:
  - deterministic golden dataset test for ranking and tie-break behavior.
  - deterministic exclusion test for unresolved contact-account linkage rows.

### **✅ Validation Checklist**
- [ ] No major perf regression vs existing consultant analytics endpoints.
- [ ] All new endpoints pass envelope consistency checks.
- [ ] Lint/type checks pass cleanly in backend and frontend repos.

---

## 6️⃣ **DOCUMENTATION AND CLOSEOUT**
*Priority: High - Keep implementation and contracts auditable*

### **🎯 Objective**
Ship complete docs and plan updates in lockstep with delivered backend/frontend functionality.

### **📚 Documentation Updates**

| Document | Section | Change Description |
|----------|---------|-------------------|
| `docs/swainos-code-documentation-backend.md` | API Endpoints, Rollups, Key Modules | Add Travel Agent/Agency endpoint family and rollup surfaces |
| `docs/swainos-code-documentation-frontend.md` | Navigation Map, Module Coverage, Data and Services | Add Travel Agencies module and service/type references |
| `docs/frontend-data-queries.md` | Travel Agencies query inventory | Add endpoint/query matrix for list/profile/search |
| `docs/sample-payloads.md` | Travel Trade payload examples | Add canonical responses for leaderboard/profile/search |
| `action-plan/12-travel-agency-agent-analytics-backend-first-plan.md` | Status + revision history | Mark progress and final completion state |

### **✅ Validation Checklist**
- [ ] Documentation reflects final contracts and actual implemented behavior.
- [ ] Plan status transitions from READY -> IN PROGRESS -> COMPLETED.
- [ ] Action log updated for major milestones.

---

## ⚠️ **RISK MANAGEMENT**

### **High Priority Risks**
- **Identity mismatch risk**: inconsistent source IDs can corrupt attribution -> **Mitigation**: enforce canonical ID mapping and exclude unresolved links.
- **Rollup freshness risk**: stale aggregates can mis-rank top entities -> **Mitigation**: deterministic refresh function + operational script cadence.
- **Search relevance risk**: fuzzy search returns weak ranking quality -> **Mitigation**: weighted rank scoring (name exact > prefix > fuzzy) and result-type signals.
- **Refresh lock contention risk**: full refresh can block read-heavy periods -> **Mitigation**: refresh in deterministic order with transactional chunking and low-traffic scheduling.

### **Medium Priority Risks**
- **Agent mobility data drift**: reassignments can distort history if not date-bounded -> **Mitigation**: effective-dated assignment model from v1.
- **Frontend parity drift**: agency/agent pages diverge from consultant UX conventions -> **Mitigation**: reuse consultant page composition and shared primitives.

### **Rollback Strategy**
1. Revert API routes while preserving schema objects if frontend release has blockers.
2. Revert migration in controlled sequence if rollup correctness issues are found pre-release.
3. Re-run consultant baseline regression suite to verify no collateral impact.

---

## 📊 **SUCCESS CRITERIA**

### **Technical Success Metrics**

| Metric | Target | Verification Method |
|--------|--------|---------------------|
| Backend contract parity | 100% envelope consistency with consultant endpoints | API contract tests + payload diff review |
| Rollup query performance | Leaderboard/profile/search within existing consultant response envelope | Query timing + explain analysis |
| YoY metric correctness | 100% parity with source rollup formulas | Data validation scripts |
| Frontend module completeness | New Sales `Travel Agencies` module fully navigable | Manual QA checklist |
| Ranking determinism | 100% deterministic ordering for equal-metric entities | Golden dataset tie-break test |

### **User Experience Success**

| Scenario | Expected Outcome |
|----------|------------------|
| User opens Sales -> Travel Agencies | Sees top agencies and top agents, current-year by default |
| User searches one term in one search bar | Receives relevant mixed results (agent/agency) and can click through |
| User opens a Travel Agency profile | Sees production, YoY KPIs, primary consultant signal, and top connected agents |
| User opens a Travel Agent profile | Sees production and YoY metrics aligned with consultant-style narrative and labels |

---

## 🔗 **RELATED DOCUMENTATION**

- `./action-plan-template.md` - Standard planning structure
- `../docs/swainos-code-documentation-backend.md` - Backend architecture, endpoints, rollups
- `../docs/swainos-code-documentation-frontend.md` - Frontend module and service structure
- `../docs/swainos-terminology-glossary.md` - Canonical label standards
- `../docs/sample-payloads.md` - API payload contract examples
- `../docs/frontend-data-queries.md` - Frontend endpoint/query inventory

---

## 📝 **REVISION HISTORY**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.0 | 2026-02-17 | AI Agent + Ian | Initial backend-first Travel Agency + Travel Agent full rollout plan |
| v1.1 | 2026-02-17 | AI Agent + Ian | Marked execution complete; reconciled trade-rule wording and completion status with implemented migrations/APIs/UI/docs |
