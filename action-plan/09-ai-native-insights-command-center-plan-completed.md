# 🎯 AI-Native Insights Command Center - Backend-First Intelligence Platform

> **Version**: v1.3  
> **Status**: ✅ COMPLETED  
> **Date**: 2026-02-17

**Target Components**: `SwianOS_Documentation/supabase/migrations/`, `SwainOS_BackEnd/src/api/`, `SwainOS_BackEnd/src/services/`, `SwainOS_BackEnd/src/repositories/`, `SwainOS_BackEnd/src/schemas/`, `SwainOS_BackEnd/scripts/`, `SwainOS_FrontEnd/apps/web/src/features/ai-insights/`, `SwainOS_FrontEnd/apps/web/src/features/command-center/`, `SwainOS_FrontEnd/apps/web/src/features/travel-consultant/`, `SwainOS_FrontEnd/apps/web/src/components/assistant/`, `SwainOS_FrontEnd/apps/web/src/lib/api/`, `SwainOS_FrontEnd/apps/web/src/lib/types/`, `SwianOS_Documentation/docs/`  
**Primary Issues**: AI surfaces are scaffolded but not powered by a persistent insight pipeline, structured recommendation queue, or domain-level explainable evidence model.  
**Objective**: Deliver a scalable AI-native insights platform where deterministic analytics and OpenAI synthesis produce actionable, auditable, role-aware insights across Command Center, AI Insights, and module-level embeds (including travel consultant intelligence).

## 📋 **QUICK SUMMARY**

**What We're Building/Fixing**: A backend-first AI intelligence layer that transforms existing rollups into compact AI-ready context, stores structured insight events, and powers frontend insight feeds, embedded recommendations, and assistant workflows.

**Critical Issues Being Addressed**:
- AI insight data is not yet persisted as a product data model -> create canonical AI insight/event/recommendation tables and context views
- Current UI has placeholders for AI feeds -> wire live backend-driven briefing, anomaly, recommendation, and history surfaces
- Travel consultant insights are partially deterministic only -> add consultant-level AI narratives and action recommendations with evidence
- Future domains (destination, invoices, FX) need consistent onboarding -> define reusable domain contract for every new AI-enabled data source

**Success Metrics**:
- AI endpoints return structured `{ data, pagination, meta }` envelopes with explainable evidence payloads
- Command Center and AI Insights consume persisted AI outputs (not hardcoded placeholders)
- Travel consultant profile and leaderboard include AI-generated recommendations aligned to deterministic KPIs
- Insight pipeline supports incremental runs after Salesforce sync cycles without full-table prompt loads
- Full auditability exists for AI outputs, status transitions, and user interactions

---

## 🎯 **EXECUTION STATUS**

**Progress**: 6 of 6 sections completed  
**Current Status**: Backend + frontend AI insights platform is complete, validated in UI, and simplified with a Focus/Explore/History navigation model for cleaner AI-first operation.

| Phase | Status | Priority | Notes |
|-------|---------|----------|-------|
| 1️⃣ AI Contract + Domain Guardrails | ✅ COMPLETED | HIGH | Contract and lifecycle/status naming frozen (`snake_case`) |
| 2️⃣ Backend AI Data Foundation (Supabase) | ✅ COMPLETED | HIGH | AI tables, context views, benchmark/company rollups, and refresh RPC shipped |
| 3️⃣ Backend AI Orchestration + API Surface | ✅ COMPLETED | HIGH | AI orchestration, GPT-5.2-first routing, API endpoints, and scripts shipped |
| 4️⃣ Frontend AI Insights Module Integration | ✅ COMPLETED | HIGH | Live briefing/feed/recommendation/history with filters, pagination, and transitions |
| 5️⃣ Embedded Intelligence (Command Center + Travel Consultant + Assistant) | ✅ COMPLETED | HIGH | Command center briefing + travel consultant embeds + assistant context grounding shipped |
| 6️⃣ Cross-Domain Expansion + Governance | ✅ COMPLETED | MEDIUM | Documentation, migration inventory, and operational runbook updated |

---

## 🚨 **CRITICAL REQUIREMENTS**

### **⚠️ MANDATORY CHECKLIST - Must Complete Before Implementation**

