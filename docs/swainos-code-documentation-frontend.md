# SwainOS Frontend Code Documentation

## Overview
- Next.js App Router with TypeScript strict and Tailwind-first styling.
- TravelOS design patterns for layout, navigation, and UI primitives.
- Contract-first API clients using `{ data, pagination, meta }` envelopes.
- AI-forward UI with assistant entry points and context scaffolding.

## Local Runbook
- Backend
  - `cd /Users/ianswain/Desktop/SwainOS_BackEnd`
  - `uvicorn src.main:app --reload`
- Frontend
  - `cd /Users/ianswain/Desktop/SwainOS_FrontEnd/apps/web`
  - Copy `.env.local.example` to `.env.local`
  - Set `NEXT_PUBLIC_API_BASE` to your backend URL (for local: `http://127.0.0.1:8000`)
  - `npm run dev`
- Verification preflight
  - Backend health check: `GET /api/v1/health`
  - Open `http://localhost:3000` and verify redirect to `/command-center`
  - Ensure backend CORS allows `http://localhost:3000` and `http://127.0.0.1:3000` for browser fetches

## Structure
```
apps/web/src/
├── app/                    # Next.js routes
├── components/             # Shared UI + layout
├── features/               # Feature modules
├── lib/
│   ├── api/                # Service clients + http client
│   ├── assistant/          # Assistant context/types
│   ├── constants/          # Navigation config
│   ├── types/              # Domain + API envelope types
│   └── utils/              # Formatting helpers
└── public/                 # Static assets
```

## Navigation Map
Primary modules (spec-aligned):
- Command Center
- Cash Flow
- Debt Service
- Itinerary Forecast
- Itinerary Actuals
- Travel Consultant
- FX Command
- Operations
- AI Insights
- Settings

## Layout System
- `components/layout/system-shell.tsx` wraps the app with a SideNav, TopBar, and AssistantPanel.
- SideNav supports icon-labeled navigation with collapsible icon-only mode for compact layouts.
- `components/layout/page-shell.tsx` provides page titles, subtitles, and action areas.

## Data & Services
- `lib/api/httpClient.ts` handles the shared fetch logic and error envelopes.
- Service clients:
  - `cashFlowService.ts`
  - `depositsService.ts`
  - `paymentsOutService.ts`
  - `bookingForecastsService.ts`
  - `itineraryTrendsService.ts`
  - `fxService.ts` (FX rates + exposure)
  - `itineraryRevenueService.ts` (forecast outlook/deposits/conversion/channels + actuals YoY + actuals channels)
  - `travelConsultantService.ts` (leaderboard/profile/forecast with typed normalization)
- Domain types live in `lib/types/*` and align to sample payloads.
- Numeric normalization is centralized in `lib/utils/parseNumber.ts` so decimal strings from backend payloads are safely converted before UI rendering.

## Module Coverage
- Implemented with live backend data
  - `Command Center` (live KPI rollups + briefing context from a consolidated hook)
  - `Cash Flow` (summary + timeseries + deposits + payments out)
  - `Itinerary Forecast` (forward outlook, lead flow, deposit control, conversion, channel leaders, forecast grid, primary metric = Gross Profit)
  - `Itinerary Actuals` (fixed 3-year Jan-Dec YoY by `travel_end_date`: yearly KPI cards with YoY deltas, lead flow, channel production, horizontal matrix, monthly detail; primary metric = Gross Profit)
  - `Travel Consultant`:
    - `/travel-consultant`: leaderboard with domain toggles (travel vs funnel), search, sortable rankings, mobile-priority columns, highlights, and team effectiveness snapshot
    - `/travel-consultant/[employeeId]`: consultant deep-dive with effectiveness summary, expanded Hero KPIs (avg gross profit, avg itinerary nights, avg group size, avg lead time, avg speed to close), 3-year revenue matrices (travel + funnel), operational snapshot, forecast/target, compensation, signals, and deduped insight cards
