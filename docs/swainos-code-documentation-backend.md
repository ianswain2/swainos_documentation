# SwainOS Backend Code Documentation

## Overview

SwainOS backend is a FastAPI service with route → service → repository layering.  
Focus: deterministic analytics contracts, traceable rollups, and API envelopes consumed by the Next.js app.

## Production Hosting Topology
- Canonical backend hostname: `api.swainos.com`
- Expected primary frontend caller: `app.swainos.com`
- Public edge and DNS authority: Cloudflare
- Backend should be exposed through the canonical API hostname rather than raw origin URLs
- **Code vs live parity:** `main` may be deployed to Render later than Supabase migrations are applied, or the Render service may be suspended/offline. Booking-pace and other MV-backed routes require migrations `0109`–`0111` (and successful refresh) on the target database before responses match this documentation.

## Architecture
- `src/api`: route surfaces and query parsing
- `src/services`: business logic and aggregation orchestration
- `src/repositories`: Supabase/PostgREST access
- `src/schemas`: typed request/response contracts
- `src/core`: config, error handling, Supabase client, structured logging, request context
- `src/shared`: response envelope and time helpers
- `scripts`: ingestion/upsert and refresh workflows
- Analytics repository read standard: `SupabaseClient.select_all_pages(...)` with deterministic composite `order` clauses for full-result pagination safety (avoid silent Supabase row-cap truncation and unstable offset traversal on ties)

## API Envelope
All successful responses use:

```json
{
  "data": {},
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-26",
    "source": "view_or_table_names",
    "timeWindow": "12m",
    "calculationVersion": "v1",
    "currency": "USD"
  }
}
```

Error envelope:

```json
{
  "error": {
    "code": "bad_request",
    "message": "Unsupported time window format",
    "details": {
      "requestId": "f9f2f6ca-1d6b-49a7-9848-a0c7a5c2d322"
    }
  }
}
```

## Active Endpoint Families
- Health: `/health`, `/healthz`, `/health/ready`
- Auth and access control: `/auth/me`, `/settings/user-access`, `/settings/user-access/{user_id}`, `/settings/user-access/{user_id}/deactivate`, `/settings/user-access/{user_id}/reactivate`
- AP liquidity: `/ap/summary|aging|payment-calendar`
- Cash flow risk suite: `/cash-flow/summary|timeseries|risk-overview|forecast|ap-schedule|ap-monthly-outflow|scenarios`
- Dashboard snapshots (SSR bundles, optional server-side cache): `/dashboard-snapshots/command-center`, `/dashboard-snapshots/cash-flow`
- Command center backing reads are assembled in snapshot orchestration from itinerary revenue + consultant services (`bookingsSnapshot`, `travelRevenueSummary`, `topOpenItineraries`, `topConsultants`) and AI briefing; the web client consumes only `/dashboard-snapshots/command-center`.
- Debt service: `/debt-service/overview|facilities|schedule|payments|covenants|scenarios|scenarios/run`
- Data jobs control plane: `/data-jobs`, `/data-jobs/run-feed`, `/data-jobs/{job_key}`, `/data-jobs/{job_key}/runs`, `/data-jobs/health`, `/data-jobs/scheduler/tick`, `/data-job-runs/{run_id}`
- Revenue bookings: `/revenue-bookings`, `/revenue-bookings/{booking_id}`
- Itinerary revenue: `/itinerary-revenue/outlook|conversion|booked-revenue-yoy|actuals-yoy|actuals-channels|actuals-channels-comparison` (`booked-revenue-yoy` reads company-month close-date serving view `vw_semantic_booked_revenue_company_monthly_v2` sourced from semantic booked-revenue facts; `actuals-channels-comparison` reads month-grain booking-pace serving views; deposit timeline and standalone channel ranking routes were removed from the public API; DB may still retain `mv_itinerary_deposit_monthly` for jobs)
- Itinerary lead flow: `/itinerary-lead-flow`
- Travel consultant: `/travel-consultants/leaderboard|{employee_id}/profile|{employee_id}/forecast`
- Travel trade: `/travel-agents/*`, `/travel-agencies/*`, `/travel-trade/search`
- Supplier analytics: `/suppliers/leaderboard|profiles|{supplier_id}/profile`
- FX: `/fx/rates|exposure|signals|transactions|holdings|intelligence|invoice-pressure`
- Marketing web analytics: `/marketing/web-analytics/overview|search|search-console|search-console/page-profile|ai-insights|page-activity|geo|events|health`
- AI insights: `/ai-insights/briefing|feed|recommendations|history|entities/*|run`
- Manual run utility endpoints: `/fx/signals/run`, `/ai-insights/run` (strict token-gated, non-UI primary paths)

