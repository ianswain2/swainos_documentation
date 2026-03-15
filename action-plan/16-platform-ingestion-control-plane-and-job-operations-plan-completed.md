# 🎯 Platform Ingestion Control Plane and Job Operations Plan - Supabase-First Scheduled Data Architecture

> **Version**: v1.3  
> **Status**: ✅ COMPLETED  
> **Date**: 2026-03-09

**Target Components**: `SwainOS_BackEnd/supabase/migrations/`, `SwainOS_BackEnd/src/api/`, `SwainOS_BackEnd/src/services/`, `SwainOS_BackEnd/src/repositories/`, `SwainOS_BackEnd/src/schemas/`, `SwainOS_BackEnd/src/integrations/`, `SwainOS_BackEnd/scripts/`, `SwainOS_FrontEnd/apps/web/src/app/settings/`, `SwainOS_FrontEnd/apps/web/src/app/operations/`, `SwainOS_FrontEnd/apps/web/src/features/settings/`, `SwainOS_FrontEnd/apps/web/src/features/operations/`, `SwainOS_FrontEnd/apps/web/src/lib/api/`, `SwainOS_FrontEnd/apps/web/src/lib/types/`, `SwianOS_Documentation/docs/`  
**Primary Issues**: External-source reads are not governed by a single platform-wide ingestion model, some backend read paths still trigger runtime sync/generation, source-specific manual run endpoints are fragmented, Supabase rollups are not orchestrated under one dependency graph, and the frontend does not yet expose a canonical jobs/settings surface with schedules, health, or manual run controls.  
**Objective**: Replace all request-time external pulls and fragmented sync mechanics with a single scheduled, Supabase-first ingestion control plane that standardizes job definitions, run history, dependency-driven rollups, manual run controls, frontend settings/operations UX, and removal of old source-specific compatibility paths.

## 📋 **QUICK SUMMARY**

**What We're Building/Fixing**: A unified SwainOS ingestion platform where every external pull and downstream Supabase rollup runs through the same job orchestration contract, while frontend and API reads become database-only and operators manage schedules and manual runs from dedicated Settings and Operations pages.

**Critical Issues Being Addressed**:
- Request-time sync behavior exists in live code -> replace with scheduled jobs and explicit freshness/degraded states.
- Source-specific job logic is fragmented across Marketing, FX, AI, and scripts -> consolidate behind one canonical job model and execution pipeline.
- Supabase rollups are refreshed inconsistently per domain -> make rollups first-class downstream jobs with dependency ordering and run tracking.
- Manual/import-style ingestion scripts and one-off operational runners are currently outside a single job inventory -> fold them into the same control plane or explicitly classify them as manual/backfill jobs.
- Schedule source-of-truth is not yet fixed -> frontend-editable schedules require a scheduler model that can read from database state, not only static deployment config.
- Settings and operations surfaces are placeholders -> replace with real job administration and observability UI.
- Old run endpoints and one-off orchestration paths create tech debt -> rip them out and move to one unified control plane without backward-compatibility shims.

**Success Metrics**:
- No frontend page load or backend GET path performs external-source fetches or triggers sync/generation work.
- All external-source and derived rollup runs are represented in a unified `data_jobs` + `data_job_runs` operating model.
- Settings page shows every job, its schedule, freshness SLA, last/next run, and manual run control.
- Operations page shows run history, failures, blocked dependencies, and drill-down diagnostics.
- Supabase rollups run only after upstream load jobs succeed or explicitly enter degraded/blocked state.
- Old source-specific sync/run patterns are removed once the new job control plane is active.

---

## 🎯 **EXECUTION STATUS**

**Progress**: 8 of 8 phases completed  
**Current Status**: Implemented and validated in production-oriented workflow. The unified ingestion control plane, scheduler guardrails, Settings/Operations admin UX, and run observability are active.

| Phase | Status | Priority | Notes |
|-------|---------|----------|-------|
| 1️⃣ Domain Inventory + Contract Freeze | ✅ COMPLETED | HIGH | Canonical inventory and scheduler model frozen in execution docs |
| 2️⃣ Supabase Job Control Plane Foundation | ✅ COMPLETED | HIGH | `data_jobs`, dependencies, runs, steps, health model, and follow-on guardrail migrations delivered |
| 3️⃣ Backend Orchestration + Unified APIs | ✅ COMPLETED | HIGH | Unified `/api/v1/data-jobs*` and `/api/v1/data-job-runs/{run_id}` surface shipped |
| 4️⃣ Source Migration + Read-Path Ripout | ✅ COMPLETED | HIGH | Request-time sync/generation removed from targeted paths; legacy run endpoints removed |
| 5️⃣ Rollup Dependency Graph + Post-Load Refresh | ✅ COMPLETED | HIGH | Dependency-driven orchestration and blocked-state handling active |
| 6️⃣ Frontend Settings + Operations UX | ✅ COMPLETED | HIGH | Real Settings/Operations surfaces live, including Settings Run Logs |
| 7️⃣ Tech-Debt Removal + Naming/Folder Cleanup | ✅ COMPLETED | HIGH | Legacy/manual-run compatibility paths removed in favor of canonical control plane |
| 8️⃣ Validation, Docs, and Operational Handoff | ✅ COMPLETED | HIGH | Lint/type/tests + docs sync + action-log milestones completed |

---

## 🚨 **CRITICAL REQUIREMENTS**

### **⚠️ MANDATORY CHECKLIST - Must Complete Before Implementation**

