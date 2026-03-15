# 🎯 Salesforce CORE Integration Foundation Plan - Extensible Bulk API 2.0 Ingestion and Rollup Orchestration

> **Version**: v1.1  
> **Status**: 🚀 READY TO IMPLEMENT  
> **Date**: 2026-03-15

**Target Components**: `SwianOS_Documentation/action-plan/`, `SwianOS_Documentation/docs/`, `SwainOS_BackEnd/supabase/migrations/`, `SwainOS_BackEnd/src/integrations/`, `SwainOS_BackEnd/src/services/`, `SwainOS_BackEnd/src/services/job_runners/`, `SwainOS_BackEnd/src/repositories/`, `SwainOS_BackEnd/src/schemas/`, `SwainOS_BackEnd/src/core/`, `SwainOS_BackEnd/scripts/`, `SwainOS_FrontEnd/apps/web/src/features/settings/`, `SwainOS_FrontEnd/apps/web/src/features/operations/`, `SwainOS_FrontEnd/apps/web/src/lib/api/`, `SwainOS_FrontEnd/apps/web/src/lib/types/`  
**Primary Issues**: The current Salesforce runtime sync foundation exists but does not yet cover the full source sequence, does not gate runs on org API consumption, does not model invoice-chain ingestion inside the recurring sync, and needs internal optimization so more Salesforce objects can be added without splitting into a second orchestration model. The team also does not currently have a non-production Supabase environment, which creates operational risk for Salesforce test loads.  
**Objective**: Deliver a production-ready, extensible Salesforce CORE ingestion platform by strengthening the existing `salesforce-readonly-sync` job with a custom Salesforce external client app, Bulk API 2.0 read patterns, hourly incremental extraction with intentional overlap, dependency-safe Supabase FK resolution, downstream rollup refreshes, API limit preflight protection, and a clear production-safe testing/cutover path.

## 📋 **QUICK SUMMARY**

**What We're Building/Fixing**: A durable Salesforce-to-Supabase ingestion platform where each synced object is modeled explicitly, loaded in dependency-safe sequence, refreshed every hour using Bulk API 2.0 incremental windows, and followed by deterministic rollup refresh jobs.

**Critical Issues Being Addressed**:
- Current recurring Salesforce sync stops at `itinerary_items` -> extend the recurring platform to include supplier invoices, invoice bookings, and invoice lines in the same dependency-aware operating model.
- Current sync client does not preflight org API usage -> block runs when Salesforce daily API consumption reaches the configured safety threshold before any extract begins.
- Current incremental cursor design is strong but incomplete for production hardening -> switch to window-based checkpointing with overlap, settle lag, and first-run bootstrap date of `2026-02-01T00:00:00Z`.
- Current object sync path is too monolithic for future expansion -> introduce an internal object registry and per-object execution contract inside the existing Salesforce job so more Salesforce objects can be added without redesigning the platform.
- Current testing posture risks loading sandbox data into production canonical tables -> define an official environment strategy using Supabase persistent branches when available, or a production-safe isolation fallback when they are not.
- Current post-sync chain is incomplete for AP-driven analytics -> ensure invoice-backed tables feed the existing AP/liquidity/FX surfaces and trigger downstream rollup refreshes in the correct order.

**Success Metrics**:
- Every hourly run checks Salesforce org API usage before creating any Bulk query job and blocks the run when usage is at or above the configured threshold.
- The recurring sync processes the full v1 object chain: agencies, suppliers, employees, itineraries, itinerary items, supplier invoices, supplier invoice bookings, and supplier invoice lines.
- Incremental windows use a durable `window_start/window_end` watermark model with one-hour overlap and first-run bootstrap beginning `2026-02-01T00:00:00Z`.
- All FK-dependent objects resolve against canonical Supabase IDs in the correct load sequence, with unresolved references exported and tracked for replay.
- Post-sync rollup jobs refresh successfully after upstream loads and leave no stale travel trade, consultant AI, AP, or FX exposure dependencies.
- The implementation path supports additional Salesforce objects through configuration and new transform/load handlers inside the existing Salesforce job, not a second orchestration model.

---

## 🎯 **EXECUTION STATUS**

**Progress**: 0 of 6 phases completed  
**Current Status**: Planning complete. Implementation should proceed as one coordinated backend-first rollout with explicit environment guardrails, then operational validation, then hosting cutover readiness.

| Phase | Status | Priority | Notes |
|-------|---------|----------|-------|
| 1️⃣ Environment, Access, and Safety Baseline | 📋 PENDING | HIGH | External client app, integration user, `/limits` preflight, environment strategy |
| 2️⃣ Extensible Sync Contract + Runtime Schema | 📋 PENDING | HIGH | Object registry, watermarks, unresolved tracking, run diagnostics |
| 3️⃣ Bulk API Extraction + Object Load Expansion | 📋 PENDING | HIGH | Hourly windowed extraction for all v1 objects |
| 4️⃣ Dependency Graph + Rollup Orchestration | 📋 PENDING | HIGH | Invoice-chain sequencing and downstream refresh jobs |
| 5️⃣ Validation, Testing, and Cutover Safety | 📋 PENDING | HIGH | Sandbox/prod-safe testing, data marking, purge and replay strategy |
| 6️⃣ Documentation, Operations UX, and Future Object Playbook | 📋 PENDING | MEDIUM | Docs, runbook, settings visibility, expansion contract |

---

## 🚨 **CRITICAL REQUIREMENTS**

### **⚠️ MANDATORY CHECKLIST - Must Complete Before Implementation**

