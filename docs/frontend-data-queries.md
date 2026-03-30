# Frontend Data Queries

Purpose: canonical map of **which frontend modules call which backend routes** (live usage, fixed-window behavior, and snapshot bundles).

**Prod / deploy note:** This map describes the app as implemented on `main`. If Vercel or Render are not deploying latest `main` (or the API is suspended), live behavior may differ; confirm deploy status and hostnames before debugging “missing” features.

## Path convention

Table paths such as `lib/...` and `features/...` are relative to **`apps/web/src/`** in the frontend repo.

## Conventions
- API envelope: `{ data, pagination, meta }` for **FastAPI** responses under `NEXT_PUBLIC_API_BASE` (`/api/v1/...`).
- Same-origin **Next.js route handlers** under `apps/web/src/app/api/**` (e.g. `POST /api/auth/login`) use a slim JSON shape documented in `sample-payloads.md`; they are not prefixed with `/api/v1` and are not served by the Python backend.
- Query params: `snake_case`
- Response JSON fields: `camelCase`
- Numeric payloads are normalized in `apps/web/src/lib/utils/parseNumber.ts`
- Frontend GET cache is disabled in non-production (`NODE_ENV !== "production"`)

## Server-first aggregates (dashboard snapshots)
These endpoints bundle multiple domain reads for SSR and optional backend caching. The UI does **not** re-fetch the underlying slices individually for these surfaces.

| Endpoint | Used in | Purpose |
|---|---|---|
| `GET /api/v1/dashboard-snapshots/command-center` | `lib/api/dashboardSnapshotsService.ts`, `features/command-center/command-center-server-loader.ts` | Command center bundle: cash-flow summaries (30d), deposits + payments-out summaries, booking forecasts, itinerary lead flow (12m), itinerary outlook (12m/monthly), itinerary actuals YoY (3y), debt overview, **plus** `GET /api/v1/ai-insights/briefing` (daily briefing) |
| `GET /api/v1/dashboard-snapshots/cash-flow` | `lib/api/dashboardSnapshotsService.ts`, `features/cash-flow/cash-flow-risk-server-loader.ts` | Cash-flow bundle: risk overview + 3m/12m forecast + scenarios (12m forward window) |

## Live direct queries (frontend callers)