- [ ] **No Request-Time External Pulls**: All external-source reads move out of GET/page-load execution paths.
- [ ] **No Backward Compatibility Layer**: Old source-specific run/sync paths are removed after migration; no dual systems remain.
- [ ] **Supabase-First Serving**: Frontend and read APIs serve from Supabase-backed state only; deterministic service-layer computation over Supabase data is allowed, but no read path may call external systems or trigger sync/generation.
- [ ] **Dependency-Driven Rollups**: Supabase rollups/materializations are explicit downstream jobs, not ad-hoc side effects.
- [ ] **Unified Job Contract**: Marketing, FX, AI, Salesforce, and future sources use one job/run model.
- [ ] **Strict Naming Conventions**: All new files, routes, services, schemas, jobs, tables, and UI modules follow SwainOS rules exactly.
- [ ] **Folder Structure Discipline**: Jobs, repositories, schemas, scripts, and frontend features are reorganized cleanly; no scattered orchestration logic.
- [ ] **Type Safety**: Strict TypeScript/Python typing only; no `any` in new frontend contracts.
- [ ] **Tech-Debt Removal**: Any replaced legacy logic, run endpoint, placeholder UI, or dead helper is deleted in the same rollout.
- [ ] **Documentation Update**: `swainos-code-documentation-backend.md`, `swainos-code-documentation-frontend.md`, `frontend-data-queries.md`, and `sample-payloads.md` are updated to reflect the new job platform.
- [ ] **Operational Observability**: Every run records status, timing, row counts, dependency outcomes, rollup outcomes, and actionable errors.

### **Documentation Update Requirement**

> **⚠️ IMPORTANT**: This rollout must update:
> - `docs/swainos-code-documentation-backend.md`
> - `docs/swainos-code-documentation-frontend.md`
> - `docs/frontend-data-queries.md`
> - `docs/sample-payloads.md`
> - `docs/swainos-terminology-glossary.md` (if new job/health terminology is introduced)

---

## 📐 **NAMING + STRUCTURE ALIGNMENT**

All code in this plan follows current SwainOS conventions and removes legacy naming drift instead of preserving it.

### **Backend**

| Element | Convention | Example |
|---------|------------|---------|
| Job modules | `snake_case.py` | `data_job_service.py` |
| Job runner scripts | `snake_case.py` | `run_data_job.py` |
| Repository modules | `snake_case.py` | `data_job_repository.py` |
| API routes | `kebab-case` in path | `/api/v1/data-jobs` |
| Query params | `snake_case` | `job_key`, `run_type` |
| JSON fields | `camelCase` | `lastRunAt`, `freshnessSlaMinutes` |
| Database tables | `snake_case`, plural | `data_jobs`, `data_job_runs` |
| Database columns | `snake_case` | `job_key`, `next_run_at` |

### **Frontend**

| Element | Convention | Example |
|---------|------------|---------|
| Route folders | `kebab-case` | `app/settings/data-jobs/` |
| Feature files | `kebab-case.tsx` | `data-jobs-settings-page.tsx` |
| Service files | `camelCaseService.ts` | `dataJobsService.ts` |
| Type files | `lowercase.ts` | `data-jobs.ts` |
| Utility files | `camelCase.ts` | `formatJobStatus.ts` |

### **Folder Ownership**

- `src/api/data_jobs.py` becomes the single backend surface for job administration.
- `src/services/data_job_service.py` becomes the canonical orchestration entrypoint.
- `src/repositories/data_job_repository.py` owns job definitions, run records, dependency queries, and health reads.
- `src/services/job_runners/` becomes the home for source-specific execution implementations.
- `src/services/rollup_runners/` becomes the home for downstream Supabase refresh implementations.
- `apps/web/src/features/settings/` owns job settings/admin UX.
- `apps/web/src/features/operations/` owns run history, failure diagnostics, and operational visibility.

---

## 🚨 **CURRENT STATE FINDINGS THAT MUST BE FIXED**

### **Request-Time Sync / Generation Debt**

- Marketing read paths still call sync logic when snapshot data is missing or stale:
  - `src/services/marketing_web_analytics_service.py`
- Search Console still triggers snapshot sync inside read service code when freshness thresholds fail:
  - `src/services/marketing_web_analytics_service.py`
- Debt schedule rows are generated on read when missing instead of being managed as a background/precompute workflow:
  - `src/services/debt_service_service.py`

### **Fragmented Manual Run Surface**

- Manual run endpoints are currently source-specific and inconsistent:
  - `POST /api/v1/marketing/web-analytics/sync/run`
  - `POST /api/v1/fx/rates/run`
  - `POST /api/v1/fx/signals/run`
  - `POST /api/v1/fx/intelligence/run`
  - `POST /api/v1/ai-insights/run`
- These must be replaced by one unified administration surface and one unified backend contract.
- There is also a frontend-only FX proxy run path that should not survive the migration:
  - `apps/web/src/app/api/fx/rates/run/route.ts`

### **Existing Operational Assets to Reuse**

- Marketing runtime sync script: `scripts/sync_marketing_web_analytics.py`
- FX runtime scripts:
  - `scripts/pull_fx_rates.py`
  - `scripts/generate_fx_intelligence.py`
  - `scripts/refresh_fx_exposure.py`
  - `scripts/backfill_fx_rates_history.py`
- Salesforce runtime sync:
  - `scripts/sync_salesforce_readonly.py`
  - `scripts/validate_salesforce_readonly_permissions.py`