- [ ] **Bulk API 2.0 Read Model**: Salesforce extraction uses Bulk API 2.0 query/queryAll for the recurring sync path.
- [ ] **Custom External Client App**: Integration authenticates through a dedicated Salesforce external client app configured for OAuth client credentials and API-only scope.
- [ ] **Dedicated Integration Identity**: The external app runs under a dedicated Salesforce integration user with least-privilege read access and explicit object/field permissions.
- [ ] **Org API Safety Gate**: Every run checks Salesforce org limits before extraction and blocks execution when daily API usage is at or above `85%`.
- [ ] **Window-Based Incremental Watermarking**: Production checkpointing uses `window_end` watermarks plus overlap; it must not rely solely on “max row seen” advancement.
- [ ] **One-Hour Overlap**: Incremental queries intentionally overlap by one hour to reduce commit-timing and pagination hole risk.
- [ ] **Bootstrap Date**: The first-ever run begins at `2026-02-01T00:00:00Z`, not “pull everything”.
- [ ] **Extensible Internal Object Registry**: Object onboarding follows a registry-driven contract inside the existing Salesforce job so new Salesforce objects can be added without reworking the scheduler model.
- [ ] **Dependency-Safe FK Resolution**: Child objects resolve Supabase UUIDs from previously loaded parent objects in the same canonical sequence.
- [ ] **Delete Awareness**: Object extracts include `IsDeleted` where supported and map deletes into SwainOS soft-delete or inactive-state semantics.
- [ ] **Singleton Execution**: Overlapping scheduled runs are blocked by both local/runtime locks and control-plane run-state guardrails.
- [ ] **No Second Salesforce Orchestration Model**: The canonical top-level recurring job remains `salesforce-readonly-sync`; optimization happens inside that job and its config/steps.
- [ ] **No Request-Time Sync**: No frontend page load or backend GET endpoint should call Salesforce directly or trigger Salesforce sync side effects.
- [ ] **Observability**: Every run records object-level counts, watermarks, blocked reasons, unresolved references, API limit snapshots, and downstream rollup outcomes.
- [ ] **Production-Safe Test Strategy**: If a non-production Supabase environment is not available, sandbox test data must never be loaded into production canonical tables without environment isolation keys and purge mechanics.
- [ ] **Documentation Update**: Backend and frontend code documentation plus Salesforce mapping docs must be updated alongside implementation.
- [ ] **Future Object Readiness**: New objects must plug into the registry, load order, diagnostics, and replay tooling without duplicating orchestration logic.

### **Documentation Update Requirement**

> **⚠️ IMPORTANT**: This rollout must update:
> - `docs/swainos-code-documentation-backend.md`
> - `docs/swainos-code-documentation-frontend.md`
> - `docs/data-mapping-user.md`
> - `docs/data-mapping-agency.md`
> - `docs/data-mapping-supplier.md`
> - `docs/data-mapping-itinerary.md`
> - `docs/data-mapping-itinerary-items.md`
> - `docs/data-mapping-supplier-invoices.md`
> - any new Salesforce runtime or operator runbook document introduced by implementation

### **External Platform Guidance Applied**

This plan intentionally aligns with current official platform guidance:

- **Salesforce Bulk API 2.0** remains the right fit for large asynchronous extracts, and Salesforce’s current guidance highlights parallel downloads as GA and partial-download events as a newer enhancement path.
- **Salesforce API monitoring guidance** explicitly recommends monitoring daily API usage, using the REST `/limits` resource, and assigning each integration a dedicated app identity and user identity for auditability.
- **Salesforce data replication guidance** reinforces that incremental extraction must account for commit-timing edge cases; this plan keeps Bulk API 2.0 as requested, but uses overlap and settle lag because those replication caveats still matter.
- **Supabase branching** is now an official product capability. Persistent branches are designed for staging/QA-style environments, are separate environments with their own credentials, and are data-less by default for safety.

This means:

- We should use **Bulk API 2.0** for the recurring data pull.
- We should **not** depend on Spring '25 Beta partial-download events for the first production rollout.
- We **should** use `/limits`, a dedicated external app, a dedicated integration user, and explicit audit identifiers.
- We **should** prefer a **Supabase persistent branch or separate staging project** before loading sandbox data; if neither is available, we must isolate test data inside production infrastructure rather than mixing it into canonical rows.

---

## 📐 **NAMING CONVENTION ALIGNMENT**

All implementation introduced by this plan must follow current SwainOS backend/frontend conventions and should standardize Salesforce-specific naming rather than invent alternate patterns.

| Element | Convention | Example |
|---------|------------|---------|
| Backend modules | `snake_case.py` | `salesforce_sync_service.py` |
| Integration modules | `snake_case.py` | `salesforce_limits_client.py` |
| Job runner keys | dot-delimited | `salesforce.object.account.sync` |
| Job keys | kebab-case text identifiers | `salesforce-account-sync` |
| Database tables | `snake_case`, plural | `salesforce_sync_watermarks` |
| Database columns | `snake_case` | `window_start`, `window_end`, `source_org_key` |
| JSON fields | `camelCase` | `windowEnd`, `blockedReason`, `recordsExtracted` |
| Frontend services | `camelCaseService.ts` | `dataJobsService.ts` |
| Type files | `lowercase.ts` | `data-jobs.ts` |
| Documentation files | `kebab-case.md` | `salesforce-core-runbook.md` |

### **Canonical Vocabulary**

- Use `Salesforce CORE integration` to describe the platform capability.
- Use `external app` or `external client app` for the Salesforce auth/application identity.
- Use `integration user` for the humanless Salesforce principal behind the client credentials flow.
- Use `watermark` to mean the durable extraction checkpoint.
- Use `windowStart/windowEnd` for incremental time slices.
- Use `overlapWindowMinutes` and `settleLagMinutes` for extraction timing guardrails.
- Use `object registry` for the configuration layer that defines how each Salesforce object is pulled and loaded.
- Use `unresolved references` for child records that could not map source external IDs to destination UUIDs.

---

## 🔧 **STRATEGIC APPROACH**

### **Implementation Philosophy**

Treat Salesforce ingestion as a platform, not a one-off script. The first production cut should solve the current object set cleanly and also establish the contract for every future Salesforce object. The core design principle is:

1. authenticate and preflight safely,
2. extract in stable windows,
3. load parents before children,
4. persist every operational signal needed for replay and diagnostics,
5. refresh deterministic downstream rollups only after upstream success,
6. expand by configuration and handlers, not by rewriting orchestration.