## Authentication and Authorization
- Auth verification boundary: `src/core/auth.py` verifies Supabase bearer tokens against `/auth/v1/user` before route access.
- Current-user access endpoint: `GET /api/v1/auth/me` returns role, active status, and permission keys.
- **Password sign-in is not a FastAPI route.** The Next.js app performs Supabase password authentication via same-origin `POST /api/auth/login` (see frontend code documentation), which sets browser session cookies. This backend never receives raw passwords for that flow; it only validates issued access tokens on subsequent `Authorization: Bearer …` API calls.
- Admin management endpoints: `src/api/settings_user_access.py` provides list/get/update/deactivate/reactivate flows for user access.
- Access service and repository:
  - `src/services/auth_access_service.py`
  - `src/repositories/auth_access_repository.py`
- Route-level authorization uses `src/api/authz.py` dependencies (`require_permission`, `require_admin`, `require_marketing_permission`).
- Permission checks are layered: route dependency checks + data-level RLS policies + frontend route/nav filtering.
- In-process API rate limiting (`src/api/rate_limits.py`, `src/core/rate_limit.py`) applies to selected **expensive run** and **mutation** endpoints (e.g. manual AI/FX runs, data-jobs scheduler tick), not to browser password sign-in. Distributed login abuse and direct hits to Supabase Auth endpoints remain governed by Supabase project settings and edge/WAF controls.
- Auto-bootstrap safeguards:
  - first authenticated access auto-creates missing `user_profiles` row as active `member`
  - admin list sync ensures auth users missing profile rows are inserted as active `member`
  - default bootstrap does not grant admin role or module permissions

## Data and Rollup Model
- Canonical Gross Profit contract key: `grossProfitAmount` (source column: `itineraries.gross_profit`)
- Status classification: `itinerary_status_reference` (`pipeline_bucket`, `pipeline_category`, `is_filter_out`)
- Itinerary revenue rollups remain keyed by travel period for actuals/outlook, while booked-revenue YoY uses a dedicated close-date semantic rollup path
- Supplier production analytics are sourced from `mv_supplier_travel_revenue_monthly_v1` (fact basis: `itinerary_items`; hierarchy enrichment via `supplier_items.location_id -> locations` with itinerary-item location fallback; current-year travel surfaces are cut to `current_date` for true YTD). Supplier-level distinct itinerary counts read from `mv_supplier_itinerary_fact_v1` via RPCs `supplier_distinct_itinerary_counts_v1` and `supplier_monthly_distinct_itinerary_counts_v1` so KPI counts are not derived by summing location-bucket distincts.
- Supplier profile KPI and all-time relationship summary fetch distinct-itinerary-count facts only for windows that are rendered (`current` and `all-time`); no unused prior-window distinct-count query is executed.
- `refresh_itinerary_revenue_rollups_v1()` refreshes itinerary-facing revenue/deposit/channel MVs; supplier-facing refresh work runs through `refresh_supplier_revenue_rollups_v1()` for supplier core (`mv_supplier_itinerary_fact_v1`, `mv_supplier_travel_revenue_monthly_v1`) and `refresh_supplier_service_type_revenue_rollups_v1()` for `mv_supplier_service_type_revenue_monthly_v1`, keeping the Salesforce downstream job graph below RPC gateway timeout pressure. Booking-pace comparisons read semantic v2 serving views refreshed by `refresh_semantic_rollups_v2()`
- `/itinerary-revenue/conversion` **observed** metrics (open quoted, confirmed, lost, pipeline total, `observed_close_ratio`, gross splits by stage class) are computed in Supabase view `itinerary_pipeline_conversion_monthly_v1`, which aggregates `mv_itinerary_pipeline_stages` in SQL. The service layer only applies **projections** (scenario close rates × open pipeline, gross-profit yield from `mv_itinerary_revenue_monthly`) and the lookback blend with revenue outlook buckets—no re-aggregation of stage rows in Python for the timeline.
- Consultant and company AI context is materialized and refreshed via `refresh_consultant_ai_rollups_v1()`
- Travel trade analytics reads from semantic v2 serving views refreshed via `refresh_semantic_rollups_v2()`
- FX exposure is refreshed via `refresh_fx_exposure_v1()`
- AP/liquidity canonical rollups are sourced from `supplier_invoice_lines` + `supplier_invoice_bookings`:
  - `ap_open_liability_v1`
  - `ap_summary_v1`
  - `ap_aging_v1`
  - `ap_payment_calendar_v1`
  - `ap_pressure_30_60_90_v1`
- FX invoice pressure endpoint reads from AP pressure rollups (`ap_pressure_30_60_90_v1`) instead of header-only supplier invoice totals.
- `/cash-flow/summary|timeseries` uses customer payments + AP payment-calendar rows for historical/net liquidity slices.
- `/cash-flow/risk-overview|forecast|ap-schedule|scenarios` uses forward AP schedule rows plus projected inflow baseline from trailing customer payment history to flag upcoming cash stress by currency.
- `/itinerary-trends` and `/itinerary-lead-flow` return explicit `503` error envelopes on repository/query failures (no silent zero-data fallback).
- `/payments-out/summary` reports AP line-based outstanding and near-term due pressure.
- `/marketing/web-analytics/*` is GA4 + Supabase snapshot-first; the canonical Google ingest job refreshes both GA4 and Search Console snapshots, and Search Console analytics requires `GOOGLE_GSC_SITE_URL` plus service-account access.