- Salesforce sub-runner scripts used by the parent sync workflow and expected to become implementation details behind the registered Salesforce runner:
  - `scripts/upsert_agencies.py`
  - `scripts/upsert_suppliers.py`
  - `scripts/upsert_employees.py`
  - `scripts/upsert_itineraries.py`
  - `scripts/upsert_itinerary_items.py`
- Existing manual/import ingestion scripts that must be explicitly inventoried under the new platform model:
  - `scripts/upsert_bookings.py`
  - `scripts/upsert_customer_payments.py`
  - `scripts/upsert_supplier_invoices.py`
  - `scripts/upsert_supplier_invoice_bookings.py`
  - `scripts/upsert_supplier_invoice_lines.py`
- Existing rollup refresh scripts:
  - `scripts/refresh_consultant_ai_rollups.py`
  - `scripts/refresh_travel_trade_rollups.py`
- Existing maintenance/operational scripts that should be classified during inventory rather than left as tribal-knowledge utilities:
  - `scripts/generate_ai_insights.py`
  - `scripts/purge_ai_insights.py`
  - `scripts/cleanup_inactive_employees.py`
- Existing placeholder frontend admin routes already exist and should be upgraded instead of duplicated:
  - `/settings`
  - `/operations`

### **Placeholder Frontend Debt**

- `apps/web/src/features/settings/settings-page.tsx` is still placeholder-only.
- `apps/web/src/features/operations/operations-page.tsx` is still placeholder-only.
- These placeholders should be replaced by real job administration and observability UI, not wrapped or preserved.

---

## 🔧 **STRATEGIC APPROACH**

### **Implementation Philosophy**

SwainOS needs one platform-wide operating model for data ingestion:

- external systems are pulled on a controlled cadence,
- raw/canonical facts are persisted to Supabase,
- deterministic validation runs before publishing,
- downstream rollups/materializations are refreshed in dependency order,
- frontend and read APIs consume only persisted data,
- operators manage everything from one job settings and run-operations experience.

This is a rip-and-replace architecture correction, not a compatibility exercise. The goal is a simpler, stricter, more auditable system with fewer hidden behaviors and lower ongoing operational cost.

Important alignment rule: this migration removes request-time external pulls and hidden sync side effects. It does not require every deterministic GET endpoint to become a materialized view if the endpoint can safely compute from Supabase-backed data at request time.

### **Key Architecture Decisions**

- **One job platform, many jobs**: every external pull and rollup is modeled as a job with the same contract.
- **Definitions separate from runs**: job metadata lives separately from execution history for clean admin UX and safe auditing.
- **Dependencies are explicit**: rollups never run “because some code happened to call them”; they run only because the dependency graph says they should.
- **Scheduler truth is dynamic and stored in Supabase**: if `/settings` can change cadence, runtime scheduling cannot rely on static per-job deployment cron config.
- **Scheduler model is Option 2: dynamic poller scheduler**: one fixed recurring scheduler tick checks Supabase-backed job definitions and dispatches only due jobs.
- **Read paths are pure**: GET endpoints and server loaders never mutate runtime state.
- **Settings and operations are distinct**:
  - `/settings` = configuration, schedule, enabled/disabled, manual-run entrypoint
  - `/operations` = history, failures, diagnostics, dependency outcomes
- **No source-by-source compatibility shims**: replace old run endpoints, old trigger paths, and placeholder settings UX completely once the new control plane ships.

### **Canonical Job Lifecycle**

1. Scheduler selects due jobs.
2. Job runner acquires lock and creates run record.
3. Extract/load writes raw or canonical source facts.
4. Validation computes counts, freshness, and integrity checks.
5. Downstream rollup jobs execute according to dependency graph.
6. Job run publishes final status with metrics.
7. Frontend and read APIs show freshness and run metadata only; no hidden sync occurs.

### **Platform Data Flow**

```
External Provider / Runtime Script
  -> data_jobs definition
  -> data_job_runs execution record
  -> canonical source tables in Supabase
  -> validation results
  -> downstream rollup jobs / RPC refresh jobs
  -> published read views / RPCs / materialized views
  -> backend GET endpoints
  -> frontend Settings and Operations surfaces
```

### **Target Job Families**

**External-source ingestion jobs**
- `marketing-ga4-sync`
- `marketing-search-console-sync`
- `fx-rates-pull`
- `fx-intelligence-generate`
- `salesforce-readonly-sync`
- future sources such as QuickBooks or other operational providers

**Manual import / batch ingestion jobs**
- `bookings-import`
- `customer-payments-import`
- `supplier-invoices-import`
- `supplier-invoice-bookings-import`
- `supplier-invoice-lines-import`

**Derived compute / publication jobs**
- `fx-exposure-refresh`
- `fx-signals-generate`
- `ai-insights-generate`
- `travel-trade-rollups-refresh`
- `consultant-ai-rollups-refresh`
- `marketing-search-console-rollups-refresh`
- future domain rollups/materializations

**Precompute / internal state jobs**
- `debt-schedule-materialize`
- future cached/prebuilt operational datasets that are currently generated on read

**Maintenance / validation / backfill jobs**
- `fx-rates-history-backfill`
- `salesforce-permission-validate`
- `ai-insights-purge`
- `inactive-employees-cleanup`

---

## 1️⃣ **PHASE 1: DOMAIN INVENTORY + CONTRACT FREEZE**
*Priority: High - define the full scope before building the platform*

### **🎯 Objective**
Inventory every source pull, every downstream rollup, every existing run path, and every read-time mutation that must be migrated into the new job control plane.

### **🔍 Analysis / Discovery**