This avoids two common failure modes:

- a fragile monolithic sync script that becomes harder to change with every new object, and
- a pile of disconnected per-object jobs with inconsistent limits, checkpoints, and diagnostics.

### **Key Architecture Decisions**

- **Bulk API 2.0 stays the extract engine**: this is the user’s requested strategy and remains the right choice for high-volume asynchronous pulls.
- **Window-based watermarking over row-only cursoring**: keep `SystemModstamp + Id` ordering for deterministic query traversal, but checkpoint successful extraction using `windowEnd`.
- **One-hour overlap is mandatory**: overlap absorbs commit timing and “record changed during query execution” edge cases.
- **Three-to-five-minute settle lag is mandatory**: avoid extracting the most recent records still near transaction completion.
- **Internal object-step contract inside the existing platform**: split logic per object inside `salesforce-readonly-sync`, but keep one shared registry, one safety model, one observability model, and one scheduler model.
- **Accounts remain one source object with two destination loaders**: continue extracting Salesforce `Account` once, then classify rows into agencies and suppliers.
- **Invoice-chain ingestion becomes first-class recurring sync**: invoice header, booking parent, and line child loads move from manual import concepts into the recurring Salesforce chain.
- **Rollups remain explicit downstream jobs**: do not hide rollup refreshes inside sync scripts except where a script is acting as an orchestrator step runner.
- **Environment isolation is explicit**: sandbox data never lands invisibly in production canonical rows.
- **Future object onboarding is registry-driven**: adding a new object means adding config, transform logic, load handler, and dependency edges inside the existing Salesforce job, not building a new orchestration path.

### **Recommended Operating Model**

```
Salesforce External Client App
  -> OAuth client credentials
  -> Dedicated integration user
  -> Read-only object + field permissions

Preflight
  -> token exchange
  -> /limits check
  -> block if >= 85% daily API usage

Hourly scheduler tick
  -> select due `salesforce-readonly-sync` job
  -> compute windowStart/windowEnd
  -> run internal object registry in dependency order
  -> Bulk API 2.0 queryAll extract per object
  -> download results
  -> transform rows
  -> resolve parent FKs from Supabase
  -> upsert canonical tables
  -> persist watermarks, counts, unresolved refs

Dependency chain
  -> accounts -> agencies/suppliers
  -> users -> employees
  -> itineraries
  -> itinerary items
  -> supplier invoices
  -> supplier invoice bookings
  -> supplier invoice lines

Post-load
  -> travel trade rollups refresh
  -> consultant AI rollups refresh
  -> FX exposure refresh (recommended)
  -> downstream operators review runs in Settings/Operations
```

### **Why We Are Not Switching Away From Bulk API 2.0**

Salesforce’s replication guidance (`getUpdated/getDeleted`) is relevant because it highlights the risk of incremental holes near transaction boundaries, but the business requirement here is a Bulk API 2.0 based pull. The right move is:

- keep Bulk API 2.0,
- retain `queryAll`,
- order by `SystemModstamp, Id`,
- checkpoint by successful window end,
- use overlap and settle lag,
- and record blocked/partial/failure states explicitly.

That preserves the scalability of Bulk API 2.0 while respecting the edge cases the replication guidance warns about.

### **Current-State Reuse**

The current codebase already provides a strong starting point:

- `scripts/sync_salesforce_readonly.py`
- `src/integrations/salesforce_bulk_client.py`
- `src/repositories/salesforce_sync_repository.py`
- `scripts/validate_salesforce_readonly_permissions.py`
- `src/services/job_runners/registry.py`
- `scripts/upsert_agencies.py`
- `scripts/upsert_suppliers.py`
- `scripts/upsert_employees.py`
- `scripts/upsert_itineraries.py`
- `scripts/upsert_itinerary_items.py`
- `scripts/upsert_supplier_invoices.py`
- `scripts/upsert_supplier_invoice_bookings.py`
- `scripts/upsert_supplier_invoice_lines.py`
- `scripts/refresh_travel_trade_rollups.py`
- `scripts/refresh_consultant_ai_rollups.py`
- `scripts/refresh_fx_exposure.py`

This plan builds on those assets rather than replacing them with an unrelated architecture.

---

## 🚨 **CURRENT STATE FINDINGS THAT MUST BE FIXED**

### **Salesforce Runtime Sync Coverage Gap**

- The current recurring script and documentation cover:
  - agencies
  - suppliers
  - employees
  - itineraries
  - itinerary items
- The current recurring script does **not** yet include:
  - supplier invoices
  - supplier invoice bookings
  - supplier invoice lines

This is a hard gap because AP/liquidity and FX invoice pressure surfaces depend on invoice-chain tables downstream.

### **API Safety Gap**

- The current Salesforce client allowlist supports token and Bulk query endpoints only.
- There is not yet a dedicated `/limits` preflight path before starting a run.
- There is not yet a blocked-run state specifically for “unsafe to proceed because org API usage is too high”.

### **Watermark Hardening Gap**

- The current implementation stores `last_systemmodstamp + last_id`.
- That is useful, but by itself is not the safest production checkpoint model for long-lived hourly syncs with intentional overlap.
- Production should advance by successful `windowEnd`, not by “largest row seen”.

### **Environment Gap**

- There is no documented non-production Supabase environment currently in use.
- The team asked whether Supabase has a built-in way to create one. Officially, yes: Supabase now supports branching, including persistent branches designed for staging/QA-style environments.
- If that feature is not available or not adopted for this project, the team still needs a safe fallback that prevents sandbox data from polluting production canonical rows.

### **Future Object Growth Gap**

- The current implementation is closer to a purpose-built recurring script than a fully extensible object platform.
- Additional Salesforce objects will become expensive to add unless the contract is normalized now.

---

## 1️⃣ **Environment, Access, and Safety Baseline**
*Priority: High - Establish the integration identity, limits guardrails, and environment strategy before any wider sync expansion*

