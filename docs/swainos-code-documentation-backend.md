# SwainOS Backend Code Documentation

## Overview
SwainOS backend is a FastAPI service with route -> service -> repository layering.  
Primary focus: deterministic analytics contracts, traceable rollups, and API envelopes used by frontend modules.

## Architecture
- `src/api`: route surfaces and query parsing
- `src/services`: business logic and aggregation orchestration
- `src/repositories`: Supabase/PostgREST access
- `src/schemas`: typed request/response contracts
- `src/core`: config, error handling, Supabase client, structured logging, request context
- `src/shared`: response envelope and time helpers
- `scripts`: ingestion/upsert and refresh workflows

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
- AP liquidity: `/ap/summary|aging|payment-calendar`
- Cash flow risk suite: `/cash-flow/summary|timeseries|risk-overview|forecast|ap-schedule|scenarios`
- Command center core: `/deposits/*`, `/payments-out/*`, `/booking-forecasts`, `/itinerary-trends`
- Debt service: `/debt-service/overview|facilities|schedule|payments|covenants|scenarios|scenarios/run`
- Revenue bookings: `/revenue-bookings`, `/revenue-bookings/{booking_id}`
- Itinerary revenue: `/itinerary-revenue/outlook|deposits|conversion|channels|actuals-yoy|actuals-channels`
- Itinerary lead flow: `/itinerary-lead-flow`
- Travel consultant: `/travel-consultants/leaderboard|{employee_id}/profile|{employee_id}/forecast`
- Travel trade: `/travel-agents/*`, `/travel-agencies/*`, `/travel-trade/search`
- FX: `/fx/rates|exposure|signals|transactions|holdings|intelligence|invoice-pressure` and run endpoints
- Marketing web analytics: `/marketing/web-analytics/overview|search|search-console|ai-insights|page-activity|geo|events|health|sync/run`
- AI insights: `/ai-insights/briefing|feed|recommendations|history|entities/*|run`

## Data and Rollup Model
- Canonical Gross Profit contract key: `grossProfitAmount` (source column: `itineraries.gross_profit`)
- Status classification: `itinerary_status_reference` (`pipeline_bucket`, `pipeline_category`, `is_filter_out`)
- Itinerary revenue rollups are keyed by travel period
- Consultant and company AI context is materialized and refreshed via `refresh_consultant_ai_rollups_v1()`
- Travel trade analytics is refreshed via `refresh_travel_trade_rollups_v1()`
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
- `/payments-out/summary` reports AP line-based outstanding and near-term due pressure.
- `/marketing/web-analytics/*` is GA4-first in v1; Search Console query-level analytics is enabled once `GOOGLE_GSC_SITE_URL` is configured.

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
- `mv_itinerary_revenue_monthly`
- `mv_itinerary_revenue_weekly`
- `mv_itinerary_deposit_monthly`
- `mv_itinerary_consortia_monthly`
- `mv_itinerary_trade_agency_monthly`
- `mv_itinerary_consortia_actuals_monthly`
- `mv_itinerary_trade_agency_actuals_monthly`
- `mv_itinerary_lead_flow_monthly`
- `mv_travel_consultant_leaderboard_monthly`
- `mv_travel_consultant_profile_monthly`
- `mv_travel_consultant_funnel_monthly`
- `mv_travel_consultant_compensation_monthly`

## AI Context Views
- `ai_context_command_center_v1`
- `ai_context_travel_consultant_v1`
- `ai_context_itinerary_health_v1`
- `ai_context_consultant_benchmarks_v1`
- `ai_context_company_metrics_v1`

## Operational Scripts
- `scripts/upsert_itineraries.py`
- `scripts/upsert_itinerary_items.py`
- `scripts/upsert_employees.py`
- `scripts/upsert_customer_payments.py`
- `scripts/upsert_agencies.py`
- `scripts/upsert_suppliers.py`
- `scripts/sync_salesforce_readonly.py`
- `scripts/validate_salesforce_readonly_permissions.py`
- `scripts/refresh_consultant_ai_rollups.py`
- `scripts/refresh_travel_trade_rollups.py`
- `scripts/pull_fx_rates.py`
- `scripts/generate_fx_intelligence.py`
- `scripts/refresh_fx_exposure.py`
- `scripts/generate_ai_insights.py`
- `scripts/sync_marketing_web_analytics.py` (GA4 runtime sync path)