**Job inventory must classify each workload by type:**
- `source_ingestion`
- `rollup_refresh`
- `derived_compute`
- `maintenance`

**Each inventory item must capture:**
- `jobKey`
- `jobGroup`
- `sourceSystem`
- `runTypeSupport` (`scheduled`, `manual`, optionally `backfill`)
- `scheduleMode` (`scheduled`, `manual_only`, `backfill_only`, `system_managed`)
- cadence
- freshness SLA
- lock strategy
- dependency list
- canonical output tables/views/RPCs
- frontend surfaces that depend on it
- old code paths/endpoints to remove

**Initial source inventory for this rollout:**
- Marketing GA4 snapshots
- Marketing Search Console snapshots
- FX rates
- FX intelligence/news synthesis
- FX exposure refresh
- FX signal generation
- AI insights generation
- Salesforce read-only sync
- Bookings import
- Customer payments import
- Supplier invoices import
- Supplier invoice bookings import
- Supplier invoice lines import
- Travel trade rollups refresh
- Consultant AI rollups refresh
- Debt schedule precompute/materialization
- FX history backfill
- Salesforce permission validation
- AI insights purge / maintenance tasks

### **⚙️ Implementation**

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `docs/swainos-code-documentation-backend.md` | Modify | Add canonical job platform architecture and current-state migration targets |
| `docs/swainos-code-documentation-frontend.md` | Modify | Add settings/operations job UX and route ownership |
| `docs/frontend-data-queries.md` | Modify | Add new job admin endpoints and remove deprecated source-specific run endpoints |
| `docs/sample-payloads.md` | Modify | Add job definition/job run payload contracts |

**Implementation Steps:**
1. Build the complete job inventory and dependency matrix.
2. Freeze the canonical job schema and naming vocabulary.
3. Freeze Option 2 as the scheduler model: dynamic poller scheduler with Supabase as schedule source-of-truth.
4. Freeze which existing endpoints/scripts remain, move, or are deleted.
5. Freeze which page reads become pure Supabase-only reads.
6. Freeze Settings vs Operations UI ownership.

### **✅ Validation Checklist**
- [ ] Every current source and rollup is listed.
- [ ] Every read-time mutation path is explicitly identified for removal.
- [ ] Every old run endpoint has a replacement or deletion path.
- [ ] Inventory is sufficient to implement without “discovering architecture during coding.”

---

## 2️⃣ **PHASE 2: SUPABASE JOB CONTROL PLANE FOUNDATION**
*Priority: High - establish the canonical runtime model*

### **🎯 Objective**
Create the Supabase schema that defines jobs, schedules, dependencies, run history, manual-run requests, and execution health.

### **🔄 Implementation**

**Core schema objects (v1):**
- `data_jobs`
- `data_job_dependencies`
- `data_job_runs`
- `data_job_run_steps`
- `data_job_run_artifacts` (optional if step payloads need separation)
- `data_job_manual_run_requests` (optional if manual request tracking is separated from runs)
- `data_job_health_v1` (view or RPC-backed read model for frontend)

**Suggested fields for `data_jobs`:**
- `id`
- `job_key` (unique)
- `job_name`
- `job_group`
- `job_type`
- `source_system`
- `runner_key`
- `schedule_cron`
- `schedule_timezone`
- `enabled`
- `manual_run_allowed`
- `freshness_sla_minutes`
- `timeout_seconds`
- `retry_limit`
- `stale_after_minutes`
- `notes`
- `created_at`
- `updated_at`

**Suggested fields for `data_job_runs`:**
- `id`
- `job_id`
- `job_key`
- `run_type`
- `triggered_by`
- `status`
- `started_at`
- `completed_at`
- `duration_ms`
- `records_processed`
- `records_created`
- `records_updated`
- `records_deleted`
- `error_message`
- `dependency_status`
- `input_watermark`
- `output_as_of_date`
- `metadata`

**Suggested step tracking (`data_job_run_steps`):**
- `step_key`
- `step_type` (`extract`, `load`, `validate`, `rollup`, `publish`)
- `status`
- `started_at`
- `completed_at`
- `records_processed`
- `error_message`
- `metadata`

**Dependency graph requirements:**
- Upstream/downstream relationships are modeled in DB, not hardcoded only in service code.
- Blocked downstream jobs are visible as `blocked`, not invisible skips.
- Rollups can depend on one or multiple upstream ingestion jobs.

**Indexes and constraints:**
- unique `data_jobs.job_key`
- unique `data_job_dependencies(job_id, depends_on_job_id)`
- `idx_data_job_runs_job_started_at`
- `idx_data_job_runs_status_started_at`
- `idx_data_job_runs_job_completed_at`
- `idx_data_job_run_steps_run_step_key`
- `idx_data_jobs_enabled_schedule`

**Policy requirements:**
- authenticated read for job definitions and run history
- service/admin write for runs and config updates
- protected path for manual-run triggering

### **✅ Validation Checklist**
- [ ] Schema supports all current job families without special-case tables.
- [ ] Dependency modeling supports rollups after one or multiple upstream jobs.
- [ ] Run history can power both Settings summary and Operations drill-down.
- [ ] Indexes align to admin and scheduler access patterns.
- [ ] Policy model matches admin/runtime responsibilities.

---

## 3️⃣ **PHASE 3: BACKEND ORCHESTRATION + UNIFIED APIS**
*Priority: High - replace fragmented run logic with one backend platform*

### **🎯 Objective**
Implement a single orchestration layer and a single API family that manages all jobs, run history, health, and manual triggers.

### **🔄 Implementation**