### **🎯 Objective**
Create the operational and security foundation for Salesforce CORE ingestion: external app, integration user, permissions, API safety preflight, and environment strategy.

### **🔍 Analysis / Discovery**

The integration should use:

- a **custom Salesforce external client app**
- **OAuth client credentials**
- **API scope only**
- a **dedicated Salesforce integration user**
- strict read-only object/field permissions

Current Salesforce guidance also recommends:

- dedicated app identity for auditability
- dedicated integration user rather than shared human credentials
- monitoring API usage using `/limits`
- and identifying calls via app identity and optional call headers

This phase also resolves the “only production Supabase exists” problem. The safest official option is now:

- **Supabase persistent branch** if available in the account and approved for use

If not available immediately, preferred fallback is:

- **a separate staging Supabase project**

If neither is possible before implementation starts, the emergency-safe fallback is:

- **production infrastructure with isolated schema + source environment keys + purge procedures**, not production canonical tables

### **⚙️ Implementation**

**Files to Create/Modify:**

| File | Action | Description |
|------|--------|-------------|
| `SwainOS_BackEnd/src/integrations/salesforce_bulk_client.py` | Modify | Add safe support for `/limits` preflight or delegate to a dedicated limits client |
| `SwainOS_BackEnd/src/integrations/salesforce_limits_client.py` | Create | Dedicated Salesforce limits reader with strict allowlist |
| `SwainOS_BackEnd/scripts/validate_salesforce_readonly_permissions.py` | Modify | Extend smoke coverage to invoice objects and limits access validation |
| `SwainOS_BackEnd/src/core/config.py` | Modify | Add configurable API threshold, overlap, settle lag, bootstrap date, and environment keys |
| `SwianOS_Documentation/docs/` | Modify/Create | Add operator runbook and environment strategy notes |

**Implementation Steps:**
1. Create or verify the Salesforce external client app configured for client credentials and API-only scope.
2. Provision a dedicated Salesforce integration user for this app with least-privilege read-only access.
3. Add a preflight step that:
   - authenticates,
   - calls `/services/data/{version}/limits`,
   - records `DailyApiRequests.max`, `remaining`, and derived usage percent,
   - blocks the run when usage is `>= 85%`.
4. Add `Sforce-Call-Options` client name tagging on API requests so event logs clearly identify SwainOS.
5. Extend validation tooling to confirm object/field access for invoice objects and any runtime permissions required by the new sync path.
6. Decide and document the environment approach:
   - preferred: persistent Supabase branch
   - fallback: separate staging project
   - emergency fallback: production-side isolated stage schema with purge procedures

### **✅ Validation Checklist**
- [ ] Salesforce external app is configured and documented
- [ ] Dedicated integration user exists and is not shared with humans
- [ ] `/limits` preflight works and returns the expected daily API metrics
- [ ] Runs block cleanly when threshold is exceeded
- [ ] Permission smoke test covers all v1 objects
- [ ] Environment strategy is chosen and documented before data testing begins

---

## 2️⃣ **Extensible Sync Contract + Runtime Schema**
*Priority: High - Normalize runtime metadata so the platform can grow without redesign*

### **🎯 Objective**
Introduce a stable internal object registry and runtime schema for watermarks, unresolved references, org limit snapshots, and replay-safe diagnostics while keeping `salesforce-readonly-sync` as the canonical top-level job.

### **🔍 Analysis / Discovery**

The current runtime state tables are helpful but too narrow for the next stage of growth. The platform now needs durable metadata that answers:

- which Salesforce objects are configured,
- how each object is extracted,
- what its parent dependencies are,
- what its current watermark is,
- whether it allows unresolved optional references,
- and what unresolved references were encountered in a given run.

This is the point where the architecture either becomes future-proof or starts accruing permanent ingestion debt.

### **⚙️ Implementation**

**Files to Create/Modify:**

| File | Action | Description |
|------|--------|-------------|
| `SwainOS_BackEnd/supabase/migrations/` | Create | Add runtime schema migration(s) for object registry/watermarks/unresolved tracking |
| `SwainOS_BackEnd/src/repositories/salesforce_sync_repository.py` | Modify | Support watermarks, preflight snapshots, and unresolved-reference persistence |
| `SwainOS_BackEnd/src/schemas/` | Create/Modify | Add typed runtime models for object configs and run diagnostics |
| `SwainOS_BackEnd/src/services/` | Create | Optional orchestration helper for object registry evaluation |

**Recommended Runtime Objects:**

| Table | Purpose |
|------|---------|
| `salesforce_sync_watermarks` | Durable `window_start`, `window_end`, `last_successful_window_end`, max row seen, overlap and lag metadata |
| `salesforce_sync_object_states` | Per-object runtime state keyed to the canonical Salesforce job: object name, dependency order, query fields, delete support, FK policy, and watermark metadata |
| `salesforce_sync_unresolved_refs` | Run-scoped unresolved external IDs for replay and reconciliation |
| `salesforce_sync_limit_snapshots` | Preflight API usage snapshots by run |
| `salesforce_sync_runs` | Existing run history expanded with blocked reasons and object-level summaries |

**Registry Fields To Freeze Early:**

- `object_name`
- `parent_job_key`
- `salesforce_api_name`
- `extract_mode`
- `supports_query_all`
- `supports_is_deleted`
- `select_fields`
- `dependency_keys`
- `destination_targets`
- `load_handler`
- `strict_fk_policy`
- `optional_fk_policy`
- `overlap_window_minutes`
- `settle_lag_minutes`
- `bootstrap_start_at`
- `enabled`

**Implementation Steps:**
1. Add runtime schema to persist object configs, watermarks, limit snapshots, and unresolved references.
2. Move extraction settings out of scattered environment-only decisions and into a typed, durable internal registry contract attached to the existing Salesforce job.
3. Preserve per-object override capability so future objects can use different overlap, lag, or unresolved policies.
4. Ensure the run model can store:
   - blocked reason
   - preflight API stats
   - object counts
   - extracted/loaded/skipped/deleted metrics
   - unresolved reference counts
   - window start/end
