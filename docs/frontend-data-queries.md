# Frontend Data Queries (Current)

> Purpose: Single source of truth for all API queries used by the frontend UI.

## Conventions
- All endpoints use `{ data, pagination, meta }` envelopes.
- Query params are `snake_case`.
- Currency scope for FX is **ZAR, USD, AUD, NZD only**.

---

## Live Queries In Use

| Endpoint | Purpose | Query Params | Used In | Notes |
|---|---|---|---|---|
| `GET /api/v1/cash-flow/summary` | Cash in/out totals | `time_window`, `currency_code`, `page`, `page_size` | `features/command-center/useCommandCenterData.ts`, `features/cash-flow/cash-flow-dashboard.tsx` | `time_window` currently `90d` |
| `GET /api/v1/cash-flow/timeseries` | Cashflow time series | `time_window`, `currency_code`, `page`, `page_size` | `features/cash-flow/cash-flow-dashboard.tsx` | `time_window` currently `90d` |
| `GET /api/v1/deposits/summary` | Deposit totals | `time_window`, `currency_code`, `page`, `page_size` | `features/command-center/useCommandCenterData.ts`, `features/deposits/deposits-summary.tsx` | `time_window` currently `90d` |
| `GET /api/v1/payments-out/summary` | Supplier invoice totals | `time_window`, `currency_code`, `page`, `page_size` | `features/command-center/useCommandCenterData.ts`, `features/payments-out/payments-out-summary.tsx` | `time_window` currently `90d` |
| `GET /api/v1/booking-forecasts` | Forecasted bookings | `lookback_months`, `horizon_months`, `page`, `page_size` | `features/command-center/useCommandCenterData.ts` | `lookback_months=12`, `horizon_months=3` |
| `GET /api/v1/itinerary-trends` | Itinerary creation/closure/travel trends | `time_window` | `features/command-center/useCommandCenterData.ts`, `features/itinerary-forecast/useItineraryForecastOutlook.ts`, `features/itinerary-actuals/useItineraryActualsYoy.ts` | Used for lead-flow tracking in Forecast/Actuals |
| `GET /api/v1/itinerary-revenue/outlook` | Owner forward outlook (on-books/potential/expected) | `time_window`, `grain`, `currency_code` | `features/itinerary-forecast/useItineraryForecastOutlook.ts` | Primary itinerary forecast source |
| `GET /api/v1/itinerary-revenue/deposits` | Deposit health against 25% target | `time_window`, `grain`, `currency_code` | `features/itinerary-forecast/useItineraryForecastOutlook.ts` | Based on `mv_itinerary_deposit_monthly` |
| `GET /api/v1/itinerary-revenue/conversion` | Quoted-to-confirmed conversion and projection | `time_window`, `grain`, `currency_code` | `features/itinerary-forecast/useItineraryForecastOutlook.ts` | Uses 12m lookback close ratio |
| `GET /api/v1/itinerary-revenue/channels` | Top consortia and trade agency performance | `time_window`, `grain`, `currency_code` | `features/itinerary-forecast/useItineraryForecastOutlook.ts` | Pulls from consortia/trade monthly rollups |
| `GET /api/v1/itinerary-revenue/actuals-yoy` | Jan-Dec year-over-year itinerary actuals | `years_back`, `currency_code` | `features/itinerary-actuals/useItineraryActualsYoy.ts` | Primary Itinerary Actuals source (travel-end basis) |
| `GET /api/v1/itinerary-revenue/actuals-channels` | Closed-won top consortia and trade agency production for actuals | `years_back`, `actuals_year`, `currency_code` | `features/itinerary-actuals/useItineraryActualsYoy.ts` | Uses actuals channel rollups by travel-end month; `actuals_year` used for current/last-year scope toggles |
| `GET /api/v1/fx/rates` | Live FX rates | `limit` | `app/fx-command/page.tsx` | Filtered to ZAR/USD/AUD/NZD pairs server-side |
| `GET /api/v1/fx/exposure` | Exposure rollup | none | `app/fx-command/page.tsx` | Returns empty if view missing; UI falls back to demo exposure |

---

## Backend Endpoints Not Consumed by UI

| Endpoint | Status | Notes |
|---|---|---|
| `GET /api/v1/revenue-bookings` | Deprecated and removed | Replaced by `/api/v1/itinerary-revenue/*` owner cockpit endpoints. |
| `GET /api/v1/revenue-bookings/{booking_id}` | Deprecated and removed | Replaced by owner cockpit and module-specific detail surfaces. |
| `GET /api/v1/itinerary-pipeline` | Deprecated and removed | Replaced by `/api/v1/itinerary-revenue/outlook` and `/conversion`. |

---

## Notes for Review
- All queries are sourced from `lib/api/*Service.ts` with typed mappings in `lib/types/*`.
- Itinerary revenue surfaces use `commissionIncomeAmount` as the primary income metric, now sourced from itinerary `gross_profit` (API field name retained for compatibility).
- Closed-won groupings behind itinerary revenue actuals/channels use the allowlist statuses:
  `Deposited/Confirming`, `Amendment in Progress`, `Pre-Departure`, `eDocs Sent`, `Traveling`, `Traveled`, `Cancel Fees`.
- Numbers returned as strings are normalized in `lib/utils/parseNumber.ts` before rendering.
- Command Center uses `Promise.allSettled` so one failed query does not block other modules.