**Canonical backend modules:**
- `src/api/data_jobs.py`
- `src/services/data_job_service.py`
- `src/repositories/data_job_repository.py`
- `src/schemas/data_jobs.py`
- `src/services/job_runners/*`
- `src/services/rollup_runners/*`

**Unified API surface (`/api/v1/data-jobs/*`):**
- `GET /api/v1/data-jobs`
- `GET /api/v1/data-jobs/{job_key}`
- `PATCH /api/v1/data-jobs/{job_key}` for admin schedule/enabled updates
- `POST /api/v1/data-jobs/{job_key}/runs`
- `GET /api/v1/data-jobs/{job_key}/runs`
- `GET /api/v1/data-job-runs/{run_id}`
- `GET /api/v1/data-jobs/health`

**Manual run standardization:**
- All source-specific run buttons call the same job-run endpoint pattern.
- Existing endpoints such as:
  - `/marketing/web-analytics/sync/run`
  - `/fx/rates/run`
  - `/fx/signals/run`
  - `/fx/intelligence/run`
  - `/ai-insights/run`
  are removed after migration and replaced by job-key based execution.
- The frontend FX proxy route `app/api/fx/rates/run/route.ts` is also removed once the unified job-run contract is in place.

**Runner registration model:**
- job definition stores `runner_key`
- orchestration service resolves runner via explicit registry
- no hidden dynamic imports or implicit script-name conventions

**Scheduler contract:**
- lightweight due-job selector calls job service
- lock acquisition prevents overlapping runs
- failed jobs do not duplicate concurrent runs
- blocked jobs are marked clearly
- scheduler implementation must support database-driven job schedules if schedule editing is exposed in `/settings`

**Chosen scheduler implementation for this rollout:**
- A single fixed recurring scheduler tick invokes the backend scheduler entrypoint.
- The backend scheduler reads due jobs from Supabase-backed job definitions.
- Supabase stores the editable schedule truth for each job (`schedule_cron`, `schedule_timezone`, `enabled`, `next_run_at`).
- Vercel, if used, acts only as the fixed scheduler tick trigger and not as the per-job schedule source-of-truth.

**Scheduler architecture options (must choose in Phase 1):**
- **Static platform cron**: deployment config triggers named job endpoints on fixed cadence
- **Dynamic poller scheduler**: one recurring scheduler tick asks the backend/DB which jobs are due
- **Database-native scheduler**: schedule state lives in DB and jobs are dispatched from DB/worker infrastructure

**Recommendation baseline for this rollout:**
- Prefer a dynamic poller or database-native scheduler if product-admin schedule editing is a real requirement.
- Use static Vercel cron only if schedule editing in the UI is display-only or intentionally constrained to a very small fixed set of schedules managed outside the app.

**Script unification strategy:**
- retain proven scripts where practical, but invoke them under the job platform rather than as standalone tribal-knowledge flows
- scripts become implementation detail behind registered runners, not the operator-facing model
- parent jobs may own internal sub-runner scripts (for example Salesforce extract plus object-specific upsert loaders) without exposing each sub-runner as a first-class scheduled job

### **✅ Validation Checklist**
- [ ] One canonical API family covers definitions, health, history, and manual runs.
- [ ] No source-specific manual run endpoint remains as the primary contract.
- [ ] Job service is the only orchestration entrypoint.
- [ ] Locks, statuses, and dependency failures are modeled consistently.
- [ ] Envelope contract remains `{ data, pagination, meta }`.

---

## 4️⃣ **PHASE 4: SOURCE MIGRATION + READ-PATH RIPOUT**
*Priority: High - eliminate hidden runtime work and move sources into scheduled pipelines*

### **🎯 Objective**
Migrate every current source and on-read generation path into the job platform, then remove old request-time execution logic.

### **🔄 Implementation**

### **Marketing**

**Current problems:**
- Marketing overview/page-activity/geo/events can still trigger sync behavior through shared loaders.
- Search Console can still trigger snapshot sync when stale.

**Target state:**
- `marketing-ga4-sync` and `marketing-search-console-sync` become scheduled jobs.
- marketing read endpoints become pure reads from canonical facts and rollups.
- stale data returns health metadata only; no sync is executed inside GET endpoints.

### **FX**

**Current problems:**
- Rates, intelligence, and signals use fragmented run endpoints.
- orchestration is still source-specific instead of platform-standard.

**Target state:**
- `fx-rates-pull`, `fx-intelligence-generate`, `fx-exposure-refresh`, and `fx-signals-generate` become registered jobs.
- FX pages read only persisted rates/signals/intelligence/exposure.
- old run endpoints are removed after frontend migration.

### **AI Insights**

**Current problems:**
- AI manual generation has a dedicated run endpoint separate from a wider job model.

**Target state:**
- `ai-insights-generate` is a first-class job with schedule and manual trigger support.
- operator visibility moves to Settings/Operations rather than hidden endpoint knowledge.

### **Salesforce**

**Current problems:**
- robust script exists, but it is not yet governed by the same platform contract as the rest of the system.

**Target state:**
- `salesforce-readonly-sync` becomes a registered scheduled job.
- run metadata and admin controls appear beside all other jobs.
- future data-source additions follow the same pattern immediately.

### **Operational Imports / Canonical Finance-Supporting Loads**

**Current problems:**
- Several important ingestion scripts exist as manual CSV/REST upsert workflows but are not represented in the current plan.
- If left out of the control-plane inventory, these workflows will remain undocumented operational debt outside the new architecture.