5. Add replay-friendly storage so unresolved rows can be retried after parent objects catch up.

### **✅ Validation Checklist**
- [ ] Runtime schema supports internal object state/config and per-object watermarks
- [ ] Blocked runs can be distinguished from failed runs
- [ ] Unresolved references are persisted by run and object
- [ ] Every v1 object can be represented in the registry without special-case schema hacks
- [ ] Future objects can be added by registry/config plus handler code

---

## 3️⃣ **Bulk API Extraction + Object Load Expansion**
*Priority: High - Build the recurring extract/load path for the full v1 object chain*

### **🎯 Objective**
Expand the recurring Salesforce sync to process the full source chain every hour using a stable windowed Bulk API 2.0 extraction model.

### **🔍 Analysis / Discovery**

The current recurring sync already shows the right pattern:

- Bulk API 2.0 `queryAll`
- `SystemModstamp + Id`
- upper-bound lag
- file lock
- canonical upsert scripts

That pattern now needs to be upgraded into a full v1 chain with formal watermarks and invoice support.

### **⚙️ Implementation**

**Files to Create/Modify:**

| File | Action | Description |
|------|--------|-------------|
| `SwainOS_BackEnd/scripts/sync_salesforce_readonly.py` | Refactor/Expand | Convert into registry-driven orchestrator and extend full object chain |
| `SwainOS_BackEnd/src/integrations/salesforce_bulk_client.py` | Modify | Support windowed query helpers and optionally parallel result download support |
| `SwainOS_BackEnd/scripts/upsert_supplier_invoices.py` | Review/Modify | Ensure recurring-safe strict upsert behavior |
| `SwainOS_BackEnd/scripts/upsert_supplier_invoice_bookings.py` | Review/Modify | Ensure recurring-safe strict FK resolution and exports |
| `SwainOS_BackEnd/scripts/upsert_supplier_invoice_lines.py` | Review/Modify | Ensure recurring-safe booking-parent resolution and replay/export handling |

**Canonical v1 Object Order:**

1. `Account` extract -> `agencies`
2. `Account` extract -> `suppliers`
3. `User` extract -> `employees`
4. `KaptioTravel__Itinerary__c` -> `itineraries`
5. `KaptioTravel__Itinerary_Item__c` -> `itinerary_items`
6. `Supplier Invoice Header Object` -> `supplier_invoices`
7. `Supplier Invoice Booking Object` -> `supplier_invoice_bookings`
8. `Supplier Invoice Line Object` -> `supplier_invoice_lines`

**Incremental Window Standard:**

- `bootstrapStartAt = 2026-02-01T00:00:00Z`
- `overlapWindowMinutes = 60`
- `settleLagMinutes = 3..5`
- `windowEnd = now_utc - settleLag`
- `windowStart = lastSuccessfulWindowEnd - overlapWindow`
- query condition:
  - `SystemModstamp >= windowStart`
  - `SystemModstamp < windowEnd`
  - `ORDER BY SystemModstamp, Id`

**Why This Is The Production Standard**

- intentional overlap catches commit-timing holes,
- stable ordering ensures deterministic page traversal,
- successful window end is a replay-safe checkpoint,
- upsert-by-`external_id` keeps the overlap idempotent.

**Parallel Downloads Guidance**

Salesforce now supports parallel result downloads for Bulk API 2.0 query results. For v1:

- support serial result download first if implementation risk is lower,
- add a clean abstraction so parallel downloads can be enabled without redesign,
- do not rely on partial-result eventing for first release because it is a newer, optional path and increases rollout complexity.

**Implementation Steps:**
1. Replace object hard-coding in the main sync script with an internal registry-driven execution loop.
2. Compute and persist `windowStart/windowEnd` per object on every run.
3. Continue extracting `Account` once while producing two destination datasets.
4. Add recurring extract/load support for invoice headers, invoice bookings, and invoice lines.
5. Keep strict parent-first resolution rules:
   - itinerary items require itineraries/suppliers
   - invoice bookings require itineraries/suppliers
   - invoice lines require booking-parent resolution and optional related-entity tolerance only where explicitly allowed
6. Export unresolved references per run for later replay.
7. Persist row counts and deletion metrics by object.

### **✅ Validation Checklist**
- [ ] Hourly run uses the full v1 object chain
- [ ] First-run bootstrap starts from `2026-02-01T00:00:00Z`
- [ ] Window checkpoints persist correctly after success
- [ ] Repeated overlap runs are idempotent
- [ ] Invoice-chain loads run under the recurring sync path
- [ ] Object-level counts and unresolved exports are recorded

---

## 4️⃣ **Dependency Graph + Rollup Orchestration**
*Priority: High - Ensure all downstream analytics surfaces refresh only after the canonical Salesforce load chain succeeds*

### **🎯 Objective**
Fold the expanded Salesforce chain into the existing `salesforce-readonly-sync` job and keep downstream rollup dependencies aligned to the current data-jobs model.

### **🔍 Analysis / Discovery**

The codebase already has:

- `salesforce.readonly.sync`
- `salesforce.travel_trade.rollups.refresh`
- `salesforce.consultant_ai.rollups.refresh`

The missing piece is aligning the expanded invoice chain and AP/FX downstreams with this dependency model. Invoice tables feed:

- `ap_open_liability_v1`
- `ap_summary_v1`
- `ap_aging_v1`
- `ap_payment_calendar_v1`
- `ap_pressure_30_60_90_v1`

Those, in turn, influence AP surfaces, cash-flow views, and FX invoice pressure/exposure semantics.

### **⚙️ Implementation**

**Files to Create/Modify:**