## Debt Service Domain
- Debt facilities are data-driven rows in `debt_facilities`; no loan constants are embedded in service code.
- Terms are effective-dated in `debt_facility_terms` so future loans and term revisions do not require code changes.
- Current seeded obligations are split as separate facilities (Citizens SBA term loan, Seller Note 1, Seller Note 2 equity standby) rather than a combined seller loan row.
- Projected schedule rows live in `debt_payment_schedule`; posted events live in `debt_payments_actual`.
- Balance snapshots in `debt_balance_snapshots` power fast KPI reads and auditability.
- Scenario simulation is isolated in `debt_scenarios` and `debt_scenario_events` and never mutates baseline ledgers.
- Core formulas are deterministic: fixed-rate monthly PI allocation, balance rollforward, extra-principal payoff delta.
- Rate interpretation is explicit using `rate_unit` (`decimal` or `percent`) with validation in schema and service normalization.
- Facility terms are protected against overlapping effective windows using `btree_gist` exclusion constraints.
- Payment posting blocks backdated entries in the live posting path to protect snapshot chronology.

## Core Materialized Views
- `mv_itinerary_pipeline_stages` (itinerary pipeline by travel month and derived stage; refreshed with itinerary revenue rollup jobs)
- `mv_itinerary_revenue_monthly`
- `mv_itinerary_revenue_weekly`
- `mv_itinerary_deposit_monthly` (retained for operational/refresh paths; not exposed via the itinerary-revenue HTTP contract)
- `mv_itinerary_consortia_monthly`
- `mv_itinerary_trade_agency_monthly`
- `mv_itinerary_consortia_actuals_monthly`
- `mv_itinerary_trade_agency_actuals_monthly`
- `mv_supplier_travel_revenue_monthly_v1`
- `mv_supplier_itinerary_fact_v1` (supplier distinct-itinerary-count fact grain; refreshed with supplier rollup jobs)
- `mv_itinerary_lead_flow_monthly`
- `mv_travel_consultant_leaderboard_monthly`
- `mv_travel_consultant_profile_monthly`
- `mv_travel_consultant_funnel_monthly`
- `mv_travel_consultant_compensation_monthly`
- `mv_travel_trade_search_v2` (DB-native trade search index sourced from active travel-agent/travel-agency tables and refreshed by `refresh_semantic_rollups_v2()`)
- **Booking pace (semantic v2 serving views):** `vw_channel_consortia_booking_pace_monthly_v2`, `vw_channel_trade_agency_booking_pace_monthly_v2`, `vw_travel_agent_booking_pace_monthly_v2`, `vw_travel_consultant_booking_pace_monthly_v2` (served from `vw_semantic_booking_pace_fact_closed_lifecycle_v2`, which extends the baseline fact with `closed_active`)

## Core analytics views (standard views on rollups)
- `itinerary_pipeline_conversion_monthly_v1` — monthly conversion / pipeline snapshot derived from `mv_itinerary_pipeline_stages` (`observed_close_ratio` = confirmed path ÷ (open quoted + confirmed + lost))

## AI Context Views
- `ai_context_command_center_v1`
- `ai_context_travel_consultant_v1`
- `ai_context_itinerary_health_v1`
- `ai_context_consultant_benchmarks_v1`
- `ai_context_company_metrics_v1`

## Operational Scripts
- `scripts/upsert_itineraries.py`
- `scripts/upsert_itinerary_items.py`
- `scripts/upsert_supplier_items.py`
- `scripts/resolve_itinerary_item_supplier_item_links.py`
- `scripts/upsert_employees.py`
- `scripts/upsert_customer_payments.py`
- `scripts/upsert_agencies.py`
- `scripts/upsert_suppliers.py`
- `scripts/sync_salesforce_data_ingestion.py`
- `scripts/validate_salesforce_data_ingestion_permissions.py`
- `scripts/refresh_consultant_ai_rollups.py`
- `scripts/pull_fx_rates.py`
- `scripts/generate_fx_intelligence.py`
- `scripts/refresh_fx_exposure.py`
- `scripts/generate_ai_insights.py` (manual runner that calls `AiInsightsService.run_manual_generation`)
- `scripts/sync_marketing_web_analytics.py` (canonical unified Google ingest runner for GA4 + Search Console snapshot refresh)
- Project-root bootstrapped operational scripts share `src/core/env_file.load_env_file` for `.env` parsing, and their default `--env-file` resolution points to repository-root `.env` rather than caller cwd-relative behavior.
- Upsert ingestion scripts with identical batching semantics share `scripts/batching_helpers.py` (`chunk_rows`, `chunk_values`) with import fallbacks that preserve both direct CLI execution and module-import test contexts (including `upsert_itinerary_names.py` and the core CRM upsert scripts).
- Upsert ingestion scripts in the bounded standardization cluster share `scripts/env_helpers.py` for `.env` parsing (including locations/suppliers/employees/agencies/customer_payments/bookings/itinerary_names and supplier-invoice/item upsert scripts), with matching fallback import behavior so CLI usage and import-based test harnesses resolve the helper consistently.
- `scripts/env_helpers.py` delegates to `src/core/env_file.load_env_file` and exposes the superset parser options (`strip_key_whitespace`, `strip_wrapping_quotes`) needed by script-specific import flows, so `.env` parsing logic has one implementation source.
- Additional operational scripts (`sync_salesforce_readonly.py`, `validate_salesforce_readonly_permissions.py`, `resolve_location_lookups.py`, `cleanup_inactive_employees.py`, `resolve_itinerary_item_supplier_item_links.py`, rollup refresh runners, and AI context purge/refresh runners) use the same `scripts/env_helpers.py` contract while preserving direct CLI execution.