**Target state:**
- `bookings-import`
- `customer-payments-import`
- `supplier-invoices-import`
- `supplier-invoice-bookings-import`
- `supplier-invoice-lines-import`

These are explicitly classified as manual-import or batch-ingestion jobs under the same run model, even if they are not all scheduled on day one.

### **Debt Schedule / Internal Precompute**

**Current problems:**
- debt schedule rows are generated when requested if missing.

**Target state:**
- schedule materialization becomes explicit background/precompute job behavior.
- GET endpoints return stored schedule or explicit missing/degraded state, not hidden generation.

### **✅ Validation Checklist**
- [ ] Marketing GET endpoints no longer trigger sync.
- [ ] Search Console GET endpoints no longer trigger sync.
- [ ] FX run endpoints are migrated behind canonical jobs.
- [ ] AI run endpoint is migrated behind canonical jobs.
- [ ] Salesforce sync is visible in the same control plane.
- [ ] Existing import/upsert workflows are explicitly represented as jobs or formally classified as manual-import workloads in the same platform model.
- [ ] Debt schedule generation-on-read is removed or reworked into an explicit job/precompute path.

---

## 5️⃣ **PHASE 5: ROLLUP DEPENDENCY GRAPH + POST-LOAD REFRESH**
*Priority: High - make Supabase rollups a first-class downstream system*

### **🎯 Objective**
Standardize every rollup/materialization/RPC refresh as a dependency-driven downstream job with observable outcomes.

### **🔄 Implementation**

**Rollup orchestration rules:**
- rollups run after successful upstream loads
- rollups can skip with `blocked` if upstream fails
- rollups record their own step/run data
- rollup freshness is published in the same health model as source jobs

**Known current rollup/refresh candidates:**
- `refresh_consultant_ai_rollups_v1`
- `refresh_travel_trade_rollups_v1`
- `refresh_fx_exposure_v1`
- Search Console workspace/page-profile RPC-backed publication flow
- AI insight generation dependency on refreshed consultant/AI context views
- future job-based refreshes for other finance/travel domains

**Required rollup design outputs:**
- one dependency map per rollup
- explicit success/failure/blocked statuses
- one canonical place to store rollup metadata and `as_of_date`
- no hidden “refresh if requested” side effects inside unrelated scripts or services

**Example dependency chains:**
- `fx-rates-pull` -> `fx-exposure-refresh` -> `fx-signals-generate`
- `marketing-ga4-sync` + `marketing-search-console-sync` -> `marketing-search-console-rollups-refresh`
- `salesforce-readonly-sync` -> `travel-trade-rollups-refresh`
- `salesforce-readonly-sync` -> `consultant-ai-rollups-refresh` -> `ai-insights-generate`
- `salesforce-readonly-sync` -> future AP/customer payment rollups as required
- `bookings-import` / `customer-payments-import` / supplier-invoice import jobs -> downstream liquidity/AP rollups when those refreshes are formalized under the same platform

**Supabase RPC / script strategy:**
- keep existing RPCs where they are correct and fast
- move invocation under registered rollup runners
- publish rollup results back into `data_job_runs` / `data_job_run_steps`

### **✅ Validation Checklist**
- [ ] Every rollup has an explicit upstream dependency map.
- [ ] Rollup invocation is visible and auditable.
- [ ] Rollup failure does not silently disappear behind a successful source pull.
- [ ] Freshness and last-run timestamps are exposed to frontend admin surfaces.

---

## 6️⃣ **PHASE 6: FRONTEND SETTINGS + OPERATIONS UX**
*Priority: High - give operators full control and visibility*

### **🎯 Objective**
Replace placeholder `/settings` and `/operations` pages with real, typed, platform-admin UI for job configuration and run monitoring.

### **🔄 Implementation**

### **`/settings` Responsibilities**

**Primary page goal**: configure and control jobs.

**Required UI elements per job row/card:**
- job name
- source system
- enabled/disabled status
- schedule time / cron summary for schedulable jobs
- `Manual only` / `Backfill only` / `System managed` label for non-schedulable jobs
- timezone
- freshness SLA
- last successful run
- next scheduled run
- current health state
- manual run button
- dependency badge/count
- quick link to history/details

**Required actions:**
- enable/disable job
- edit schedule for schedulable jobs only
- optionally edit freshness SLA and timeout where allowed
- manual run

**Suggested frontend structure:**
- `app/settings/page.tsx`
- `features/settings/data-jobs-settings-page.tsx`
- `features/settings/data-jobs-settings-grid.tsx`
- `features/settings/data-job-settings-card.tsx`
- `lib/api/dataJobsService.ts`
- `lib/types/data-jobs.ts`

### **`/operations` Responsibilities**

**Primary page goal**: inspect execution history and failures.

**Required UI sections:**
- latest failed jobs
- latest blocked jobs
- recent run history table
- job detail drawer/page
- step timeline (`extract`, `load`, `validate`, `rollup`, `publish`)
- metrics panel (`recordsProcessed`, duration, as-of date, dependency result)
- error details and retry/manual rerun action

**Suggested frontend structure:**
- `app/operations/page.tsx`
- `features/operations/data-job-operations-page.tsx`
- `features/operations/data-job-runs-table.tsx`
- `features/operations/data-job-run-detail.tsx`
- `features/operations/data-job-health-summary.tsx`

### **Navigation cleanup**

- Keep `/settings` and `/operations` as the admin destinations already present in navigation.
- Do not add duplicate “sync admin” routes elsewhere.
- Any existing feature-specific run buttons should either:
  - be removed, or
  - deep-link into the unified job settings/detail experience.