| Endpoint | Used in | Purpose |
|---|---|---|
| `GET /api/v1/cash-flow/ap-schedule` | `lib/api/cashFlowService.ts`, `features/cash-flow/cash-flow-risk-server-loader.ts`, `features/cash-flow/cash-flow-ap-schedule.tsx` | Paginated AP schedule rows by due date/currency |
| `GET /api/v1/cash-flow/ap-monthly-outflow` | `lib/api/cashFlowService.ts`, `features/cash-flow/cash-flow-risk-server-loader.ts` | Monthly AP outflow trend (cash-flow overview page) |
| `GET /api/v1/ap/summary` | `lib/api/apService.ts` | AP open-line/booking/supplier liquidity rollup |
| `GET /api/v1/ap/aging` | `lib/api/apService.ts` | AP aging buckets by currency |
| `GET /api/v1/ap/payment-calendar` | `lib/api/apService.ts` | AP payment schedule timeline |
| `GET /api/v1/debt-service/overview` | `features/debt-service/debt-service-server-loader.ts` | Debt KPI overview (command center embeds the same payload via `dashboard-snapshots/command-center`, not a second browser call) |
| `GET /api/v1/debt-service/facilities` | `features/debt-service/debt-service-server-loader.ts` | Facility list and identifiers for debt slices |
| `GET /api/v1/debt-service/schedule` | `features/debt-service/debt-service-server-loader.ts`, `features/debt-service/debt-service-page.tsx` | Amortization timeline rows |
| `GET /api/v1/debt-service/payments` | `features/debt-service/debt-service-server-loader.ts`, `features/debt-service/debt-service-page.tsx` | Logged payment ledger entries |
| `POST /api/v1/debt-service/payments` | `features/debt-service/debt-service-page.tsx` | Principal/interest posting workflow |
| `GET /api/v1/debt-service/scenarios` | `features/debt-service/debt-service-server-loader.ts` | Persisted scenario result summaries |
| `POST /api/v1/debt-service/scenarios/run` | `features/debt-service/debt-service-page.tsx` | Deterministic payoff/interest scenario compare run |
| `GET /api/v1/itinerary-revenue/outlook` | `lib/api/itineraryRevenueService.ts`, `features/itinerary-forecast/itinerary-forecast-server-loader.ts`, `features/itinerary-actuals/itinerary-actuals-server-loader.ts` | Forward revenue outlook (forecast page: `12m` + `monthly`; actuals page: `12m` + `monthly` for context) |
| `GET /api/v1/itinerary-revenue/conversion` | `lib/api/itineraryRevenueService.ts`, `features/itinerary-forecast/itinerary-forecast-server-loader.ts` | Conversion timeline (`itinerary_pipeline_conversion_monthly_v1` + service projections) |
| `GET /api/v1/itinerary-revenue/booked-revenue-yoy` | `lib/api/itineraryRevenueService.ts`, `features/itinerary-forecast/itinerary-forecast-server-loader.ts` | Close-date-keyed booked-revenue YoY matrix used by itinerary forecast booked-history surfaces (closed lifecycle: `closed_won` + `closed_active`) |
| `GET /api/v1/itinerary-revenue/actuals-yoy` | `lib/api/itineraryRevenueService.ts`, `features/itinerary-actuals/itinerary-actuals-server-loader.ts` | Multi-year Jan–Dec matrix + year summaries (loader uses `years_back=3`) |
| `GET /api/v1/itinerary-revenue/actuals-channels` | `lib/api/itineraryRevenueService.ts`, `features/itinerary-actuals/itinerary-actuals-server-loader.ts` | Consortia + trade channel actuals for selected calendar year (default current year) |
| `GET /api/v1/itinerary-revenue/actuals-channels-comparison` | `lib/api/itineraryRevenueService.ts`, `features/sales/travel-trade-server-loader.ts` | Travel Agencies booking-pace channels for the selected travel cohort (`travel_start_date` in window) using month-grain booking cutoff (`close_date` month <= as-of month) with prior-year same-month comparison (`priorYear`) |
| `GET /api/v1/itinerary-lead-flow` | `lib/api/itineraryLeadFlowService.ts`, `features/itinerary-actuals/itinerary-actuals-server-loader.ts` | Lead-flow trend (loader uses `36m` window) — command center consumes lead flow only via dashboard snapshot |
| `GET /api/v1/travel-consultants/leaderboard` | `lib/api/travelConsultantService.ts`, `features/travel-consultant/travel-consultant-server-loader.ts` | Consultant ranking by **travel** metrics: primary read `vw_travel_consultant_travel_monthly_v2` (itinerary **`travel_start_date`** month, closed lifecycle `closed_won` + `closed_active`). Default `sort_by=travel_revenue`. Response lists `travelRankings` with `travelRevenue` and `travelGrossProfit`. Funnel counts and PYTD **travel revenue** / **travel gross profit** variance fields still mix in lead and booking-pace baselines as implemented in the service. |
| `GET /api/v1/travel-consultants/{employee_id}/profile` | `lib/api/travelConsultantService.ts`, `features/travel-consultant/travel-consultant-server-loader.ts` | Consultant profile: hero KPIs and **travel** rollups use `vw_travel_consultant_profile_monthly_v2` (**travel-start** basis, closed lifecycle). **Booked** (close-date) matrices use `vw_travel_consultant_booked_revenue_monthly_v2` and are capped through the **current calendar year** only (no future-year close dates). Multi-year **travel** matrices include the anchor year plus prior years and **next** calendar year for forward travel. |
| `GET /api/v1/travel-agents/leaderboard` | `lib/api/travelAgentsService.ts`, `features/sales/travel-trade-server-loader.ts` | Agent leaderboard — Travel Agencies route hard-codes current calendar year + `top_n=10` |
| `GET /api/v1/travel-agents/{agent_id}/profile` | `lib/api/travelAgentsService.ts`, `features/sales/travel-trade-server-loader.ts` | Agent profile (includes travel-period `bookedItineraries` table). YoY chart series uses booking-pace month-cutoff comparisons |
| `GET /api/v1/travel-agencies/leaderboard` | `lib/api/travelAgenciesService.ts`, `features/sales/travel-trade-server-loader.ts` | Agency leaderboard — same fixed window as agents |
| `GET /api/v1/travel-agencies/{agency_id}/profile` | `lib/api/travelAgenciesService.ts`, `features/sales/travel-trade-server-loader.ts` | Agency profile — hard-coded current-year window; YoY chart series uses booking-pace month-cutoff comparisons |
| `GET /api/v1/suppliers/leaderboard` | `lib/api/suppliersService.ts`, `features/suppliers/suppliers-server-loader.ts`, `app/suppliers/page.tsx` | Supplier production leaderboard for current-year travel revenue YTD (`itinerary_items` fact basis; current-year cut at `today`, prior-year comparator uses same-month/day cutoff) with hierarchy-aware location filters (`country`, `region`, `city`, `location_query`); supplier `itineraryCount` is sourced from MV-backed distinct itinerary counts (`supplier_distinct_itinerary_counts_v1` over `mv_supplier_itinerary_fact_v1`, not summed location buckets) |
| `GET /api/v1/suppliers/profiles` | `lib/api/suppliersService.ts`, `features/suppliers/suppliers-server-loader.ts`, `app/suppliers/profiles/page.tsx` | Supplier directory payload for profile discovery/navigation |
| `GET /api/v1/suppliers/{supplier_id}/profile` | `lib/api/suppliersService.ts`, `features/suppliers/suppliers-server-loader.ts`, `app/suppliers/[supplierId]/page.tsx` | Supplier profile KPIs, YoY series, and top location mix (current-year focus) |
| `GET /api/v1/travel-trade/search` | `features/sales/useTravelTradeSearch.ts`, `features/sales/travel-agencies-page.tsx` | Unified trade search |
| `GET /api/v1/itinerary-destinations/summary` | `features/sales/destination-server-loader.ts`, `features/sales/destination-page.tsx` | Booked destination KPI summary |
| `GET /api/v1/itinerary-destinations/trends` | `features/sales/destination-server-loader.ts`, `features/sales/destination-page.tsx` | Monthly destination trend |
| `GET /api/v1/itinerary-destinations/breakdown` | `features/sales/destination-server-loader.ts`, `features/sales/destination-page.tsx` | Country/city breakdown |
| `GET /api/v1/itinerary-destinations/matrix` | `features/sales/destination-server-loader.ts`, `features/sales/destination-page.tsx` | Horizontal Jan–Dec matrix + YoY |
| `GET /api/v1/ai-insights/feed` | `features/ai-insights/useAiInsightsData.ts` | AI events feed |
| `GET /api/v1/ai-insights/recommendations` | `features/ai-insights/useAiInsightsData.ts`, `features/travel-consultant/travel-consultant-server-loader.ts` | Recommendation queue |
| `PATCH /api/v1/ai-insights/recommendations/{id}` | `features/ai-insights/useAiInsightsData.ts` | Recommendation status transition |
| `GET /api/v1/ai-insights/history` | `features/ai-insights/useAiInsightsData.ts` | Historical AI events |
| `GET /api/v1/ai-insights/entities/{entity_type}/{entity_id}` | `features/travel-consultant/profile/useTravelConsultantEntityAi.ts`, `components/assistant/assistant-panel.tsx` | Entity-scoped AI context |
| `POST /api/auth/login` | `app/login/page.tsx` | Same-origin password sign-in; server route sets Supabase session cookies and applies app-layer throttle (`lib/auth/loginRateLimit.ts`). When `NEXT_PUBLIC_TURNSTILE_SITE_KEY` is set, request body must include `captchaToken` (see `sample-payloads.md`). |
| `GET /api/v1/auth/me` | `lib/auth/getAuthenticatedUser.ts` | SSR-authenticated access resolution (role + permission keys) |
| `GET /api/v1/settings/user-access` | `app/settings/user-access/page.tsx`, `features/settings/user-access-page.tsx` | Admin user-access list and page bootstrapping (**admin-only** route; non-admins never reach the page) |
| `PUT /api/v1/settings/user-access/{user_id}` | `features/settings/user-access-page.tsx` | Admin role/permission updates |
| `POST /api/v1/settings/user-access/{user_id}/deactivate` | `features/settings/user-access-page.tsx` | Admin deactivation action |
| `POST /api/v1/settings/user-access/{user_id}/reactivate` | `features/settings/user-access-page.tsx` | Admin reactivation action |
| `GET /api/v1/fx/rates` | `app/fx-command/page.tsx`, `features/fx-command/useFxCommandData.ts` | FX rate series and metadata |
| `GET /api/v1/fx/exposure` | `app/fx-command/page.tsx`, `features/fx-command/useFxCommandData.ts` | Exposure by currency |
| `GET /api/v1/fx/signals` | `app/fx-command/page.tsx`, `features/fx-command/useFxCommandData.ts` | Buy/wait signals |
| `GET /api/v1/fx/holdings` | `app/fx-command/page.tsx`, `features/fx-command/useFxCommandData.ts` | Holdings snapshot |
| `GET /api/v1/fx/transactions` | `app/fx-command/page.tsx`, `features/fx-command/useFxCommandData.ts` | Ledger feed |
| `POST /api/v1/fx/transactions` | `features/fx-command/fx-command-page.tsx` | Ledger write path |
| `GET /api/v1/fx/intelligence` | `app/fx-command/page.tsx`, `features/fx-command/useFxCommandData.ts` | Intelligence feed |
| `GET /api/v1/fx/invoice-pressure` | `lib/api/fxServerService.ts`, `lib/api/fxService.ts`, `features/fx-command/useFxCommandData.ts` | Due-window payable pressure sourced from AP rollups |
| `GET /api/v1/marketing/web-analytics/overview` | `lib/api/marketingService.ts`, `features/marketing/marketing-server-loader.ts` | Strategic web analytics KPIs and trend context; optional `country` scope (`United States` / `all`) |
| `GET /api/v1/marketing/web-analytics/page-activity` | `lib/api/marketingService.ts`, `features/marketing/marketing-server-loader.ts` | Page-level behavior + quality scoring; optional `days_back` + `country` |
| `GET /api/v1/marketing/web-analytics/geo` | `lib/api/marketingService.ts`, `features/marketing/marketing-server-loader.ts` | Geographic performance + demographics + device mix |
| `GET /api/v1/marketing/web-analytics/events` | `lib/api/marketingService.ts`, `features/marketing/marketing-server-loader.ts` | Event catalog + conversion classification |
| `GET /api/v1/marketing/web-analytics/search` | `lib/api/marketingService.ts`, `features/marketing/marketing-server-loader.ts` | Source Tracking surface; optional `days_back` + `country` |
| `GET /api/v1/marketing/web-analytics/search-console` | `lib/api/marketingService.ts`, `features/marketing/marketing-server-loader.ts` | Search Console Insights workspace |
| `GET /api/v1/marketing/web-analytics/search-console/page-profile` | `lib/api/marketingService.ts`, `features/marketing/marketing-server-loader.ts`, `app/marketing/search-console-insights/pages/[...pagePath]/page.tsx` | Single URL profile |
| `GET /api/v1/marketing/web-analytics/ai-insights` | `lib/api/marketingService.ts`, `features/marketing/marketing-server-loader.ts` | Structured marketer/sales action engine output |
| `GET /api/v1/data-jobs` | `app/settings/page.tsx`, `app/operations/page.tsx` | Control-plane job inventory (**Settings** path admin-only; **Operations** remains permission-gated) |
| `GET /api/v1/data-jobs/run-feed` | `app/settings/run-logs/page.tsx`, `features/settings/settings-run-logs-page.tsx` | Cross-job run stream (`job_key`, `run_status`, pagination) (**admin-only** when used from Settings run logs) |
| `PATCH /api/v1/data-jobs/{job_key}` | `features/settings/settings-page.tsx` | Toggle job enabled state and schedule controls (**admin-only** page) |
| `POST /api/v1/data-jobs/{job_key}/runs` | `features/settings/settings-page.tsx`, `lib/api/fxService.ts`, `lib/api/aiInsightsService.ts` | Canonical manual run trigger |
| `GET /api/v1/data-jobs/{job_key}/runs` | `app/operations/page.tsx` | Per-job run history |
| `GET /api/v1/data-jobs/health` | `app/operations/page.tsx`, `app/settings/page.tsx` | Fleet health + Settings last-run/status (**Settings** admin-only) |
| `GET /api/v1/data-job-runs/{run_id}` | `features/operations/operations-page.tsx` | Run detail and step diagnostics |