- Implemented with live + optional mock
  - `FX Command` (live rates and exposure scoped to supplier currencies **ZAR, USD, AUD, NZD** only; live from DB or demo data)
- Implemented with structured UI (data pending)
  - `Debt Service` (schedule, risk watchlist scaffolds)
  - `Operations` (advisor productivity + margin panels)
  - `AI Insights` (anomaly feed, recommendations, history shells)
  - `Settings` (integrations, sync schedules, audit trail shells)

## Map Widget (Mapbox)
- Active Travelers Map scaffold lives in `features/command-center/active-travelers-map.tsx`.
- Mapbox token stored in `.env.local` as `NEXT_PUBLIC_MAPBOX_TOKEN`.
- The component currently renders a placeholder until traveler coordinates are wired.

## Assistant UX
- `lib/assistant/assistantContext.tsx` provides assistant state and toggling.
- `components/assistant/*` includes a launcher and right-side panel scaffold.

## Contract Decisions
- Revenue owner cockpit canonical query params are:
  - `time_window`
  - `grain` (`weekly` or `monthly`)
  - `currency_code` (optional)
- Itinerary actuals canonical query params:
  - `years_back` (API supports `2` to `5`; frontend defaults to fixed `3`)
  - `currency_code` (optional)
- Frontend default behavior:
  - Itinerary Actuals currently requests a fixed `3` years for consistent year-over-year comparisons.
- Income semantics:
  - UI labels use **Gross Profit** while API field `commissionIncomeAmount` remains stable for contract continuity.
  - `commissionIncomeAmount` is sourced from itinerary `gross_profit`.
- JSON properties remain `camelCase`; query params remain `snake_case`.
- Travel Consultant profile payload key additions:
  - `threeYearPerformance`
  - `ytdVariancePct`
  - `funnelHealth.avgSpeedToBookDays`
  - extended `heroKpis` set for advisor effectiveness coaching

## Organization and Simplification Notes
- `features/command-center/useCommandCenterData.ts` consolidates command center data loading into one flow to avoid scattered business-fetch logic.
- `features/command-center/kpi-grid.tsx` and `active-travelers-map.tsx` are now presentation-focused and consume live data props.
- Command-center business values are sourced from backend responses; hardcoded metrics were removed.
- Legacy revenue-booking module UIs (`itinerary-pipeline`, `itinerary-trends-overview`, and standalone booking-forecast summary card) were removed in favor of itinerary forecast + actuals modules.
- Legacy `features/revenue-bookings/*` shared forecast panel files were removed; forecast panels now live under `features/itinerary-forecast/*` and shared channel panels under `features/itinerary-shared/*`.
- `features/itinerary-forecast/itinerary-forecast-cockpit.tsx` consumes `/api/v1/itinerary-revenue/outlook|deposits|conversion|channels`.
- `features/itinerary-actuals/itinerary-actuals-page-content.tsx` consumes `/api/v1/itinerary-revenue/actuals-yoy` and `/api/v1/itinerary-revenue/actuals-channels` (closed-won production rollups).
- `features/itinerary-shared/itinerary-leads-panel.tsx` is shared by forecast and actuals for itinerary-creation lead flow visualization.
- Forecast visuals are modularized in:
  - `outlook-chart.tsx`
  - `deposit-health-panel.tsx`
  - `conversion-panel.tsx`
  - `channel-performance-panel.tsx`
  - `forecast-grid.tsx`
- Route files under `app/` are kept thin and delegate to `features/*` modules for UI logic.
- Shared UI primitives (e.g., `SectionHeader`, `MetricCard`) live in `components/ui` for reuse.

## Conventions
- Components use `kebab-case.tsx`; utilities use `camelCase.ts`.
- JSON properties are `camelCase`; API slugs are `kebab-case`.
- No `any` types, no unused imports, no dead code.