## Script and Code Standardization Rules
- Keep `.env` parsing in shared helpers (`src/core/env_file.py` for project-root scripts, `scripts/env_helpers.py` for script-cluster imports); avoid inline parser copies.
- Keep row batching in `scripts/batching_helpers.py`; avoid script-local `chunk_rows` / `chunk_values` duplicates.
- For scripts that must run both as direct CLI files and as import targets in tests, use dual-path imports (`scripts.*` first, fallback to local module after `sys.path` insertion).
- Keep `--help` bootstrap safe: parse args and load env before runtime-only service imports; defer optional heavy dependencies to execution paths.
- Preserve behavior-specific parsing through explicit helper options (for example `strip_wrapping_quotes`) instead of introducing one-off script forks.
- Apply route → service → repository layering for API behavior; keep script logic operational and side-effect scoped.
- Treat this document as a contract snapshot: describe system behavior in present tense and avoid migration-status phrasing.

## Data Jobs Control Plane
- Runtime API routes:
  - `src/api/data_jobs.py`
  - `src/api/data_job_runs.py`
- Core orchestration:
  - `src/services/data_job_service.py`
  - `src/services/data_job_run_executor.py`
  - `src/repositories/data_job_repository.py`
  - `src/services/job_runners/*`
- Data-jobs execution boundary:
  - `DataJobService` owns job lookup, dependency/active-run gating, scheduler dispatch decisions, and delegation.
  - `data_job_run_executor.execute_job_run` owns run lifecycle execution (run row creation, step/progress persistence, checkpoint emission, runner invocation, terminal updates, and post-success dependent dispatch callback execution).
- Supabase control-plane objects (migration `0090_create_data_jobs_control_plane_v1.sql`):
  - `public.data_jobs`
  - `public.data_job_dependencies`
  - `public.data_job_runs`
  - `public.data_job_run_steps`
  - `public.data_job_run_checkpoints`
  - `public.data_job_health_v1`
- Additional control-plane guardrail migrations:
  - `0091_data_job_run_guardrails_v1.sql`:
    - enforces one `running` row per job via partial unique index
    - auto-closes legacy duplicate `running` rows before index creation
  - `0092_data_job_retry_backoff_v1.sql`:
    - adds per-job `retry_backoff_minutes`
    - enforces valid range (`0..10080`)
    - seeds recurring default backoff (`30`) and non-recurring default (`0`)
  - `0093_data_job_run_metrics_v1.sql`:
    - adds persisted run analytics fields on `data_job_runs`: `duration_seconds`, `output_size_bytes`
    - backfills historical rows from `started_at`/`finished_at` and serialized `output` payload size
  - `0149_create_data_job_run_checkpoints.sql`:
    - adds append-only run checkpoint timeline table `data_job_run_checkpoints` with ordered sequence per run
    - persists durable stage transitions (`started`, `exported`, `load_started`, `completed`, `failed`, `skipped`)
    - applies control-plane-aligned RLS policy (`data_job_run_checkpoints_admin_manage`)
  - `0151_constrain_data_job_run_checkpoint_status.sql`:
    - adds DB-level status constraint for `data_job_run_checkpoints.status`
    - keeps checkpoint schema aligned with the typed API contract and prevents malformed operator/runtime writes
- Scheduler model:
  - one fixed scheduler tick endpoint (`POST /api/v1/data-jobs/scheduler/tick`)
  - scheduler tick is machine-authenticated by `x-scheduler-token` and does not depend on human permission JWTs
  - due-job selection reads `data_jobs.next_run_at`
  - dependency blocking and run-state locks are applied in service orchestration
  - optional dependencies (`required=false`) are not blocking; stale gating honors `allow_stale_dependency`
  - dependency freshness is resolved from the latest successful dependency run
  - scheduler loop isolates per-job failures and continues dispatching later due jobs
  - stale `running` rows are auto-failed after `max_runtime_seconds`
  - scheduler-triggered blocked runs advance `next_run_at` to prevent repeated due-job churn
  - failed recurring jobs honor per-job retry cooldown before re-dispatch
  - high-cost run and mutation paths are request-rate limited (app-layer + edge-layer)