These requirements are **NON-NEGOTIABLE** for this action plan.

- [ ] **Type Safety**: All new backend/frontend code uses explicit typing (`mypy`-friendly Python, strict TypeScript) with no `any`
- [ ] **Naming Conventions**: Backend `snake_case`, frontend component `kebab-case.tsx`, utilities/services `camelCase.ts`, API/DB naming aligned to SwainOS conventions
- [ ] **Envelope Consistency**: All API responses continue to use `{ data, pagination, meta }` and standard error envelopes
- [ ] **AI Explainability**: Every AI recommendation includes deterministic evidence fields (`asOfDate`, source metrics, comparison windows, confidence)
- [ ] **Security + Access**: Sensitive insights (compensation, salary-related coaching, pay projections) are role-gated and auditable
- [ ] **Documentation Update**: Update backend/frontend documentation, sample payloads, and query inventory mappings for all new AI surfaces
- [ ] **No Dead Code**: No unused imports, stale scaffolds, commented-out branches, or duplicate insight logic paths

### **Documentation Update Requirement**

> **⚠️ IMPORTANT**: This plan adds backend, frontend, and schema changes and requires:
> - `docs/swainos-code-documentation-backend.md` updates for AI data model, orchestration, and endpoints
> - `docs/swainos-code-documentation-frontend.md` updates for AI Insights page, Command Center briefing, assistant, and travel consultant embeds
> - `docs/sample-payloads.md` updates for all AI endpoint request/response contracts
> - `docs/frontend-data-queries.md` updates for new AI query paths and hook/service usage

---

## 📐 **NAMING CONVENTION ALIGNMENT**

All implementation in this plan must follow SwainOS naming and layering conventions.

### **Backend (FastAPI + Supabase)**
- Files/modules: `snake_case.py`
- Routes/controllers: `src/api/*`
- Business orchestration: `src/services/*`
- Data access: `src/repositories/*`
- Schemas: `src/schemas/*` with camelCase JSON mapping maintained at API boundary
- DB tables/materialized views/indexes: `snake_case` with existing `idx_*` pattern

### **Frontend (Next.js App Router)**
- Feature components: `kebab-case.tsx`
- Hooks: `useX.ts`
- Service clients: `camelCaseService.ts`
- Domain types: `lib/types/*` with `camelCase` properties
- Route files remain thin and delegate logic to `features/*`

### **AI-Specific Naming**
- AI context views: `ai_context_<domain>_v1`
- AI event tables: `ai_insight_events`, `ai_recommendation_queue`, `ai_briefings_daily`
- API family: `/api/v1/ai-insights/*`
- Job/scheduler names: `generate_ai_*`, `refresh_ai_*`

---

## 🔧 **STRATEGIC APPROACH**

### **Implementation Philosophy**
SwainOS AI should be grounded in deterministic business metrics, not raw table prompts. The platform first computes compact, auditable context features from trusted rollups, then uses OpenAI for synthesis, prioritization, and narrative generation. AI outputs are treated as first-class product data with lifecycle status, evidence, and ownership.

### **Key Architecture Decisions**
- **Deterministic-first, LLM-second**: rule/metric computation generates stable evidence; LLM layers reasoning and recommendations
- **Persisted AI outputs**: insights and recommendations live in dedicated tables, not transient response payloads
- **Delta-based orchestration**: after each sync cycle, process changed entities/domains rather than full historical datasets
- **Travel consultant as first-class AI domain**: advisor-level KPIs, funnel health, target pacing, and coaching recommendations are generated with evidence
- **Single insights substrate, multiple UI surfaces**: Command Center, AI Insights page, module embeds, and assistant all consume the same backend insights model

### **Platform Data Flow**

```
Salesforce/other sync jobs
  -> Supabase normalized tables + existing MVs (itinerary/travel consultant/fx/etc)
  -> AI context views (small, explainable, domain-scoped)
  -> AI orchestration service (rules + OpenAI synthesis + policy checks)
  -> persisted outputs (briefings, insight events, recommendation queue, interaction logs)
  -> API endpoints (/api/v1/ai-insights/* + module-specific embeds)
  -> Frontend surfaces (Command Center, AI Insights, Travel Consultant, Assistant)
```