## Typed API clients without route callers

These `lib/api/*` wrappers are not imported by any `app/` route or feature loader today; the backend routes remain available.

| Symbol / path | Endpoint |
|---|---|
| `cashFlowService.getSummary` | `GET /api/v1/cash-flow/summary` |
| `cashFlowService.getRiskOverview` | `GET /api/v1/cash-flow/risk-overview` |
| `cashFlowService.getForecast` | `GET /api/v1/cash-flow/forecast` |
| `cashFlowService.getScenarios` | `GET /api/v1/cash-flow/scenarios` |
| `depositsService.getSummary` | `GET /api/v1/deposits/summary` |
| `paymentsOutService.getSummary` | `GET /api/v1/payments-out/summary` |
| `bookingForecastsService` | `GET /api/v1/booking-forecasts` |

Command center and cash-flow overview / forecast / scenarios use **dashboard snapshots** on the backend instead of these direct client calls.

## Backend routes with no current web UI

| Endpoint | Note |
|---|---|
| `GET /api/v1/debt-service/covenants` | Live API; debt UI uses aggregate covenant posture from overview only. |
| `GET /api/v1/travel-consultants/{employee_id}/forecast` | Forecast is embedded in profile; standalone route unused by UI. |
| `GET /api/v1/marketing/web-analytics/health` | Operator/diagnostic; no dedicated Marketing tab. |
| `GET /api/v1/itinerary-revenue/deposits` | **Removed** from the API; deposit MVs may still exist for jobs/ops. |