| File | Action | Description |
|------|--------|-------------|
| `SwainOS_BackEnd/src/services/job_runners/registry.py` | Modify | Keep `salesforce.readonly.sync` as the canonical runner and extend its internal config/behavior |
| `SwainOS_BackEnd/supabase/migrations/` | Create | Seed/update `data_jobs` and `data_job_dependencies` for the final graph |
| `SwainOS_BackEnd/scripts/refresh_travel_trade_rollups.py` | Review | Confirm it fits the expanded dependency chain |
| `SwainOS_BackEnd/scripts/refresh_consultant_ai_rollups.py` | Review | Confirm it fits the expanded dependency chain |
| `SwainOS_BackEnd/scripts/refresh_fx_exposure.py` | Review | Add as recommended downstream dependency where appropriate |

**Recommended Job Graph**

Canonical platform shape:

1. `salesforce-readonly-sync`
2. `travel-trade-rollups-refresh`
3. `consultant-ai-rollups-refresh`
4. `fx-exposure-refresh` (recommended)

Inside `salesforce-readonly-sync`, the internal object-step sequence should be:

1. `Account` -> agencies/suppliers
2. `User` -> employees
3. `Itinerary` -> itineraries
4. `Itinerary Item` -> itinerary items
5. `Supplier Invoice` -> supplier invoices
6. `Supplier Invoice Booking` -> supplier invoice bookings
7. `Supplier Invoice Line` -> supplier invoice lines

**Recommendation**

Keep **one canonical Salesforce job** and optimize it internally. The benefits are:

- alignment with the existing control-plane inventory and Settings/Operations model,
- no second job vocabulary to seed, migrate, and document,
- simpler operator mental model,
- preservation of the current `salesforce.readonly.sync` runner contract,
- and a cleaner path to future growth through internal registry/config steps rather than a parallel orchestration system.

### **✅ Validation Checklist**
- [ ] Dependency graph reflects the real parent/child object chain
- [ ] Rollup jobs do not run unless upstream object loads succeed
- [ ] FX exposure refresh is included where invoice changes materially affect downstream analytics
- [ ] Operators can see which internal object step or downstream rollup blocked the chain
- [ ] The graph remains extensible for future Salesforce objects

---

## 5️⃣ **Validation, Testing, and Cutover Safety**
*Priority: High - Define how to test Salesforce sandbox data safely without damaging production canonical state*

### **🎯 Objective**
Create a safe test-and-cutover path that works whether the team gets a Supabase staging environment or must temporarily validate against production infrastructure.

### **🔍 Analysis / Discovery**

The team currently does not have a non-production Supabase environment in active use. Officially, Supabase now supports branching and persistent branches for staging/QA-style environments, but those branches are data-less by default. That matters because:

- schema testing is easy,
- ingestion-logic testing is possible,
- but realistic data testing still requires planned seed/masking/import strategy.

For this project, the safest testing priority is:

1. use a Supabase persistent branch if available,
2. otherwise create a second Supabase project,
3. only if neither is possible, use production infrastructure with strict isolation.

### **🧪 Testing Strategy**

**Preferred: Supabase Persistent Branch**

- Create a persistent branch for Salesforce CORE testing.
- Apply migrations and run the full sync there.
- Seed only controlled sample/reference data where needed.
- Use sandbox Salesforce app credentials against the branch environment.
- Validate counts, FK resolution, rollups, and replay behavior.

**Fallback: Separate Staging Project**

- Create a dedicated staging project.
- Push all migrations there.
- Use sandbox Salesforce credentials and the same control-plane/job setup.
- Validate end-to-end before touching production.

**Emergency Fallback: Production Infrastructure Isolation**

If only production infrastructure is available, then sandbox testing must not write into canonical production tables directly. Use all of the following:

- separate schema, for example `sf_stage`
- `source_org_key`
- `source_env`
- `test_run_id`
- `ingested_at`
- purge stored procedure(s)
- feature flags so frontend/backend read paths never point at stage schema by accident

**Production-Only Safety Rules**

- Never mix sandbox and production source rows under the same uniqueness contract.
- Do not rely on “we can tell later which rows are test rows” as an informal process.
- If canonical tables must ever hold both environments, uniqueness must include source environment, for example `(source_org_key, external_id)`.
- Any production-only testing must have a documented purge plan before the first run starts.

**Purge Strategy**

If testing in isolated production-side stage schema:

1. stop schedulers,
2. disable the Salesforce job family,
3. purge by `source_env = 'sandbox'` and `test_run_id`,
4. clear stage-schema watermarks and unresolved records,
5. re-enable only after verification.

### **Cutover Sequence**

1. Validate external app, integration user, and `/limits` preflight.
2. Validate object permission smoke tests for all v1 objects.
3. Run sandbox load in non-production environment if available.
4. Reconcile unresolved references and replay until stable.
5. Validate downstream rollup outputs and AP/FX surfaces.
6. Freeze config values for production bootstrap start, overlap, lag, and schedule.
7. Run first production bootstrap from `2026-02-01T00:00:00Z`.
8. Confirm first successful production watermark advancement.
9. Enable hourly recurring schedule.

### **✅ Validation Checklist**
- [ ] Non-production environment path is selected or explicitly waived with written risk acceptance
- [ ] Sandbox test data can be purged cleanly
- [ ] Production canonical rows cannot be polluted by sandbox loads accidentally
- [ ] Replay flow for unresolved references has been tested
- [ ] First production bootstrap checklist is documented

---

## 6️⃣ **Documentation, Operations UX, and Future Object Playbook**
*Priority: Medium - Finish the rollout with durable operator guidance and an explicit expansion contract*

### **🎯 Objective**
Document the final operating model, expose the right operational fields in the jobs UI, and define how new Salesforce objects are added safely.

### **📚 Documentation Updates**

**Required Documentation Changes:**