---

## 1️⃣ **PHASE 1: AI CONTRACT + DOMAIN GUARDRAILS**
*Priority: HIGH - Freeze the cross-domain AI contract before implementation*

### **🎯 Objective**
Define one canonical AI insight contract (schema, status lifecycle, evidence payload, confidence semantics) reusable across itinerary, travel consultant, cash flow, FX, destination, and invoices.

### **🔍 Analysis / Discovery**
- Inventory existing deterministic KPI sources (`mv_itinerary_*`, `mv_travel_consultant_*`, `mv_fx_*`)
- Define AI insight types: `briefing`, `anomaly`, `recommendation`, `forecastNarrative`, `coachingSignal`
- Define lifecycle: `new`, `acknowledged`, `in_progress`, `resolved`, `dismissed`
- Define confidence model with explicit source/quality metadata
- Define evidence schema rules so every recommendation references concrete metrics and periods

### **⚙️ Implementation**

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `docs/swainos-code-documentation-backend.md` | Modify | Add AI contract, evidence rules, and service boundaries |
| `docs/sample-payloads.md` | Modify | Add AI insights endpoint payload examples |
| `docs/frontend-data-queries.md` | Modify | Add AI query inventory and contract map |
| `action-plan/09-ai-native-insights-command-center-plan.md` | Modify | Update progress/checklists through implementation |

**Implementation Steps:**
1. Publish AI insight schema dictionary and required fields for every insight type.
2. Freeze domain-specific metric lineage mapping (which rollups feed which insight classes).
3. Finalize role-based field exposure policy for sensitive data (especially travel consultant compensation coaching).
4. Freeze endpoint naming and pagination/meta conventions for AI feeds.

### **✅ Validation Checklist**
- [ ] AI insight schema is documented with required fields and examples
- [ ] Every insight type has deterministic evidence requirements
- [ ] Role-based visibility policy is defined for sensitive fields
- [ ] Contract aligns with existing SwainOS envelope/error patterns

---

## 2️⃣ **PHASE 2: BACKEND AI DATA FOUNDATION (SUPABASE)**
*Priority: HIGH - Create persistent AI data model and context layer*

### **🎯 Objective**
Implement schema and view foundations that make AI computation fast, auditable, and scalable.

### **⚙️ Implementation**

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `supabase/migrations/00xx_create_ai_insight_events.sql` | Create | Core AI insight event table + indexes |
| `supabase/migrations/00xx_create_ai_recommendation_queue.sql` | Create | Action queue with ownership/status |
| `supabase/migrations/00xx_create_ai_briefings_daily.sql` | Create | Persisted command-center briefing snapshots |
| `supabase/migrations/00xx_create_ai_context_views.sql` | Create | Domain context views for compact LLM input |
| `supabase/migrations/00xx_ai_rls_policies.sql` | Create/Modify | RLS + role policy alignment for AI tables |
| `supabase/migrations/00xx_ai_indexes.sql` | Create | Query/performance indexes by domain, status, entity, recency |

**Implementation Steps:**
1. Create AI tables with strict typed columns and jsonb evidence payloads.
2. Build context views:
   - `ai_context_command_center_v1`
   - `ai_context_travel_consultant_v1`
   - `ai_context_itinerary_health_v1`
3. Include travel consultant context features (target variance, funnel bottlenecks, margin shifts, speed-to-book drift, compensation variance signals).
4. Add indexes for feed queries (`domain`, `severity`, `status`, `created_at`, `entity_type`, `entity_id`).
5. Apply RLS policies aligned with existing authenticated/admin/service-role patterns.

### **✅ Validation Checklist**
- [ ] AI schema migrations run cleanly and are reversible
- [ ] Context views produce compact, non-duplicative payloads
- [ ] Travel consultant AI context includes both travel and funnel domain signals
- [ ] RLS and index coverage are validated for expected query patterns

---

## 3️⃣ **PHASE 3: BACKEND AI ORCHESTRATION + API SURFACE**
*Priority: HIGH - Generate, persist, and serve AI insights*

### **🎯 Objective**
Implement backend orchestration that converts deterministic context into structured, explainable AI outputs and exposes APIs for all frontend surfaces.

