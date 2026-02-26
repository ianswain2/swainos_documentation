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
- Command center core: `/cash-flow/*`, `/deposits/*`, `/payments-out/*`, `/booking-forecasts`, `/itinerary-trends`
- Revenue bookings: `/revenue-bookings`, `/revenue-bookings/{booking_id}`
- Itinerary revenue: `/itinerary-revenue/outlook|deposits|conversion|channels|actuals-yoy|actuals-channels`
- Itinerary lead flow: `/itinerary-lead-flow`
- Travel consultant: `/travel-consultants/leaderboard|{employee_id}/profile|{employee_id}/forecast`
- Travel trade: `/travel-agents/*`, `/travel-agencies/*`, `/travel-trade/search`
- FX: `/fx/rates|exposure|signals|transactions|holdings|intelligence|invoice-pressure` and run endpoints
- AI insights: `/ai-insights/briefing|feed|recommendations|history|entities/*|run`

## Data and Rollup Model
- Canonical Gross Profit contract key: `grossProfitAmount` (source column: `itineraries.gross_profit`)
- Status classification: `itinerary_status_reference` (`pipeline_bucket`, `pipeline_category`, `is_filter_out`)
- Itinerary revenue rollups are keyed by travel period
- Consultant and company AI context is materialized and refreshed via `refresh_consultant_ai_rollups_v1()`
- Travel trade analytics is refreshed via `refresh_travel_trade_rollups_v1()`
- FX exposure is refreshed via `refresh_fx_exposure_v1()`

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
- `scripts/refresh_consultant_ai_rollups.py`
- `scripts/refresh_travel_trade_rollups.py`
- `scripts/pull_fx_rates.py`
- `scripts/generate_fx_intelligence.py`
- `scripts/refresh_fx_exposure.py`
- `scripts/generate_ai_insights.py`

## Contract Rules
- Query params remain `snake_case`
- JSON payload fields remain `camelCase`
- New terms and metric labels follow `docs/swainos-terminology-glossary.md`
- No compatibility shims: active contracts are represented directly and refactored when needed
- AI insights require live model execution (no deterministic fallback contract)