| Document | Section | Change Description |
|----------|---------|-------------------|
| `docs/swainos-code-documentation-backend.md` | Salesforce ingestion and data-jobs sections | Update object list, watermark model, preflight limits gate, invoice-chain sync, and rollup dependencies |
| `docs/swainos-code-documentation-frontend.md` | Settings/Operations sections | Document operator visibility for Salesforce job health, blocked runs, and rollup chain status |
| `docs/data-mapping-user.md` | Sync scope | Confirm recurring sync ownership and any new user-field assumptions |
| `docs/data-mapping-agency.md` | Sync scope | Confirm account classification and recurring load semantics |
| `docs/data-mapping-supplier.md` | Sync scope | Confirm account classification and recurring load semantics |
| `docs/data-mapping-itinerary.md` | Sync semantics | Add hourly watermark model and dependency notes |
| `docs/data-mapping-itinerary-items.md` | Resolver policy | Confirm unresolved export/replay semantics |
| `docs/data-mapping-supplier-invoices.md` | Operational sequence | Promote invoice chain into recurring Salesforce sync |
| `docs/salesforce-core-runbook.md` | New doc | Operator runbook for preflight, blocked runs, replay, purge, and cutover |

### **Operations UX Requirements**

Settings/Operations should expose:

- current job enabled status
- cadence
- last run status
- blocked reason
- latest preflight API usage percent
- latest object counts
- unresolved reference counts
- current watermark window
- downstream rollup outcomes

### **Future Object Onboarding Playbook**

Every new Salesforce object added after v1 must define:

1. Salesforce API name
2. select field set
3. source technical fields
4. delete semantics
5. destination tables
6. transform handler
7. FK dependencies
8. unresolved-reference policy
9. validation queries
10. downstream rollup impacts
11. operator metrics
12. documentation updates

This keeps future growth predictable inside the existing Salesforce job rather than introducing a second orchestration surface.

### **✅ Validation Checklist**
- [ ] Operator UI exposes blocked runs and watermarks clearly
- [ ] Documentation matches the final implemented object chain
- [ ] A runbook exists for preflight, replay, purge, and cutover
- [ ] Future object onboarding checklist is published and reusable

---

## ⚠️ **RISK MANAGEMENT**

### **High Priority Risks**

- **API limit exhaustion during business hours**: An hourly integration can contribute to org-wide API pressure.  
  **Mitigation**: Hard `/limits` preflight, blocked-run status, API usage notifications in Salesforce, and capacity review before production cutover.

- **Incremental holes caused by commit timing or moving result boundaries**: Records updated near query execution boundaries can be missed if checkpoints advance too aggressively.  
  **Mitigation**: Window-based watermarking, one-hour overlap, settle lag, deterministic ordering by `SystemModstamp, Id`, and idempotent upserts.

- **Sandbox data contaminates production analytics**: Testing without environment isolation can pollute production fact tables and rollups.  
  **Mitigation**: Prefer Supabase persistent branch or staging project. If impossible, isolate in separate schema with `source_env`, `source_org_key`, and purge mechanics.

- **Child-object FK resolution fails at scale**: Itinerary items or invoice lines can arrive before parents are available or fully loaded.  
  **Mitigation**: Parent-first dependency graph, strict resolver policies, unresolved export tracking, replay tooling, and blocked downstream rollups when required parents are incomplete.

- **Future object growth breaks orchestration simplicity**: Adding more objects can turn the sync into a tangle of custom branches.  
  **Mitigation**: Freeze a registry-driven contract now and require every new object to conform to it.

### **Medium Priority Risks**

- **Over-engineering around new Bulk API features**: Partial downloads and event-driven result consumption are attractive but add rollout complexity.  
  **Mitigation**: Use standard Bulk API 2.0 polling and completed-job downloads for v1; keep future enhancement path open but defer beta/event-driven behavior.

- **Operational ambiguity between blocked, failed, and partial runs**: Operators may misinterpret system state without clear run semantics.  
  **Mitigation**: Distinct run statuses, explicit blocked reasons, and object-level metrics surfaced in Settings/Operations.

- **Invoice-chain rollout changes downstream AP/FX expectations**: Once recurring sync owns invoice tables, downstream data freshness becomes more visible.  
  **Mitigation**: Validate AP and FX surfaces during cutover and explicitly include `fx-exposure-refresh` in the dependency plan.

### **Rollback Strategy**

1. Disable recurring Salesforce jobs in the control plane.
2. Stop scheduler-triggered dispatch for the Salesforce family.
3. Preserve existing canonical production data while reverting new Salesforce runner config or newly introduced object jobs.
4. If sandbox/test data was loaded in isolated paths, purge by environment/test-run identifiers or drop isolated stage schema.
5. Restore previous documented runtime schedule only after confirming no active Salesforce runs remain.
6. Verify downstream travel trade, consultant AI, AP, and FX surfaces are reading from known-good state.

---

## 📊 **SUCCESS CRITERIA**

### **Technical Success Metrics**

| Metric | Target | Verification Method |
|--------|--------|---------------------|
| Preflight API safety gate | 100% of runs check limits before extraction | Run logs and `salesforce_sync_limit_snapshots` |
| Incremental watermark durability | No gap in successful hourly watermark advancement | Watermark table audit |
| Full v1 object coverage | All 8 v1 destination object stages run under recurring sync | Job inventory and run history |
| FK resolution quality | No required-parent unresolved rows left after replay window | Unresolved reference reports |
| Rollup freshness | Travel trade, consultant AI, and FX exposure refresh after successful source loads | Downstream run logs |
| Replay capability | Unresolved references can be reprocessed without destructive resets | Manual replay validation |

### **Operational Success**

| Scenario | Expected Outcome |
|----------|------------------|
| Org API usage is below threshold | Run proceeds and records preflight metrics |
| Org API usage is at or above threshold | Run is blocked, not failed, with visible blocked reason |
| First production bootstrap starts | Extraction begins from `2026-02-01T00:00:00Z` |
| Hourly run repeats within overlap window | Upserts remain idempotent and no duplicates are created |
| Parent object lands before child replay | Previously unresolved child rows can be retried and resolved |
| New Salesforce object is introduced later | Team adds an internal registry entry, transform/load handler, dependencies, and docs without redesigning the platform |

---

## 🔗 **RELATED DOCUMENTATION**