## Marketing Web Analytics (GA4-First)
- Runtime API route: `src/api/marketing_web_analytics.py`
- Service orchestration: `src/services/marketing_web_analytics_service.py`
- Search Console collaborator: `src/services/marketing_search_console_service.py`
- Persistence adapter: `src/repositories/marketing_web_analytics_repository.py`
- GA4 integration client: `src/integrations/google_analytics_client.py`
- Search Console integration client: `src/integrations/google_search_console_client.py`
- Runtime state and snapshot tables:
  - `public.marketing_web_analytics_daily`
  - `public.marketing_web_analytics_channels_daily`
  - `public.marketing_web_analytics_country_daily`
  - `public.marketing_web_analytics_landing_pages_daily`
  - `public.marketing_web_analytics_events_daily`
  - `public.marketing_web_analytics_page_activity_daily`
  - `public.marketing_web_analytics_geo_daily`
  - `public.marketing_web_analytics_demographics_daily`
  - `public.marketing_web_analytics_devices_daily`
  - `public.marketing_web_analytics_internal_search_daily`
  - `public.marketing_web_analytics_overview_period_summaries`
  - `public.marketing_web_analytics_sync_runs`
  - `public.marketing_search_console_daily`
  - `public.marketing_search_console_query_daily`
  - `public.marketing_search_console_page_daily`
  - `public.marketing_search_console_page_query_daily`
  - `public.marketing_search_console_country_daily`
  - `public.marketing_search_console_device_daily`
  - migrations:
    - `SwainOS_BackEnd/supabase/migrations/0077_create_marketing_web_analytics_runtime_tables.sql`
    - `SwainOS_BackEnd/supabase/migrations/0078_expand_marketing_web_analytics_dimension_storage.sql`
    - `SwainOS_BackEnd/supabase/migrations/0079_add_marketing_page_activity_and_geo_breakdowns.sql`
    - `SwainOS_BackEnd/supabase/migrations/0086_harden_marketing_analytics_canonical_facts.sql`
    - `SwainOS_BackEnd/supabase/migrations/0087_create_search_console_analytics_tables.sql`
    - `SwainOS_BackEnd/supabase/migrations/0099_allow_marketing_sync_partial_status_v1.sql`
- Search Console ingestion is active and persisted in Supabase canonical facts for query/page/country/device analysis.
- Search Console sync-window resolution, snapshot upsert orchestration, insights rollup shaping, and page-profile shaping are owned by `MarketingSearchConsoleService`; `MarketingWebAnalyticsService` delegates those responsibilities while keeping API contracts unchanged.
- Shared pure helpers for marketing country-scope normalization, Search Console date parsing, branded-query detection, decimal coercion, and safe-rate math live in `src/services/marketing_web_analytics_helpers.py`; both services import the same implementation so helper behavior does not drift between GA4 and Search Console code paths.
- Search Console Supabase rollups:
  - `marketing_search_console_insights_rollup_v1` (baseline workspace rollup)
  - `marketing_search_console_us_workspace_v1` (US-first workspace rollup)
  - `marketing_search_console_page_profile_v1` (single-page profile rollup)
- Deep-dive endpoints:
  - `/marketing/web-analytics/page-activity` for best/worst pages, itinerary-page filtering, quality scoring, and dedicated lookbook/destination page slices via deterministic path-contains classification
  - `/marketing/web-analytics/geo` for country/region/city performance plus audience demographics (`userAgeBracket`, `userGender`) and device-category breakdowns
  - `/marketing/web-analytics/events` for event catalog definitions and conversion classification
- `/marketing/web-analytics/search` includes source/medium mix, referral source leaders, value-ranked sources, and explicit traffic-quality signals (`bounceRate`, `qualifiedSessionRate`, `qualityLabel`) in addition to landing pages and internal site-search demand
  - `/marketing/web-analytics/search-console` serves a US-first Search Console workspace: overview, top queries/pages, country+device mix, market benchmarks, query intent buckets, position-band summaries, and deterministic opportunities/challenges/issues from Supabase rollups (with controlled live refresh when stale)
  - `/marketing/web-analytics/search-console/page-profile` serves single-page URL drill-down data (overview, daily trend, top queries, market benchmarks, diagnostics, recommended actions) from Supabase rollups
- `/marketing/web-analytics/ai-insights` returns structured action-engine output (`category`, `focusArea`, `targetLabel`, `ownerHint`, `impactScore`, `confidenceScore`) for direct marketer/sales/AI consumption
- Scope contract:
  - GA4-backed read endpoints accept optional `country` query param.
  - `country=all` (or omitted) keeps snapshot-first `All markets` behavior for GA4-backed surfaces.
  - `country=United States` (and other non-`all` values) uses exact GA4 country-filtered reads for overview/search/page-activity/geo/events/AI composition so scoped metrics do not drift from global marts.
  - Search Console Insights routes are intentionally US-first and do not accept a market selector parameter.
  - Response meta includes `marketScope` and `marketLabel` to make backend-applied scope explicit to clients.