### **⚙️ Implementation**

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `src/repositories/ai_insights_repository.py` | Create | AI table/context view access methods |
| `src/services/ai_insights_service.py` | Create | Insight generation, ranking, state transitions |
| `src/services/ai_orchestration_service.py` | Create | Pipeline runner (delta detection + model calls) |
| `src/services/openai_insights_service.py` | Create | OpenAI request/response adapter with guardrails |
| `src/schemas/ai_insights.py` | Create | Request/response models and filters |
| `src/api/ai_insights.py` | Create | AI routes (`briefing`, `feed`, `recommendations`, `history`, `entity-insights`) |
| `src/api/router.py` | Modify | Register new AI insights routes |
| `scripts/generate_ai_insights.py` | Create | Canonical scheduler entrypoint for AI insight generation |
| `src/core/config.py` | Modify | AI config settings, model controls, limits |

**Implementation Steps:**
1. Build deterministic pre-checks (missing data, stale context, invalid confidence conditions).
2. Add LLM prompt templates per domain with strict JSON schema output contracts.
3. Persist outputs to AI tables with evidence and source lineage fields.
4. Add APIs:
   - `GET /api/v1/ai-insights/briefing`
   - `GET /api/v1/ai-insights/feed`
   - `GET /api/v1/ai-insights/recommendations`
   - `GET /api/v1/ai-insights/history`
   - `GET /api/v1/ai-insights/entities/{entity_type}/{entity_id}`
   - `PATCH /api/v1/ai-insights/recommendations/{id}` (status transitions)
5. Integrate orchestration with sync cadence so insights regenerate from changed data windows.
6. Add tests for schema validation, deterministic evidence checks, and API contracts.

### **✅ Validation Checklist**
- [ ] Endpoints return envelope-compliant, typed payloads
- [ ] AI outputs persist with evidence and confidence metadata
- [ ] Travel consultant entity endpoint returns actionable consultant-level insights
- [ ] Background generation supports incremental runs and retries
- [ ] Tests cover happy path, stale data path, and invalid model output fallback

---

## 4️⃣ **PHASE 4: FRONTEND AI INSIGHTS MODULE INTEGRATION**
*Priority: HIGH - Replace AI placeholders with live backend data*

### **🎯 Objective**
Implement the AI Insights page as a live operational command surface for anomalies, recommendations, and history.

### **⚙️ Implementation**

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `apps/web/src/lib/types/ai-insights.ts` | Create | Typed AI insight/recommendation/briefing contracts |
| `apps/web/src/lib/api/aiInsightsService.ts` | Create | AI insights API client |
| `apps/web/src/features/ai-insights/useAiInsightsData.ts` | Create | Data orchestration hook for feeds and actions |
| `apps/web/src/features/ai-insights/ai-insights-page.tsx` | Modify | Live cards, feed, recommendation queue, history |
| `apps/web/src/features/ai-insights/anomaly-feed-panel.tsx` | Create | Severity-filterable anomaly/insight stream |
| `apps/web/src/features/ai-insights/recommendation-queue-panel.tsx` | Create | Prioritized recommendation action queue |
| `apps/web/src/features/ai-insights/insight-history-panel.tsx` | Create | Audit-style chronological history |

**Implementation Steps:**
1. Replace current pending placeholders with live fetch states (loading/empty/error/success).
2. Add filters by domain, severity, status, and entity type.
3. Add action controls for recommendation state transitions.
4. Surface evidence snippets and confidence badges without clutter.
5. Preserve existing SwainOS UI primitives and presentation patterns.

### **✅ Validation Checklist**
- [ ] AI Insights page is fully live and no longer scaffold-only
- [ ] Filters and status actions work without breaking envelope contracts
- [ ] Error/loading/empty states are clear and consistent
- [ ] TypeScript/ESLint clean with no dead UI code

---

## 5️⃣ **PHASE 5: EMBEDDED INTELLIGENCE (COMMAND CENTER + TRAVEL CONSULTANT + ASSISTANT)**
*Priority: HIGH - Make AI insights native inside core workflows*