## Fixed analytics windows (current product behavior)
- **Travel Agencies** (`/travel-agencies`): current **calendar year** for agent/agency leaderboards plus booking-pace channel comparison (`close_date` month-cutoff; prior-year same-month baseline).
- **Travel Consultant** leaderboard + profiles: route loaders use fixed windows (typically current-year leaderboard). **Leaderboard** ranking and primary currency columns are **travel-start-date** metrics (`travelRevenue`, `travelGrossProfit`) from `vw_travel_consultant_travel_monthly_v2`, not close-date booked revenue. **Profile** exposes three matrices: **Travel Gross Profit**, **Travel Revenue** (both travel-start, including future active months and optional next-year column in the matrix), and **Booked Revenue** (close-date, years capped at today’s calendar year). Funnel and some PYTD comparison fields still use booking-pace or lead semantics as computed in `TravelConsultantsService`. Leaderboard top-summary callouts aggregate from the full returned `travelRankings` list (never from UI search-filtered rows).
- **Travel Agent / Agency profiles** (`/travel-agencies/agents/[agentId]`, `/travel-agencies/agencies/[agencyId]`): current window KPIs stay lead/travel-period scoped; YoY chart series use booking-pace month-cutoff comparisons; agent profile retains travel-period `bookedItineraries`.
- **Itinerary Actuals** (`/itinerary-actuals`): fixed **3-year** YoY matrix + lead flow **36m** + **current-year** channel actuals + **12m** outlook context; no channel-scope toggle in UI (loader default `current-year`).
- **Itinerary Forecast** (`/itinerary-forecast`): fixed **12m** monthly outlook + conversion plus booked-revenue YoY (`years_back=5`) for booking-date history surfaces; outlook chart metric is **Gross Profit** only in UI (no client-side refetch hooks).