- Quality/consolidation notes:
- Daily/channel/country storage persists canonical per-day facts (`date+channel`, `date+country`) and is overwrite-safe for repeated same-day sync runs.
  - Historical GA4 daily/channel/country snapshot refresh is incremental: first bootstrap can backfill ~800 days, then later runs refresh only a bounded catch-up window (`max(14 days, staleness + 3 days)` per scope) instead of replaying the full history each run.
  - Search Console snapshot refresh is also incremental under the same unified ingest job: first bootstrap backfills up to 365 days, then later runs refresh only a bounded catch-up window (`max(14 days, staleness + 3 days)` per scope).
  - Overview KPI windows (`current_30d`, `previous_30d`, `year_ago_30d`, `today`, `yesterday`) are synced into `marketing_web_analytics_overview_period_summaries` so frontend reads do not recompute distinct users by summing daily rows.
  - Page-activity ingestion is stored at GA4 `pagePath` grain and avoids segmented-row merges that can distort non-additive metrics.
  - Large page-activity and geo writes are chunked in repository upserts to keep sync reliability stable at higher row counts.
  - `/marketing/web-analytics/overview` returns an extended trend window (up to ~800 days) from Supabase snapshots so frontend YoY visualizations do not require live GA4 calls.
- Rolling 7/30/90-day marketing/search snapshots and Search Console opportunity/challenge inputs are recomputed for each run-date as-of value rather than historical replay, so suggestion surfaces stay current without scanning the full source history on every run.
  - Demographics/device/internal-search enrichment is snapshotted during sync for 30-day reads; custom day ranges still use exact same-window GA4 pulls.
- Optional section failures (demographics/devices/internal-search/Search Console) are recorded as `partial` sync runs with section details in `error_message` instead of silently reporting full `success`.
- AI insight generation applies ruthless marketing heuristics across landing pages, channels, device mix, geo quality, internal site search, destination demand, and content-removal candidates.

## Salesforce Data Ingestion
- Runtime orchestrator: `scripts/sync_salesforce_data_ingestion.py`
- Permission smoke check: `scripts/validate_salesforce_data_ingestion_permissions.py`
- Client guardrails live in: `src/integrations/salesforce_bulk_client.py`
- Runtime state storage:
  - `public.salesforce_sync_cursors`
  - `public.salesforce_sync_runs`
  - migrations:
    - `SwainOS_BackEnd/supabase/migrations/0076_create_salesforce_sync_runtime_tables.sql`
    - `SwainOS_BackEnd/supabase/migrations/0103_salesforce_sync_runtime_and_dependency_hardening.sql`

### External Client App Configuration
- App name: `SwainOS`
- OAuth enabled: yes
- OAuth scopes: `Manage user data via APIs (api)` only
- Flow enablement:
  - Client Credentials: enabled
  - Authorization Code + Credentials: disabled
  - Device Flow: disabled
  - JWT Bearer Flow: disabled
  - Token Exchange Flow: disabled
- Non-required app types disabled:
  - SAML/Web App
  - Canvas
  - Mobile
  - Push Notifications
- Callback URL is placeholder-only for this flow (for example `https://localhost/callback`)

### Permission Model and Scope
- One-way pull only: Salesforce -> SwainOS.
- Integration principal is read-only for scoped objects and fields.
- Required technical fields include: `Id`, `SystemModstamp`, `LastModifiedDate`, `IsDeleted` (where supported).
- Business fields are sourced from:
  - `docs/data-mapping-agency.md`
  - `docs/data-mapping-supplier.md`
  - `docs/data-mapping-user.md`
  - `docs/data-mapping-itinerary.md`
  - `docs/data-mapping-item.md`
  - `docs/data-mapping-itinerary-items.md`
  - `docs/data-mapping-supplier-invoices.md`

### Sync Semantics
- Scheduler cadence: hourly (`0 * * * *`).
- Limits preflight: read `/limits` before extraction and block the run when `DailyApiRequests` usage is `>= 85%`.
- Window watermarking:
  - bootstrap start: `2026-02-01T00:00:00Z`
  - overlap: `60` minutes
  - settle lag: `5` minutes
- Deterministic cursor state is still recorded as `SystemModstamp + Id` per object, with `last_completed_upper_bound` tracking the last finished extraction window.
- Extract mode: Bulk API 2.0 `queryAll`.
- Delete handling: map Salesforce `IsDeleted` into destination soft-delete behavior.
- Validated Salesforce object API names in sandbox:
  - agencies/suppliers source: `Account`
  - employees source: `User` (active mapping intentionally excludes unavailable sandbox fields `Salary__c` and `Commission_Rate__c`)
  - itineraries source: `KaptioTravel__Itinerary__c`
  - supplier items source: `KaptioTravel__Item__c`
  - itinerary items source: `KaptioTravel__Itinerary_Item__c`
  - supplier invoice chain: `KaptioTravel__SupplierInvoice__c`, `KaptioTravel__SupplierInvoiceBooking__c`, `KaptioTravel__SupplierInvoiceLine__c`
