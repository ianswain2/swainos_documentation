# SwainOS Backend Code Documentation

Last updated: 2026-02-16

## Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Response Envelope](#response-envelope)
- [API Contract Rules](#api-contract-rules)
- [Error Envelope](#error-envelope)
- [AI-Ready Design](#ai-ready-design)
- [External Identifiers](#external-identifiers)
- [API Endpoints](#api-endpoints)
- [Itinerary Revenue Owner Cockpit API](#itinerary-revenue-owner-cockpit-api)
- [Deprecated and Removed Endpoints](#deprecated-and-removed-endpoints)
- [Itinerary Enrichment and Rollups](#itinerary-enrichment-and-rollups)
- [Travel Consultant Analytics API](#travel-consultant-analytics-api)
- [AI Insights Platform API](#ai-insights-platform-api)
- [AI Insights Migrations and RLS](#ai-insights-migrations-and-rls)
- [Operational Scripts](#operational-scripts)
- [Key Modules](#key-modules)

## Overview
SwainOS backend is a FastAPI service layered as routes -> services -> repositories. The platform is deterministic-first: business KPIs are computed from canonical rollups, and AI synthesis is persisted as auditable product data.

## Architecture
- **API Layer**: `src/api/` versioned routes and request/response envelopes.
- **Service Layer**: `src/services/` orchestration and business rules.
- **Repository Layer**: `src/repositories/` Supabase read access with explicit mapping.
- **Analytics Layer**: `src/analytics/` cashflow and booking forecast calculations.
- **Shared**: `src/shared/` response envelope and camelCase serialization.

## Response Envelope
All API responses return a standard envelope:
```json
{
  "data": {},
  "pagination": {
    "page": 1,
    "pageSize": 50,
    "totalItems": 0,
    "totalPages": 0
  },
  "meta": {
    "asOfDate": "2026-02-06",
    "source": "salesforce_kaptio",
    "timeWindow": "90d",
    "calculationVersion": "v1",
    "currency": "USD"
  }
}
```

## API Contract Rules
- Canonical payloads: `docs/sample-payloads.md`
- All list endpoints return pagination fields; single-resource endpoints return `pagination: null`.
- Query params are `snake_case`; JSON fields remain `camelCase` via shared schema config.

## Error Envelope
Errors return a consistent envelope:
```json
{
  "error": {
    "code": "bad_request",
    "message": "Unsupported time window format",
    "details": null
  }
}
```

## AI-Ready Design
- Outputs include `asOfDate`, `timeWindow`, `source`, and `calculationVersion`.
- Lineage fields are attached to normalized entities: `sourceSystem`, `sourceRecordId`, `ingestedAt`.
- Forecast outputs are deterministic for fixed datasets to support explainability.

## External Identifiers
- Canonical external identifier column is `external_id` across normalized tables.
- `Lineage.sourceRecordId` reflects the upstream `external_id` (e.g., Salesforce/Kaptio).
- `employees.analysis_disabled` is the canonical consultant-analytics opt-out flag. When `true`, the employee is excluded from consultant-focused rollups and consultant AI context, while company itinerary/revenue/forecast reporting remains unchanged.

## API Endpoints
- `GET /api/v1/health`
- `GET /api/v1/cash-flow/summary`
- `GET /api/v1/cash-flow/timeseries`
- `GET /api/v1/deposits/summary`
- `GET /api/v1/payments-out/summary`
- `GET /api/v1/booking-forecasts`
- `GET /api/v1/itinerary-trends`
- `GET /api/v1/itinerary-revenue/outlook`
- `GET /api/v1/itinerary-revenue/deposits`
- `GET /api/v1/itinerary-revenue/conversion`
- `GET /api/v1/itinerary-revenue/channels`
- `GET /api/v1/itinerary-revenue/actuals-yoy`
- `GET /api/v1/itinerary-revenue/actuals-channels`
- `GET /api/v1/fx/rates`
- `GET /api/v1/fx/exposure`
- `GET /api/v1/travel-consultants/leaderboard`
- `GET /api/v1/travel-consultants/{employee_id}/profile`
- `GET /api/v1/travel-consultants/{employee_id}/forecast`
- `GET /api/v1/ai-insights/briefing`
- `GET /api/v1/ai-insights/feed`
- `GET /api/v1/ai-insights/recommendations`
- `PATCH /api/v1/ai-insights/recommendations/{id}`
- `GET /api/v1/ai-insights/history`
- `GET /api/v1/ai-insights/entities/{entity_type}/{entity_id}`
- `POST /api/v1/ai-insights/run` (manual trigger)

## Itinerary Revenue Owner Cockpit API
- Primary endpoint family: `/api/v1/itinerary-revenue/*`
- Backing rollups:
  - `mv_itinerary_revenue_monthly`
  - `mv_itinerary_revenue_weekly`
  - `mv_itinerary_deposit_monthly`
  - `mv_itinerary_consortia_monthly`
  - `mv_itinerary_trade_agency_monthly`
  - `mv_itinerary_consortia_actuals_monthly`
  - `mv_itinerary_trade_agency_actuals_monthly`
  - `mv_itinerary_pipeline_stages` (conversion signal)
- Purpose: owner-forward outlook, deposit controls, conversion, and channel performance.
- Canonical income metric: `commissionIncomeAmount = gross_profit` (materialized in rollups as `commission_income_amount` to preserve API contracts).
- Added actuals surface: Jan-Dec year-over-year recognized revenue and productivity from `travel_end_date`.
- Added actuals channel production surface: closed-won consortia and trade agency production from `travel_end_date`.
- Exclusions: `filter_out` statuses are excluded via `itinerary_status_reference`.
- Closed-won grouping is allowlist-driven via `itinerary_status_reference` and excludes statuses outside:
  `Deposited/Confirming`, `Amendment in Progress`, `Pre-Departure`, `eDocs Sent`, `Traveling`, `Traveled`, `Cancel Fees`.
- Windowing: forward by travel period with 12-month lookback for close-ratio projection.
- Trade agency rollup classification includes `trade` and `agent` contact types with agency fallback resolution for better data coverage.

## Deprecated and Removed Endpoints
- `GET /api/v1/revenue-bookings`
- `GET /api/v1/revenue-bookings/{booking_id}`
- `GET /api/v1/itinerary-pipeline`

## Itinerary Enrichment and Rollups
- Enriched itinerary fields added for forecasting and ETL fidelity:
  - `close_date`, `trade_commission_due_date`, `trade_commission_status`, `consortia`
  - `final_payment_date`, `gross_profit`, `cost_amount`, `number_of_days`, `number_of_nights`
  - `trade_commission_amount`, `outstanding_balance`, `owner_external_id`, `lost_date`, `lost_comments`
- Status reference table: `itinerary_status_reference` for canonical status-to-pipeline classification.
- Added forecast foundations:
  - `mv_itinerary_revenue_monthly` (recognized revenue by `travel_end_date`)
  - `mv_itinerary_revenue_weekly` (weekly recognized revenue by `travel_end_date`)
  - `mv_itinerary_deposit_monthly` (deposit performance by `close_date` against 25% target)
  - `mv_itinerary_consortia_monthly` (monthly consortia performance)
  - `mv_itinerary_trade_agency_monthly` (monthly trade-agency performance)
  - `mv_itinerary_consortia_actuals_monthly` (closed-won actuals channel production by `travel_end_date`)
  - `mv_itinerary_trade_agency_actuals_monthly` (closed-won actuals trade-agency production by `travel_end_date`)
- Rollup enhancements:
  - `0025_commission_income_rollups.sql` introduced `commission_income_amount` across itinerary revenue and channel rollups.
  - `0027_commission_income_gross_profit_and_closed_won_allowlist.sql` redefines `commission_income_amount` to use `gross_profit` and enforces a strict closed-won allowlist (`Deposited/Confirming`, `Amendment in Progress`, `Pre-Departure`, `eDocs Sent`, `Traveling`, `Traveled`, `Cancel Fees`).

## Travel Consultant Analytics API
- New canonical consultant identity table: `employees`.
  - Keys and core fields: `id`, `external_id`, `first_name`, `last_name`, `email`, `salary`, `commission_rate`.
- Itinerary attribution link: `itineraries.employee_id` (resolved from `owner_external_id -> employees.external_id` during ingest/backfill).
- New materialized views:
  - `mv_travel_consultant_leaderboard_monthly` (realized travel-date production leaderboard)
  - `mv_travel_consultant_profile_monthly` (consultant profile travel outcomes)
  - `mv_travel_consultant_funnel_monthly` (lead-created to closed outcomes; includes median speed-to-book)
  - `mv_travel_consultant_compensation_monthly` (salary + commission impact rollup)
- API contracts:
  - `GET /api/v1/travel-consultants/leaderboard`
    - Query params (`snake_case`): `period_type`, `domain`, `year`, `month`, `sort_by`, `sort_order`, `currency_code`
    - Returns ranking rows + highlights for story-first leaderboard UI.
  - `GET /api/v1/travel-consultants/{employee_id}/profile`
    - Query params (`snake_case`): `period_type`, `year`, `month`, `yoy_mode`, `currency_code`
    - Returns ordered narrative sections (`heroKpis`, `trendStory`, `funnelHealth`, `forecastAndTarget`, `compensationImpact`, `signals`, `insightCards`).
  - `GET /api/v1/travel-consultants/{employee_id}/forecast`
    - Query params (`snake_case`): `horizon_months`, `currency_code`
    - Returns consultant-level forecast timeline with 12% growth target comparison.
- Story-first response contract:
  - Backend includes deterministic section ordering and comparison context (`currentPeriod`, `baselinePeriod`, `yoyMode`) to avoid frontend guesswork.
  - Payloads include signal metadata (`displayLabel`, `description`, `trendDirection`, `trendStrength`, `isLaggingIndicator`) for data-story rendering.
  - `spend_to_book` is reserved for v2 until a canonical sales/marketing spend source is finalized.

## AI Insights Platform API
- Primary endpoint family: `/api/v1/ai-insights/*`.
- Core persistence tables:
  - `ai_insight_events`
  - `ai_recommendation_queue`
  - `ai_briefings_daily`
- AI context materialized views:
  - `ai_context_command_center_v1`
  - `ai_context_travel_consultant_v1`
  - `ai_context_itinerary_health_v1`
  - `ai_context_consultant_benchmarks_v1` (team benchmark context by `period_type` and `domain`)
  - `ai_context_company_metrics_v1` (company KPI snapshot + command-center KPI joins)
- Lifecycle status contract is frozen in `snake_case`:
  - `new`, `acknowledged`, `in_progress`, `resolved`, `dismissed`
- Model routing policy:
  - Decision-critical operations (briefings, recommendations, consultant coaching) use `OPENAI_MODEL_DECISION` (default `gpt-5.2`).
  - Support operations use `OPENAI_MODEL_SUPPORT`.
  - Decision operations cannot run on support-tier unless explicitly allowed by config.
  - Model retry fallback stays on GPT-5 lineage only (`gpt-5.2`, `gpt-5.1`, `gpt-5` / `gpt-5-mini`), with deterministic fallback if model calls fail.
- Trigger mode:
  - Default is manual on-demand generation via `scripts/generate_ai_insights.py`.
  - `POST /api/v1/ai-insights/run` supports controlled manual API-trigger execution and requires `x-ai-run-token` matching `AI_MANUAL_RUN_TOKEN`.
  - `refresh_consultant_ai_rollups_v1()` must run against migrations `0043` and `0045` so benchmark/company context views are refreshed with consultant rollups.

## AI Insights Migrations and RLS
- Migrations added:
  - `0036_create_ai_insight_events.sql`
  - `0037_create_ai_recommendation_queue.sql`
  - `0038_create_ai_briefings_daily.sql`
  - `0039_create_ai_context_views.sql`
  - `0040_ai_rls_policies.sql`
  - `0041_ai_indexes.sql`
  - `0042_fix_ai_context_command_center_conversion_rate_numeric.sql`
  - `0043_create_refresh_consultant_ai_rollups_rpc.sql`
  - `0044_create_ai_benchmark_rollups.sql`
  - `0045_update_refresh_consultant_ai_rollups_rpc_with_benchmarks.sql`
  - `0046_add_employee_analysis_disabled_and_rollup_filters.sql`
- RLS pattern follows existing conventions:
  - `*_select_authenticated`
  - `*_insert_service`
  - `*_update_admin_or_service`

## Operational Scripts
- `scripts/refresh_consultant_ai_rollups.py`: calls `refresh_consultant_ai_rollups_v1()` for canonical AI + consultant context refresh.
- `scripts/generate_ai_insights.py`: manual trigger for orchestration run + persisted outputs.
- `scripts/purge_ai_insights.py`: clears AI tables for clean regeneration after major data cleanup.
- `scripts/cleanup_inactive_employees.py`: removes inactive employee records; run AI purge + rollup refresh after execution.
- After toggling `employees.analysis_disabled`, run `scripts/refresh_consultant_ai_rollups.py` so itinerary, consultant, and AI context materialized views recompute with the exclusion applied.

## Key Modules
- `src/core/config.py`: settings and environment variables.
- `src/core/errors.py`: error envelope and handlers.
- `src/core/supabase.py`: PostgREST client for Supabase.
- `src/analytics/cash_flow.py`: cash-in/out aggregation logic.
- `src/analytics/booking_forecast.py`: trend-based booking forecast.
- `src/api/itinerary_revenue.py`: owner cockpit endpoint surface.
- `src/services/itinerary_revenue_service.py`: forward outlook/deposit/conversion/channel orchestration.
- `src/repositories/itinerary_revenue_repository.py`: rollup materialized view access.
- `src/api/travel_consultants.py`: travel consultant leaderboard/profile/forecast endpoint surface.
- `src/api/ai_insights.py`: AI insights briefing/feed/recommendation/history/entity/run endpoints.
- `src/services/travel_consultants_service.py`: consultant KPI aggregation, YoY storytelling, signals, and forecast logic.
- `src/services/ai_insights_service.py`: API-facing AI insight retrieval and recommendation state transitions.
- `src/services/ai_orchestration_service.py`: AI generation orchestration from context views to persisted outputs.
- `src/services/openai_insights_service.py`: model-tier routing, strict JSON output handling, and fallback execution.
- `src/repositories/travel_consultants_repository.py`: consultant rollup materialized view access.
- `src/repositories/ai_insights_repository.py`: AI insights tables/context-view read-write operations.
- `scripts/upsert_bookings.py`: REST upsert loader for bookings.
- `scripts/upsert_itineraries.py`: REST upsert loader for enriched itineraries with external-id FK resolver for agencies/contacts/employees.
- `scripts/upsert_employees.py`: REST upsert loader for Salesforce consultant identity records.
- `scripts/upsert_itinerary_items.py`: REST upsert loader for itinerary items.
- `scripts/upsert_customer_payments.py`: REST upsert loader for customer payments.
- `scripts/generate_ai_insights.py`: manual AI insights generation trigger.