### **🎯 Objective**
Embed persisted AI intelligence in high-leverage workflows, especially travel consultant management and command-center decisioning.

### **⚙️ Implementation**

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `apps/web/src/features/command-center/command-center-page.tsx` | Modify | Replace static briefing block with API-backed daily briefing + top actions |
| `apps/web/src/features/travel-consultant/leaderboard/travel-consultant-leaderboard-page.tsx` | Modify | Add team-level AI coaching strip and prioritized outlier advisor signals |
| `apps/web/src/features/travel-consultant/profile/travel-consultant-profile-page.tsx` | Modify | Add consultant-specific AI action cards + evidence callouts |
| `apps/web/src/components/assistant/assistant-panel.tsx` | Modify | Context-aware insight retrieval for active module/entity |
| `apps/web/src/lib/assistant/assistantContext.tsx` | Modify | Add entity context keys for AI retrieval grounding |
| `apps/web/src/lib/api/travelConsultantService.ts` | Modify | Include AI insight payload surfaces where contract requires |

**Implementation Steps:**
1. Command Center: consume persisted daily briefing and top recommendations.
2. Travel Consultant Leaderboard: show coachable outliers and team trend warnings.
3. Travel Consultant Profile: render advisor-specific recommendations tied to KPI evidence.
4. Assistant panel: query current module/entity insights and present concise action-focused responses.
5. Ensure no contradictory values between deterministic KPI cards and AI narrative text.

### **✅ Validation Checklist**
- [ ] Command Center AI briefing is backend-driven and auditable
- [ ] Travel consultant pages include advisor-level AI insights with evidence
- [ ] Assistant panel is context-grounded to current module/entity
- [ ] Embedded insights align with deterministic KPI values

---

## 6️⃣ **PHASE 6: CROSS-DOMAIN EXPANSION + GOVERNANCE**
*Priority: MEDIUM - Keep the entire platform in scope while scaling safely*

### **🎯 Objective**
Operationalize AI as a reusable platform capability and onboard upcoming domains (destination, invoices, FX) without redesign.

### **🧪 Testing and Governance**
- Add backend unit tests for insight ranking, evidence validation, and confidence gates
- Add integration tests for AI insight APIs and recommendation status transitions
- Add frontend tests for AI feed rendering, filter logic, and action workflows
- Add observability for generation run outcomes, model errors, and cost telemetry
- Add fallback behavior when AI generation fails (serve deterministic-only summaries)

### **📚 Documentation Updates**

**Required Documentation Changes:**

| Document | Section | Change Description |
|----------|---------|-------------------|
| `docs/swainos-code-documentation-backend.md` | AI Insights Architecture | New AI tables, orchestration services, endpoints, and governance |
| `docs/swainos-code-documentation-frontend.md` | AI Surfaces and Assistant UX | AI Insights module, Command Center embeds, travel consultant AI embeds |
| `docs/sample-payloads.md` | AI endpoint contracts | Request/response examples for briefing/feed/history/recommendations |
| `docs/frontend-data-queries.md` | AI data query map | Hook/service routes, filters, and query responsibilities |

### **✅ Validation Checklist**
- [ ] New domains can plug into AI contract by adding context views + orchestration mapping only
- [ ] Observability and cost controls are visible and actionable
- [ ] Documentation accurately reflects end-to-end AI architecture

---

## ⚠️ **RISK MANAGEMENT**

### **High Priority Risks**
- **Hallucinated recommendations**: AI may assert unsupported claims -> **Mitigation**: strict evidence schema, deterministic cross-checks, reject-on-invalid outputs
- **Metric drift between AI and dashboard KPIs**: conflicting numbers reduce trust -> **Mitigation**: AI must consume canonical rollups and return referenced metric keys
- **Sensitive compensation leakage**: consultant pay insights visible to wrong users -> **Mitigation**: role-gated fields, endpoint policy tests, audit logging
- **Runaway token/cost growth**: full-table prompts are expensive -> **Mitigation**: context views, delta processing, hard token budgets, caching

