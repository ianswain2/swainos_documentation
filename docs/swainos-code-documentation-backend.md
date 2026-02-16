# SwainOS Backend Code Documentation

## Overview
SwainOS backend is a FastAPI service layered as routes → services → repositories. The current MVP focuses on read-only analytics using Salesforce/Kaptio data already loaded into Supabase, with REST upsert scripts for historical data ingestion.

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
- `src/services/travel_consultants_service.py`: consultant KPI aggregation, YoY storytelling, signals, and forecast logic.
- `src/repositories/travel_consultants_repository.py`: consultant rollup materialized view access.
- `scripts/upsert_bookings.py`: REST upsert loader for bookings.
- `scripts/upsert_itineraries.py`: REST upsert loader for enriched itineraries with external-id FK resolver for agencies/contacts/employees.
- `scripts/upsert_employees.py`: REST upsert loader for Salesforce consultant identity records.
- `scripts/upsert_itinerary_items.py`: REST upsert loader for itinerary items.
- `scripts/upsert_customer_payments.py`: REST upsert loader for customer payments.