### **✅ Validation Checklist**
- [ ] `/settings` is a real job admin page, not a placeholder.
- [ ] `/operations` is a real run-monitoring page, not a placeholder.
- [ ] Each job shows schedule time and manual-run control.
- [ ] Each job shows health, freshness, last run, and next run.
- [ ] UI contracts are strictly typed and follow naming conventions.
- [ ] No duplicate admin experiences are introduced.

---

## 7️⃣ **PHASE 7: TECH-DEBT REMOVAL + NAMING/FOLDER CLEANUP**
*Priority: High - finish the migration cleanly and remove obsolete architecture*

### **🎯 Objective**
Remove old code paths, obsolete endpoints, placeholder admin UI, and scattered orchestration logic so the new platform is the only operating model.

### **🔄 Implementation**

**Code to remove or replace:**
- source-specific run endpoints superseded by `/api/v1/data-jobs/*`
- request-time sync logic in marketing services
- request-time generation logic in debt schedule path
- frontend FX manual-run proxy route once job-based execution is live
- placeholder settings and operations components
- stale docs that describe old run endpoints as primary operator workflows
- fragmented source-specific admin handling on the frontend

**Folder cleanup requirements:**
- runner code is centralized by responsibility
- source-specific orchestration helpers move under `job_runners/` or `rollup_runners/`
- duplicated or misleading helper names are corrected
- no leftover “legacy” folders or compatibility layers remain

**No-backward-compatibility rule for this rollout:**
- old endpoints are deleted once frontend and operations UIs use the new platform
- old code branches are removed, not hidden behind flags forever
- docs are rewritten to the new architecture, not annotated with “old vs new” long-term duality

### **✅ Validation Checklist**
- [ ] No request-time sync/generation code remains in target domains.
- [ ] No placeholder admin surfaces remain.
- [ ] No deprecated run endpoint remains documented as current.
- [ ] No dead helper/service/repository code is left behind.
- [ ] Folder structure reflects ownership clearly.

---

## 8️⃣ **PHASE 8: VALIDATION, DOCUMENTATION, AND OPERATIONAL HANDOFF**
*Priority: High - prove the platform is correct, observable, and maintainable*

### **🎯 Objective**
Validate correctness across backend, frontend, scheduling behavior, dependency execution, and documentation.

### **🧪 Testing**

**Backend tests**
- unit tests for job definition parsing, health mapping, lock behavior, dependency blocking, and runner resolution
- service tests for job orchestration lifecycle
- repository tests for job/run reads and writes
- integration tests for manual run endpoints and job health reads
- domain-specific migration tests verifying removed request-time sync behavior does not regress

**Frontend tests**
- typed service parsing for job definitions and run history
- settings page admin interactions
- manual run button workflow
- operations run-history rendering
- error/blocked/degraded states

**Operational validation**
- dynamic scheduler tick selects only due jobs correctly from Supabase schedule state
- scheduled run performs source pull and downstream rollup sequence correctly
- blocked dependency surfaces clearly
- stale data is visible without triggering sync
- admin schedule changes persist and display correctly

### **📚 Documentation Updates**

| Document | Section | Change Description |
|----------|---------|-------------------|
| `docs/swainos-code-documentation-backend.md` | Architecture, scripts, job platform | Add canonical control plane, runner model, and endpoint family |
| `docs/swainos-code-documentation-frontend.md` | Settings/Operations routes | Add real job admin and run-monitoring UX |
| `docs/frontend-data-queries.md` | Job admin endpoints | Add `data-jobs` endpoints and remove deprecated run endpoints |
| `docs/sample-payloads.md` | Job payloads | Add sample job definition, health, and run detail envelopes |
| `action-plan/action-log` | Milestones | Record implementation milestones as work progresses |

### **✅ Validation Checklist**
- [ ] Backend tests for touched modules pass.
- [ ] Frontend lint/type/build for touched modules pass.
- [ ] Job dependency graph behaves correctly in real runs.
- [ ] Docs describe only the new canonical architecture.
- [ ] Action log is updated for each major milestone.

---

## ⚠️ **RISK MANAGEMENT**

### **High Priority Risks**
- **Hidden read-time mutations remain after migration**: a page still triggers sync or generation indirectly -> **Mitigation**: explicit audit of GET/service code paths and removal checklist per domain.
- **Dual-system drift**: old run endpoints continue to operate alongside new jobs -> **Mitigation**: delete old endpoints after migration and update all callers in one rollout.
- **Dependency mis-ordering**: rollups refresh before source facts are ready -> **Mitigation**: DB-backed dependency graph plus blocked-state handling and integration tests.
- **Scheduler overlap or duplicate manual runs**: two runs mutate the same domain concurrently -> **Mitigation**: lock acquisition and active-run guardrails.
- **Settings page becomes configuration-only but not operationally useful** -> **Mitigation**: require last/next run, health, and manual-run control on day one.
- **Scheduler/design mismatch**: UI allows editing schedules but runtime scheduler is static deployment config -> **Mitigation**: choose dynamic scheduler model before implementation and reflect non-editable jobs honestly in UI.

### **Medium Priority Risks**
- **Excess platform abstraction**: over-engineered generic runner model slows implementation -> **Mitigation**: keep one simple runner registry and only support current job families.
- **Doc drift during migration**: architecture changes land faster than docs -> **Mitigation**: documentation updates are mandatory before completion.
- **Naming inconsistency across job keys and UI labels** -> **Mitigation**: freeze job naming vocabulary in Phase 1 and enforce in schemas/tests/docs.