### **Medium Priority Risks**
- **Insight fatigue**: too many low-value alerts -> **Mitigation**: ranking thresholds, severity gating, recommendation dedupe
- **Cross-domain inconsistency**: each module invents its own AI format -> **Mitigation**: enforce single schema contract and shared frontend types
- **Data freshness mismatch**: outdated context powering fresh UI -> **Mitigation**: include `asOfDate`, `generatedAt`, and stale indicators in meta

### **Rollback Strategy**
1. Feature-flag AI endpoints and UI embeds by module.
2. Fall back to deterministic KPI-only rendering with existing backend services.
3. Preserve AI tables for audit; disable generation jobs until fixes are validated.

---

## 📊 **SUCCESS CRITERIA**

### **Technical Success Metrics**

| Metric | Target | Verification Method |
|--------|--------|---------------------|
| Backend schema/API correctness | Contract-compliant | Migration checks + API tests |
| Backend quality gates | Pass | `pytest`, lint, type checks |
| Frontend quality gates | Pass | `npm run lint`, `tsc --noEmit` |
| Insight generation reliability | Stable run success | Job logs + failure telemetry |
| API performance | Dashboard-ready latency | Endpoint timing logs |

### **User Experience Success**

| Scenario | Expected Outcome |
|----------|------------------|
| Open AI Insights page | Live briefing, anomaly feed, recommendation queue, and history render from backend |
| Review Travel Consultant module | Advisor-level AI coaching is visible, actionable, and evidence-backed |
| Open Command Center | Daily briefing and top priorities are current, clear, and aligned to KPIs |
| Use assistant in a module | Responses are grounded to current context and reflect persisted insights |

---

## 🔗 **RELATED DOCUMENTATION**

- **`action-plan/08-travel-consultant-analytics-bible-plan-completed.md`** - Existing travel consultant KPI and API foundation
- **`action-plan/07-itinerary-status-pipeline-plan-completed.md`** - Rollup-first analytics implementation pattern
- **`docs/swainos-code-documentation-backend.md`** - Current backend architecture and endpoint inventory
- **`docs/swainos-code-documentation-frontend.md`** - Current frontend module and assistant scaffolding
- **`docs/SwainOS_Project_Specification.pdf`** - AI-first system direction and source integration scope

---

## 🎯 **COMPLETION CHECKLIST**

### **Pre-Implementation**
- [x] Confirm AI insight schema and domain contracts with stakeholders
- [x] Validate travel consultant AI insight requirements and role visibility rules
- [x] Confirm backend-first sequencing and feature-flag strategy

### **Implementation Quality Gates**
- [x] Backend code follows routes -> services -> repositories layering
- [x] Frontend code follows feature-module organization and naming conventions
- [x] No `any`, no dead code, no inconsistent schema naming
- [x] API envelopes and error contracts remain consistent

### **Testing**
- [x] AI generation and validation logic tested (unit/integration)
- [x] Feed/recommendation/history endpoints tested with realistic fixtures
- [x] Frontend AI views tested for loading/empty/error/success paths
- [x] Deterministic metric parity checks pass for AI evidence references

### **Documentation** *(MANDATORY)*
- [x] `swainos-code-documentation-backend.md` updated
- [x] `swainos-code-documentation-frontend.md` updated
- [x] `sample-payloads.md` updated for all AI contracts
- [x] `frontend-data-queries.md` updated for AI data access patterns
- [x] Action plan status updated to ✅ COMPLETED when all validations pass

### **Final Review**
- [x] All build phases completed
- [x] No unresolved security or data-quality concerns
- [x] Platform is ready for additional domains (destination, invoices, FX) using the same AI contract

---

## 📝 **REVISION HISTORY**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.0 | 2026-02-16 | SwainOS Assistant | Initial backend-first AI-native insights action plan with travel consultant and cross-domain platform scope |
| v1.1 | 2026-02-16 | SwainOS Assistant | Normalized AI lifecycle status values to snake_case and locked scheduler path to `scripts/generate_ai_insights.py` for repository consistency |
| v1.2 | 2026-02-16 | SwainOS Assistant | Marked implementation complete, updated execution matrix/checklists, and aligned file inventory to shipped anomaly-feed panel naming |
| v1.3 | 2026-02-17 | SwainOS Assistant | Updated current-state notes for completed AI Insights UX simplification and finalized plan metadata before moving file to completed naming convention |
