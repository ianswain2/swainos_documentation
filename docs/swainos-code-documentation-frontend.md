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
- `/login`
- `/auth/callback`
- `/unauthorized`
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
- `/marketing/search-console-insights/pages/[...pagePath]`
- `/marketing/ai-website-insights`
- `/operations`
- `/ai-insights`
- `/settings`
- `/settings/run-logs`
- `/settings/user-access`
- `/revenue-bookings` (route exists and redirects to `/itinerary-forecast`)

## Data Access Pattern
- Service modules under `lib/api/*Service.ts`
- Server-first route loaders under `features/*/*-server-loader.ts`
- Shared parallel server loader orchestration in `lib/api/parallelFetch.ts`
- Envelope handling in `lib/api/httpClient.ts`
- Supabase clients:
  - browser: `lib/supabase/browser.ts`
  - server: `lib/supabase/server.ts`
- Auth route protection and cookie/session synchronization: `src/proxy.ts` (Next.js proxy file convention)
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
  - `/api/v1/marketing/web-analytics/search-console/page-profile`
  - `/api/v1/marketing/web-analytics/ai-insights`
- AI Insights reads briefing/feed/recommendations/history/entity insights
- Auth/access reads:
  - `/api/v1/auth/me` (SSR access resolution path)
  - `/api/v1/settings/user-access` (admin page server load)
  - `/api/v1/settings/user-access/{user_id}` (update flows)
  - `/api/v1/settings/user-access/{user_id}/deactivate`
  - `/api/v1/settings/user-access/{user_id}/reactivate`
- Settings and Operations read the canonical data-jobs control-plane APIs:
  - `/api/v1/data-jobs`
  - `/api/v1/data-jobs/run-feed`
  - `/api/v1/data-jobs/{job_key}`
  - `/api/v1/data-jobs/{job_key}/runs`
  - `/api/v1/data-jobs/health`
  - `/api/v1/data-job-runs/{run_id}`

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
- Search Console Insights (route: `/marketing/search-console-insights`) is a dedicated tabbed workspace (`Overview`, `Opportunities`, `Challenges`, `Queries`, `Pages`, `Diagnostics`) backed by canonical Search Console snapshots; it remains a dedicated left-navigation destination, not a Web Analytics sub-tab
  - Overview is AI-led (`What matters this week`) with deterministic action callouts, market benchmark comparison, and compact KPI support cards
  - Opportunities and Challenges render as clickable tables with explicit typed categories (`opportunityType`, `challengeType`)
  - Queries is a keyword-visibility workspace with intent/rank-band context and summary chips
  - Pages links drill down into dedicated page profiles at `/marketing/search-console-insights/pages/[...pagePath]`
  - AI Website Insights for structured marketer/sales action cards with category, focus area, owner hint, target, impact score, and confidence score
  - Shared `Market` selector (layout-level) is used on web-analytics routes and preserves `country` + `days` URL state; Search Console Insights is intentionally US-first and hides market selection.
- Settings page (`/settings`) presents a compact, grouped data-jobs admin surface:
  - rows are grouped by `jobKind` (`Source Ingestion`, `Rollup Refresh`, `Derived Compute`, `Manual Imports`, `Maintenance`)
  - each row shows last-run timestamp and last-run status from `/api/v1/data-jobs/health`
  - recurring schedules display plain-English cadence labels (cron retained as secondary detail)
- Settings navigation is left-nav driven under `Settings` with explicit children (`Job Controls`, `Run Logs`) instead of a top-page toggle.
- Access navigation controls are permission-aware:
  - side nav filters routes by permission keys for members
  - admin users are treated as full-access
  - unauthorized route access redirects to `/unauthorized`
- Login flow:
  - `/login` performs Supabase password sign-in
  - callback exchange route is `/auth/callback`
  - protected-route redirects are handled in `proxy.ts`
- Settings Run Logs (`/settings/run-logs`) provides a compact cross-job run feed with filters and periodic polling:
  - reads `/api/v1/data-jobs/run-feed` with optional `job_key` and `run_status`
  - uses paginated history controls (`Previous`/`Next`) backed by API pagination totals so operators can move beyond recent rows
  - displays persisted run analytics fields (`durationSeconds`, `outputSizeBytes`) for historical performance analysis
  - periodic refresh uses a minimal `useEffect` polling loop as an explicit external-system synchronization exception
  - polling pauses when browser tab is hidden and resumes with immediate refresh on visibility return

## Environment
- Frontend env file: `apps/web/.env.local`
- Required:
  - `NEXT_PUBLIC_API_BASE`
  - `NEXT_PUBLIC_SUPABASE_URL`
  - `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- Optional: `NEXT_PUBLIC_MAPBOX_TOKEN`

## Conventions
- Component files: `kebab-case.tsx`
- Utility files: `camelCase.ts`
- No unused imports, dead code, or compatibility shims
- Contract/display terms align with `docs/swainos-terminology-glossary.md`
- Legacy manual-run frontend proxy route (`/app/api/fx/rates/run/route.ts`) is removed; active frontend manual runs call `/api/v1/data-jobs/{job_key}/runs`.