### **Rollback Strategy**
1. Freeze manual runs and schedules in admin UI.
2. Keep canonical source tables and rollups intact while temporarily pausing scheduler execution.
3. Re-enable only verified jobs in dependency order if platform issues are discovered.
4. Do not revive old request-time sync behavior; fix the platform and redeploy the canonical path.

---

## 📊 **SUCCESS CRITERIA**

### **Technical Success Metrics**

| Metric | Target | Verification Method |
|--------|--------|---------------------|
| External fetches on GET/page load | Zero | Code audit + runtime checks |
| Job platform coverage | 100% of current source/rollup workloads migrated | Job inventory vs implementation audit |
| Manual-run standardization | One canonical run contract | API/query doc audit |
| Schedule edit fidelity | 100% of schedule changes persist and affect due-job selection | Settings edit QA + scheduler integration tests |
| Run observability | Every run records lifecycle and metrics | DB query + operations UI |
| Rollup orchestration | All downstream refreshes dependency-driven | Integration tests + run history |
| Frontend admin readiness | Settings + Operations fully functional | Manual QA + build/lint |

### **User Experience Success**

| Scenario | Expected Outcome |
|----------|------------------|
| Operator opens Settings | Sees every job, schedule time, health, and manual-run control immediately |
| Operator edits a schedule | New cadence persists and displays correctly |
| Operator runs a job manually | Run appears in history with live status and final metrics |
| Scheduler tick runs | Only due jobs dispatch based on Supabase schedule state |
| Upstream source fails | Downstream rollup shows blocked/degraded instead of silently misreporting freshness |
| User opens Marketing/FX/other pages | Data loads from Supabase only with freshness metadata and no hidden sync |
| Admin investigates a failure | Operations page shows run steps, error details, dependency context, and retry path |

---

## 🔗 **RELATED DOCUMENTATION**

- **[Action Plan Template](./action-plan-template.md)** - Required planning structure
- **[FX Core Currency Plan](./13-fx-core-currency-buy-timing-framework-plan-completed.md)** - Existing source/rollup and manual-run patterns to absorb into the control plane
- **[Budget + QuickBooks Plan](./15-budget-forecast-quickbooks-operating-model-plan.md)** - Future-source ingestion planning reference
- **[SwainOS Backend Code Documentation](../docs/swainos-code-documentation-backend.md)** - Current runtime scripts, endpoints, and rollup inventory
- **[SwainOS Frontend Code Documentation](../docs/swainos-code-documentation-frontend.md)** - Current `/settings` and `/operations` placeholders plus route ownership
- **[Frontend Data Queries](../docs/frontend-data-queries.md)** - Current manual-run and source endpoint inventory to replace
- **[Sample Payloads](../docs/sample-payloads.md)** - Canonical request/response reference to update during implementation

---

## 🎯 **COMPLETION CHECKLIST**

### **Pre-Implementation**
- [ ] Inventory every current source pull, run endpoint, rollup refresh, and read-time mutation path
- [ ] Freeze job naming vocabulary and dependency graph
- [ ] Confirm Settings vs Operations page responsibilities
- [ ] Confirm which old endpoints and code paths will be deleted

### **Implementation Quality Gates**
- [ ] All new backend modules use strict `snake_case` naming and clean layering
- [ ] All new frontend modules use strict `kebab-case` / `camelCaseService.ts` / typed contracts
- [ ] No `any` types introduced
- [ ] No dead code or compatibility branches left behind
- [ ] No request-time external pulls remain

### **Testing**
- [ ] Source jobs and rollup jobs execute end-to-end successfully
- [ ] Manual-run flow works from frontend Settings page
- [ ] Operations run-history and error states are correct
- [ ] Dependency-blocked states are visible and correct
- [ ] Frontend/server read paths remain pure after migration

### **Documentation** *(MANDATORY)*
- [ ] `docs/swainos-code-documentation-backend.md` updated
- [ ] `docs/swainos-code-documentation-frontend.md` updated
- [ ] `docs/frontend-data-queries.md` updated
- [ ] `docs/sample-payloads.md` updated
- [ ] Action plan status updated to reflect implementation progress/completion

### **Final Review**
- [ ] All phases completed
- [ ] Old run endpoints removed
- [ ] Old request-time sync/generation logic removed
- [ ] Settings and Operations are real production admin surfaces
- [ ] Documentation reflects only the canonical new architecture

---

## 📝 **REVISION HISTORY**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.0 | 2026-03-09 | AI Agent + Ian | Initial comprehensive action plan for a platform-wide ingestion control plane, scheduled jobs, dependency-driven rollups, and frontend job settings/operations administration |
| v1.1 | 2026-03-09 | AI Agent + Ian | Audit pass: added manual/import ingestion jobs, maintenance/backfill workloads, FX frontend proxy cleanup, and explicit consultant-rollup to AI-insights dependency coverage |
| v1.2 | 2026-03-09 | AI Agent + Ian | Final audit pass: added scheduler source-of-truth decision, dynamic-scheduling requirement for editable job cadence, and clearer UI handling for scheduled vs manual-only jobs |
| v1.3 | 2026-03-09 | AI Agent + Ian | Locked Option 2: dynamic poller scheduler with Supabase-backed per-job schedule truth and due-job selection |
| v1.4 | 2026-03-10 | AI Agent + Ian | Marked execution complete after control-plane implementation, guardrails (`0091`, `0092`), run metrics (`0093`), Settings left-nav run logs, and final validation/doc sync |