- Load order per run:
  1. agencies
  2. suppliers
  3. employees
  4. supplier_items
  5. itineraries
  6. itinerary_items
  7. itinerary-item supplier-item resolver (`resolve_itinerary_item_supplier_item_links.py`) only when the current incremental run extracted itinerary-item rows
  8. supplier_invoices
  9. supplier_invoice_bookings
  10. supplier_invoice_lines
- Unresolved-reference policy:
  - itinerary items export unresolved itinerary/supplier references for retry while skipping unresolved strict rows
  - itinerary-item supplier-item resolver emits `with_external`, `with_fk`, `unresolved`, and `unresolved_pct` counters when invoked from the incremental run
  - historical itinerary-item supplier-item reconciliation is handled by dedicated backfill job `itinerary-item-supplier-links-backfill`, which wraps DB function `backfill_itinerary_item_supplier_links_v1()`
  - supplier invoice headers/bookings preserve optional unresolved related references and export them for backfill
  - supplier invoice lines require booking-parent resolution and may preserve optional unresolved itinerary/item/supplier references for later reconciliation

### API Safety and Failure Behavior
- Singleton lock prevents overlapping scheduled runs.
- Conservative polling and per-run API budgets are enforced.
- No automatic retries; failures stop current run and continue next scheduled interval.
- Endpoint allowlist blocks non-Bulk/token/limits Salesforce API paths.
- Loop-prevention design is first-principles and deterministic:
  - each run computes one `upperBound` and never chases in-run writes
  - each object window start is derived from `last_completed_upper_bound - overlap` (or bootstrap) rather than from transient partial progress
  - blocked/failed runs do not auto-retry inside the same process and therefore cannot spin in a tight Salesforce ping loop
- Run observability is persisted and emitted:
  - object-level metrics include `extracted`, `staged`, `loaded`, unresolved/duplicate skip counts, `csv_bytes`, and `window_start`
  - counters include `jobsCreated`, `pollsMade`, and `resultPagesRead`
  - terminal JSON status payloads (`success`/`blocked`) are parsed into `data_job_runs.output.parsed` for operator inspection
  - active `data_job_run_steps.output` stores throttled live progress snapshots (`latestProgress`, bounded `progressEvents`, bounded `recentLogs`) on a best-effort basis so observability failure does not fail the job itself
  - durable stage timeline events are persisted in `data_job_run_checkpoints` and returned from `GET /api/v1/data-job-runs/{run_id}` as `checkpoints`
- Successful `salesforce-data-ingestion-sync` runs fan out dependent `system_managed` rollups:
  - `itinerary-revenue-rollups-refresh`
  - `supplier-revenue-rollups-refresh`
  - `supplier-service-type-rollups-refresh`
  - `semantic-rollups-v2-refresh`
  - `consultant-ai-rollups-refresh`
  - `fx-exposure-refresh`

## Contract Rules
- Query params remain `snake_case`
- JSON payload fields remain `camelCase`
- New terms and metric labels follow `docs/swainos-terminology-glossary.md`
- No compatibility shims: active contracts are represented directly and refactored when needed
- AI insights require live model execution (no deterministic fallback contract)
- Travel agent and travel agency leaderboard/profile `period_type=year` windows are full calendar year (`Jan 1` through `Dec 31`) for the selected year.
- Travel consultant leaderboard/profile: **travel** rollups use itinerary **`travel_start_date`** month and closed lifecycle (`closed_won` + `closed_active`). **Booked** close-date matrices and profile `ytdVariancePct` use booking-pace / close-date serving views with same-month cutoff rules where implemented.
- Booking-pace contracts use `travel_start_date` as the cohort year/month and `close_date` month <= as-of month as the inclusion rule. Current-year comparisons use current-month cutoff; prior-year comparisons use the same month in the prior year. Closed-lifecycle booking-pace serving views include both `closed_won` and `closed_active`.
- Travel consultant leaderboard `travelYtdVariancePct` compares PYTD **travel revenue** (travel-start basis from `vw_travel_consultant_travel_monthly_v2`). Profile `ytdVariancePct` remains **booking-pace booked revenue** (close-date cohort cutoffs). Travel-trade profile YoY series use booking-pace semantics. Itinerary actuals YoY reads travel-start closed-lifecycle (`closed_won` + `closed_active`) and includes next travel year when data exists.
- Frontend declutter (route header removal, Travel Agencies KPI-card removal, removal of client-side period toggles in favor of route-level fixed queries) is UI/routing-only; backend contracts stay explicit query-param driven for automation and future toggles.

