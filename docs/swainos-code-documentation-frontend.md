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
- `apps/web/src/lib/api`: typed API clients, server loaders, and HTTP client
- `apps/web/src/lib/types`: contract types
- `apps/web/src/lib/utils`: shared parsing/formatting helpers

## Route Surface
- `/command-center`
- `/cash-flow`
- `/cash-flow/forecast`
- `/cash-flow/ap-schedule`
- `/cash-flow/scenarios`
- `/debt-service`
- `/itinerary-forecast`
- `/itinerary-actuals`
- `/travel-consultant`
- `/travel-agencies`
- `/fx-command`
- `/marketing`
- `/marketing/page-activity`
- `/marketing/geography-events`
- `/marketing/search-performance`
- `/marketing/search-console-insights`
- `/marketing/ai-website-insights`
- `/operations`
- `/ai-insights`
- `/settings`
- `/revenue-bookings` (route exists and redirects to `/itinerary-forecast`)

## Data Access Pattern
- Service modules under `lib/api/*Service.ts`
- Server-first route loaders under `features/*/*-server-loader.ts`
- Shared parallel server loader orchestration in `lib/api/parallelFetch.ts`
- Envelope handling in `lib/api/httpClient.ts`
- `httpClient` raises typed `ApiClientError` values and wraps network failures with actionable diagnostics (`network_error`), including resolved API base/path context
- In local development, `httpClient` falls back to `http://127.0.0.1:8000` when `NEXT_PUBLIC_API_BASE` is not set
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
  - cash-flow, deposits, payments-out, booking-forecasts, itinerary-lead-flow, itinerary outlook/actuals/deposits
- Cash-flow module reads dedicated risk-first endpoints:
  - `/api/v1/cash-flow/risk-overview`
  - `/api/v1/cash-flow/forecast`
  - `/api/v1/cash-flow/ap-schedule`
  - `/api/v1/cash-flow/scenarios` (read-only)
- Command center keeps existing lightweight cash-flow summary contract:
  - `/api/v1/cash-flow/summary`
- Debt service reads:
  - `/api/v1/debt-service/overview`
  - `/api/v1/debt-service/facilities`
  - `/api/v1/debt-service/schedule`
  - `/api/v1/debt-service/payments` (GET/POST)
  - `/api/v1/debt-service/scenarios` and `/api/v1/debt-service/scenarios/run`
- Travel consultant pages read leaderboard/profile endpoints
- Travel agencies pages read agent/agency leaderboards, profiles, and trade search
- FX Command reads rates/exposure/signals/holdings/transactions/intelligence/invoice-pressure
- Marketing Web Analytics reads:
  - `/api/v1/marketing/web-analytics/overview`
  - `/api/v1/marketing/web-analytics/page-activity`
  - `/api/v1/marketing/web-analytics/geo`
  - `/api/v1/marketing/web-analytics/events`
  - `/api/v1/marketing/web-analytics/search`
  - `/api/v1/marketing/web-analytics/search-console`
  - `/api/v1/marketing/web-analytics/ai-insights`
- AI Insights reads briefing/feed/recommendations/history/entity insights

## UX and Composition Notes
- System shell and navigation live in `components/layout/*`
- Assistant panel uses module/entity context and entity AI endpoint when entity context is present
- Itinerary lead-flow panel is rendered on itinerary actuals, not itinerary forecast
- Chart containers use fluid `ResponsiveContainer` rendering; shell enforces `min-w-0` and overflow guards for responsive stability
- Forecast chart card labels:
  - `Booked This Year` (calendar-year actuals source)
  - `Expected`
  - `Forecast`
  - `Target (+12% YoY)`
- Debt Service module includes:
  - Creditor-level debt table (`Debt by Creditor`) showing lender, facility, outstanding, and next due values
  - Top-bar manual payment action that opens a prefilled prompt and only posts on explicit confirm
  - Facility selector to target schedule/payments/scenarios and payment posting per selected loan
  - Initial server-load failure path returns a safe snapshot with user-visible error state instead of route crash
  - KPI typography can be tuned per page with `valueClassName` overrides (Debt Service currently uses compact `1rem` values)
  - Event-driven refresh and mutation handlers (`useState`, `useMemo`, `useTransition`) without `useEffect`-driven synchronization logic
- Cash Flow + Command Center liability posture includes explicit AR/AP semantics:
  - Supplier liabilities from AP line-based rollups (`payments-out` and AP endpoints)
  - Customer receivable/deposit posture from deposits summary
  - AR/AP-adjusted liquidity displayed as a first-class metric
- Cash Flow module is split into overview + detail routes with decision-first order:
  - Overview answers `Are we at risk?`, `When is first risk?`, and `Why?`
  - Forecast and AP Schedule provide drill-down tables by currency and horizon
  - Scenarios page is read-only and does not mutate baseline data
- Marketing module is split into strategic operator tabs:
  - Web Analytics Overview for KPI + trend direction (DoD/MoM/YoY) plus visual trend graphs (30d line trend, 6-month MoM bars, 12-month YoY sessions comparison, and a rolling 12-month horizontal sessions+YoY indicator table); KPI windows are served from synced canonical period summaries (not client-side rollups)
  - Page Activity for page-level usage behavior, itinerary diagnostics, best/worst ranking, dedicated lookbook/destination activity, and explicit rescue/scale focus cards
  - Geography & Events for geo segmentation, audience demographics, device mix, event meaning transparency, and market-priority focus cards (top-country cards use exact same-window country totals)
  - Source Tracking (route: `/marketing/search-performance`) for source/medium mix, referral analysis, value-ranked source decisions, and explicit traffic-quality scoring (`qualifiedSessionRate`, `bounceRate`, `qualityLabel`) plus landing-page/internal-search demand
  - Search Console Insights (route: `/marketing/search-console-insights`) for Search Console readiness plus SEO proxy analytics while query-level ingestion is deferred; it is a dedicated left-navigation destination, not a Web Analytics sub-tab
  - AI Website Insights for structured marketer/sales action cards with category, focus area, owner hint, target, impact score, and confidence score
  - Shared `Market` selector (layout-level) defaults to `United States`, supports `All markets`, and preserves `country` + `days` URL state across tab/date navigation for consistent cross-surface scope.

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