## Operational notes
- Itinerary revenue surfaces use `grossProfitAmount` as canonical Gross Profit.
- Itinerary actuals stay **travel-period** reporting. Travel Agencies channel comparison and consultant/agency/agent YoY-variance fields now use booking-pace month-cutoff logic by `close_date`.
- Booking-pace closed-lifecycle reads and booked-revenue-by-close-date reads include both `closed_won` and `closed_active` statuses from `itinerary_status_reference`.
- Itinerary lead-flow panel: **itinerary actuals** (direct API) and **command center** (via dashboard snapshot only).
- Destination matrix API still carries passenger fields; destination UI hides passenger metrics where noted in product rules.
- Debt Service payment posting is user-confirmed from a prefilled prompt.
- Cash Flow subpages: overview/forecast/scenarios hydrate from `dashboard-snapshots/cash-flow`; AP schedule + monthly outflow use dedicated cash-flow routes.
- Cash Flow renders per-currency rows; avoid mixed-currency totals as a single currency.
- Search Console Insights is US-first; benchmark markets remain Australia, New Zealand, and South Africa.
- Search Console page-profile route encodes page paths (including absolute URLs) before backend fetch.
- Manual-run triggers use `POST /api/v1/data-jobs/{job_key}/runs`; token utility routes (`/ai-insights/run`, `/fx/signals/run`) remain backend-only.
- Sign-in uses `POST /api/auth/login` on the Next.js host only; analytics API traffic continues to use `GET/POST /api/v1/...` on the FastAPI host.
- **Settings vs Operations:** `/settings`, `/settings/run-logs`, and `/settings/user-access` are **admin-only** in the root layout (`adminOnly` + `role === admin`). `/operations` remains a normal module permission (`operations`); members with that key can use Operations without accessing Settings.
- Salesforce operators use data-jobs + run detail (`output.parsed`) — no direct Salesforce calls from the web app.
- Semantic rollup v2 serving views back travel-trade leaderboard/profile, booking-pace channel-comparison reads, and consultant **booked** (close-date) revenue paths; consultant **travel** leaderboard rows are served from `vw_travel_consultant_travel_monthly_v2` (replacing the retired misnamed `vw_travel_consultant_booked_monthly_v2` for that use case). Apply backend migrations `0146`–`0148` (and dependencies) so profile, aliases, and parity checks align.

## Related documentation

- [Backend code documentation](swainos-code-documentation-backend.md) — layering, endpoint families, rollups
- [Frontend code documentation](swainos-code-documentation-frontend.md) — App Router structure, SSR loaders
- [Sample payloads](sample-payloads.md) — example envelopes and JSON shapes
- [Terminology glossary](swainos-terminology-glossary.md) — canonical labels and field naming
- [Vercel guidelines](vercel-guidelines.md) / [Render guidelines](render-guidelines.md) — when production may lag `main`
