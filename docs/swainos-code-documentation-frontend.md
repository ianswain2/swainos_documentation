# SwainOS Frontend Code Documentation

## Overview
SwainOS frontend is a Next.js App Router application with feature-based modules and typed API services.

## Stack
- Next.js + React + TypeScript (strict)
- Tailwind CSS
- Recharts and lightweight-charts for visualizations

## Structure
- `apps/web/src/app`: route entrypoints
- `apps/web/src/features`: page-level feature modules and hooks
- `apps/web/src/components`: shared layout and UI primitives
- `apps/web/src/lib/api`: typed API clients and HTTP client
- `apps/web/src/lib/types`: contract types
- `apps/web/src/lib/utils`: shared parsing/formatting helpers

## Route Surface
- `/command-center`
- `/cash-flow`
- `/debt-service`
- `/itinerary-forecast`
- `/itinerary-actuals`
- `/travel-consultant`
- `/travel-agencies`
- `/fx-command`
- `/operations`
- `/ai-insights`
- `/settings`
- `/revenue-bookings` (route exists; active itinerary analytics live in itinerary forecast/actuals modules)

## Data Access Pattern
- Service modules under `lib/api/*Service.ts`
- Envelope handling in `lib/api/httpClient.ts`
- Number normalization in `lib/utils/parseNumber.ts`
- In-flight GET dedupe is enabled
- GET caching is disabled in non-production and enabled in production unless `skipCache` is set

## Feature Contracts
- Itinerary forecast reads:
  - `/api/v1/itinerary-revenue/outlook`
  - `/api/v1/itinerary-revenue/deposits`
  - `/api/v1/itinerary-revenue/conversion`
  - `/api/v1/itinerary-revenue/channels`
  - `/api/v1/itinerary-revenue/actuals-yoy` (for Booked This Year card)
- Itinerary actuals reads:
  - `/api/v1/itinerary-revenue/actuals-yoy`
  - `/api/v1/itinerary-revenue/actuals-channels`
  - `/api/v1/itinerary-lead-flow`
- Command center reads:
  - cash-flow, deposits, payments-out, booking-forecasts, itinerary-trends, itinerary-lead-flow
- Travel consultant pages read leaderboard/profile endpoints
- Travel agencies pages read agent/agency leaderboards, profiles, and trade search
- FX Command reads rates/exposure/signals/holdings/transactions/intelligence/invoice-pressure
- AI Insights reads briefing/feed/recommendations/history/entity insights

## UX and Composition Notes
- System shell and navigation live in `components/layout/*`
- Assistant panel uses module/entity context and entity AI endpoint when entity context is present
- Itinerary lead-flow panel is rendered on itinerary actuals, not itinerary forecast
- Forecast chart card labels:
  - `Booked This Year` (calendar-year actuals source)
  - `Expected`
  - `Forecast`
  - `Target (+12% YoY)`

## Environment
- Frontend env file: `apps/web/.env.local`
- Required: `NEXT_PUBLIC_API_BASE`
- For FX server proxy route: `API_BASE`, `FX_MANUAL_RUN_TOKEN`
- Optional: `NEXT_PUBLIC_MAPBOX_TOKEN`

## Conventions
- Component files: `kebab-case.tsx`
- Utility files: `camelCase.ts`
- No unused imports, dead code, or compatibility shims
- Contract/display terms align with `docs/swainos-terminology-glossary.md`