- **[Platform Ingestion Control Plane and Job Operations Plan](./16-platform-ingestion-control-plane-and-job-operations-plan-completed.md)** - Canonical job-platform architecture already implemented
- **[Platform Ingestion Job Inventory v1](./16a-platform-ingestion-job-inventory-v1-completed.md)** - Existing schedulable job inventory and dependency framing
- **`docs/swainos-code-documentation-backend.md`** - Backend runtime, Salesforce sync, rollup, and data-jobs documentation
- **`docs/swainos-code-documentation-frontend.md`** - Settings/Operations frontend contract and operator visibility
- **`docs/data-mapping-user.md`** - Employee/consultant mapping
- **`docs/data-mapping-agency.md`** - Agency/account mapping
- **`docs/data-mapping-supplier.md`** - Supplier/account mapping
- **`docs/data-mapping-itinerary.md`** - Itinerary mapping
- **`docs/data-mapping-itinerary-items.md`** - Itinerary item mapping and resolver rules
- **`docs/data-mapping-supplier-invoices.md`** - Invoice header/booking/line mapping and strict resolver rules

---

## 📚 **TECHNICAL REFERENCE**

### **v1 Object Registry Reference**

| Registry Key | Salesforce Object | Destination Targets | Depends On | Notes |
|-------------|------------------|---------------------|------------|-------|
| `account` | `Account` | `agencies`, `suppliers` | - | One extract, two classified destination loaders |
| `user` | `User` | `employees` | - | Owner resolution source for itineraries |
| `itinerary` | `KaptioTravel__Itinerary__c` | `itineraries` | `account`, `user` | Resolves agencies and employees |
| `itinerary_item` | `KaptioTravel__Itinerary_Item__c` | `itinerary_items` | `itinerary`, `account` | Resolves itinerary + supplier |
| `supplier_invoice` | `SupplierInvoiceHeaderObject` | `supplier_invoices` | `account` | Exact object API name to be confirmed in Salesforce metadata |
| `supplier_invoice_booking` | `SupplierInvoiceBookingObject` | `supplier_invoice_bookings` | `supplier_invoice`, `itinerary`, `account` | Resolves itinerary + supplier |
| `supplier_invoice_line` | `SupplierInvoiceLineObject` | `supplier_invoice_lines` | `supplier_invoice_booking`, `itinerary_item`, `itinerary`, `account` | Booking parent resolution is required |

### **Recommended Configuration Constants**

```python
SALESFORCE_API_USAGE_THRESHOLD_PERCENT = 85
SALESFORCE_BOOTSTRAP_START_AT = "2026-02-01T00:00:00Z"
SALESFORCE_OVERLAP_WINDOW_MINUTES = 60
SALESFORCE_SETTLE_LAG_MINUTES = 5
SALESFORCE_DEFAULT_POLL_INTERVAL_SECONDS = 5
SALESFORCE_DEFAULT_MAX_POLLS_PER_JOB = 120
SALESFORCE_MAX_RESULT_PAGES_PER_JOB = 200
```

### **Canonical Incremental Query Shape**

```sql
SELECT Id, SystemModstamp, IsDeleted, ...
FROM SomeObject__c
WHERE SystemModstamp >= :windowStart
  AND SystemModstamp < :windowEnd
ORDER BY SystemModstamp, Id
```

### **Run Status Contract**

| Status | Meaning |
|--------|---------|
| `success` | Preflight passed, extraction/load completed, watermark advanced |
| `failed` | Technical failure or data failure prevented completion |
| `blocked` | Run intentionally did not start because of threshold, lock, dependency, or safety gate |
| `partial` | Allowed only when policy explicitly permits object-level partial completion and the run records unresolved impacts clearly |

### **Production-Safe Test Isolation Minimum**

```text
source_org_key: identifies sandbox vs production Salesforce org
source_env: sandbox | production
test_run_id: groups all rows written by a given test campaign
ingested_at: import timestamp
```

If testing cannot occur in a separate Supabase environment, these identifiers and a purge path become mandatory.

---

## 🎯 **COMPLETION CHECKLIST**

### **Pre-Implementation**
- [ ] Review all existing Salesforce sync, upsert, and rollup scripts
- [ ] Confirm exact Salesforce API names for supplier invoice objects
- [ ] Confirm external app and integration user permissions
- [ ] Select the non-production or isolation strategy before testing starts

### **Implementation Quality Gates**
- [ ] `/limits` preflight is live and blocks unsafe runs
- [ ] Watermark model uses `windowStart/windowEnd` with overlap
- [ ] Full v1 object chain runs under recurring sync
- [ ] Invoice chain is part of recurring orchestration, not manual-only thinking
- [ ] All required unresolved-reference exports and replay hooks exist
- [ ] Rollup dependency chain is explicit and observable
- [ ] No request-time Salesforce calls remain in read paths

### **Testing**
- [ ] Permission smoke test passes for all v1 objects
- [ ] Sandbox or isolated environment test load completes successfully
- [ ] Parent/child replay validation is complete
- [ ] AP, cash-flow, travel trade, consultant AI, and FX downstream surfaces validate successfully
- [ ] First production bootstrap checklist is executed and signed off

### **Documentation** *(MANDATORY)*
- [ ] `docs/swainos-code-documentation-backend.md` updated
- [ ] `docs/swainos-code-documentation-frontend.md` updated
- [ ] All Salesforce mapping docs updated
- [ ] Operator runbook added or updated
- [ ] Action plan status updated to ✅ COMPLETED when rollout finishes

### **Final Review**
- [ ] All phases completed
- [ ] All validation checklists passed
- [ ] Environment strategy remains accurate
- [ ] Future object onboarding contract is documented and reusable

---

## 📝 **REVISION HISTORY**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.1 | 2026-03-15 | GPT-5.4 / Cursor | Refined plan to preserve `salesforce-readonly-sync` as the canonical top-level job and move extensibility into internal object registry/step optimization rather than a second orchestration model |
| v1.0 | 2026-03-15 | GPT-5.4 / Cursor | Initial action plan for Salesforce CORE integration foundation, Bulk API 2.0 sync expansion, API safety preflight, environment strategy, and future-object extensibility |