## Marketing Web Analytics (GA4-First)
- Runtime API route: `src/api/marketing_web_analytics.py`
- Service orchestration: `src/services/marketing_web_analytics_service.py`
- Persistence adapter: `src/repositories/marketing_web_analytics_repository.py`
- GA4 integration client: `src/integrations/google_analytics_client.py`
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
  - migrations:
    - `supabase/migrations/0077_create_marketing_web_analytics_runtime_tables.sql`
    - `supabase/migrations/0078_expand_marketing_web_analytics_dimension_storage.sql`
    - `supabase/migrations/0079_add_marketing_page_activity_and_geo_breakdowns.sql`
    - `supabase/migrations/0086_harden_marketing_analytics_canonical_facts.sql`
- Search Console is optional/deferred in v1; `/marketing/web-analytics/search-console` surfaces partial status when not connected.
- Deep-dive endpoints:
  - `/marketing/web-analytics/page-activity` for best/worst pages, itinerary-page filtering, quality scoring, and dedicated lookbook/destination page slices via deterministic path-contains classification
  - `/marketing/web-analytics/geo` for country/region/city performance plus audience demographics (`userAgeBracket`, `userGender`) and device-category breakdowns
  - `/marketing/web-analytics/events` for event catalog definitions and conversion classification
  - `/marketing/web-analytics/search` now includes source/medium mix, referral source leaders, value-ranked sources, and explicit traffic-quality signals (`bounceRate`, `qualifiedSessionRate`, `qualityLabel`) in addition to landing pages and internal site-search demand
  - `/marketing/web-analytics/search-console` provides Search Console connection status plus SEO proxy analytics (`organicLandingPages`, `internalSiteSearchTerms`) while query-level GSC ingestion remains deferred
  - `/marketing/web-analytics/ai-insights` now returns structured action-engine output (`category`, `focusArea`, `targetLabel`, `ownerHint`, `impactScore`, `confidenceScore`) for direct marketer/sales/AI consumption
- Scope contract:
  - All read endpoints accept optional `country` query param.
  - `country=all` (or omitted) keeps existing snapshot-first `All markets` behavior.
  - `country=United States` (and other non-`all` values) uses exact GA4 country-filtered reads for overview/search/page-activity/geo/events/AI composition so scoped metrics do not drift from global marts.
  - Response meta now includes `marketScope` and `marketLabel` to make backend-applied scope explicit to clients.
- Quality/consolidation notes:
  - Daily/channel/country storage now persists canonical per-day facts (`date+channel`, `date+country`) and is overwrite-safe for repeated same-day sync runs.
  - Overview KPI windows (`current_30d`, `previous_30d`, `year_ago_30d`, `today`, `yesterday`) are synced into `marketing_web_analytics_overview_period_summaries` so frontend reads do not recompute distinct users by summing daily rows.
  - Page-activity ingestion is stored at GA4 `pagePath` grain and avoids segmented-row merges that can distort non-additive metrics.
  - Large page-activity and geo writes are chunked in repository upserts to keep sync reliability stable at higher row counts.
  - `/marketing/web-analytics/overview` returns an extended trend window (up to ~800 days) from Supabase snapshots so frontend YoY visualizations do not require live GA4 calls.
  - Demographics/device/internal-search enrichment is snapshotted during sync for 30-day reads; custom day ranges still use exact same-window GA4 pulls.
  - AI insight generation now applies ruthless marketing heuristics across landing pages, channels, device mix, geo quality, internal site search, destination demand, and content-removal candidates.

## Salesforce Read-Only Ingestion
- Runtime orchestrator: `scripts/sync_salesforce_readonly.py`
- Permission smoke check: `scripts/validate_salesforce_readonly_permissions.py`
- Client guardrails live in: `src/integrations/salesforce_bulk_client.py`
- Runtime state storage:
  - `public.salesforce_sync_cursors`
  - `public.salesforce_sync_runs`
  - migration: `supabase/migrations/0076_create_salesforce_sync_runtime_tables.sql`

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
  - `docs/data-mapping-itinerary-items.md`

### Sync Semantics
- Incremental cursor: `SystemModstamp + Id` tie-break.
- Extract mode: Bulk API 2.0 `queryAll`.
- Delete handling: map Salesforce `IsDeleted` into destination soft-delete behavior.
- Load order per run:
  1. agencies
  2. suppliers
  3. employees
  4. itineraries
  5. itinerary_items

### API Safety and Failure Behavior
- Singleton lock prevents overlapping scheduled runs.
- Conservative polling and per-run API budgets are enforced.
- No automatic retries; failures stop current run and continue next scheduled interval.
- Endpoint allowlist blocks non-Bulk/token Salesforce API paths.

## Contract Rules
- Query params remain `snake_case`
- JSON payload fields remain `camelCase`
- New terms and metric labels follow `docs/swainos-terminology-glossary.md`
- No compatibility shims: active contracts are represented directly and refactored when needed
- AI insights require live model execution (no deterministic fallback contract)