## Semantic Rollup v2 (live read paths)
- Semantic split is active for travel-trade leaderboards/profiles, consultant **travel** leaderboard rows (`vw_travel_consultant_travel_monthly_v2`) vs **booked** close-date matrices (`vw_travel_consultant_booked_revenue_monthly_v2`), funnel comparators, and itinerary channel comparison booking-pace reads.
- Canonical v2 fact materialized views:
  - `mv_semantic_lead_fact_monthly_v2` (`created_at` month basis)
  - `mv_semantic_booked_fact_monthly_v2` (`travel_start_date` month basis, closed-won)
  - `mv_semantic_booking_pace_fact_monthly_v2` (`travel_start_date` month + `close_date` booked month, closed-won baseline fact)
  - `mv_semantic_booked_revenue_fact_monthly_v2` (dedicated close-date booked-revenue fact; closed lifecycle includes `closed_won` + `closed_active`)
- Booked revenue company-month serving view:
  - `vw_semantic_booked_revenue_company_monthly_v2` (pre-aggregated month totals consumed by `/itinerary-revenue/booked-revenue-yoy`)
- Closed-lifecycle booking-pace helper view:
  - `vw_semantic_booking_pace_fact_closed_lifecycle_v2` (unions baseline booking-pace fact with `closed_active` close-date bookings)
- Search coverage also uses the v2 runtime path: `mv_travel_trade_search_v2` is refreshed alongside semantic v2 rollups.
- Serving views (`vw_*_v2`) provide domain-focused slices for agents, agencies, consultants, and channel tables without mixing semantic bases in one row.
- Consultant-serving coverage includes:
  - `vw_travel_consultant_profile_monthly_v2` (travel-start; columns include `travel_revenue_amount`, `travel_gross_profit_amount` and legacy `booked_revenue_amount` / `gross_profit_amount` aliases where defined in migrations)
  - `vw_travel_consultant_travel_monthly_v2` (canonical travel-monthly slice for leaderboard **travel** metrics; supersedes app usage of retired `vw_travel_consultant_booked_monthly_v2`)
  - `vw_travel_consultant_compensation_monthly_v2`
  - `vw_travel_consultant_lead_monthly_v2`
  - `vw_travel_consultant_booked_revenue_monthly_v2` (close-date booked revenue + `booked_gross_profit_amount`)
  - `vw_travel_consultant_booking_pace_monthly_v2` (booking-pace; includes `booked_gross_profit_amount`)
- Travel-agent consultant affinity reads from `vw_travel_agent_consultant_affinity_monthly_v2`.
- New refresh RPC: `refresh_semantic_rollups_v2()`; wired as a parallel system-managed data job (`semantic-rollups-v2-refresh`).
- Parity diagnostic view: `vw_semantic_rollup_v2_parity_checks` (legacy closed-won baseline checks for totals/top-10/PYTD pace sanity); booked-revenue close-date diagnostics are exposed via `vw_semantic_booked_revenue_v2_checks`.
- Baseline freeze utility script: `scripts/capture_semantic_rollup_baseline_v1.py` snapshots v2 serving views to a v2 baseline artifact for parity validation.

## Runtime Security and Cost Guardrails
- Production startup requires non-empty values for:
  - `AI_MANUAL_RUN_TOKEN`
  - `FX_MANUAL_RUN_TOKEN`
  - `DATA_JOBS_SCHEDULER_TOKEN`
- Non-production behavior: manual/scheduler token headers are only enforced when a token is configured; unconfigured local/dev environments are allowed for developer workflows.
- Production request handling enforces trusted host allowlists before route execution.
- Subprocess-backed data jobs are bounded by `max_runtime_seconds`; timed-out jobs are force-killed and marked with `runner_timeout_killed`.
- Subprocess job output handling streams from process stdout and keeps only bounded tail output in persisted run metadata; no unused in-memory stdout accumulator state is retained in the runner path.
- CLI operational scripts should remain `--help` safe before runtime-only service initialization; runtime-only dependencies should load in execution paths, not block argument parsing/bootstrap.
- AI generation enforces run-level budgets (`max model calls`, `max tokens`, `max consultants`) and returns partial-safe status when budget limits are hit.
- Application-layer rate limits are in-memory and process-local; Cloudflare edge rate limiting remains the global cross-instance enforcement layer.

## Related documentation

- [Frontend code documentation](swainos-code-documentation-frontend.md) — Next.js structure, loaders, UX notes
- [Frontend data queries](frontend-data-queries.md) — which app paths call which `/api/v1/*` routes
- [Sample payloads](sample-payloads.md) — JSON examples
- [Terminology glossary](swainos-terminology-glossary.md) — display terms and field mapping
- [Render guidelines](render-guidelines.md) — FastAPI hosting, health checks, resume-after-suspend
- [Cloudflare guidelines](cloudflare-guidelines.md) — API proxy vs DNS-only, edge expectations
