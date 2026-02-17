# Frontend Data Queries (Current)

> Purpose: Single source of truth for all API queries used by the frontend UI.

## Table of Contents
- [Conventions](#conventions)
- [Live Queries In Use](#live-queries-in-use)
- [Backend Endpoints Not Consumed by UI](#backend-endpoints-not-consumed-by-ui)
- [Notes for Review](#notes-for-review)

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
| `GET /api/v1/travel-consultants/leaderboard` | Consultant leaderboard for travel outcomes or funnel views | `period_type` (`monthly`/`year`/`rolling12`), `domain`, `year`, `month`, `sort_by`, `sort_order`, `currency_code` | `features/travel-consultant/leaderboard/useTravelConsultantLeaderboard.ts` | Powers `/travel-consultant` ranking table, consultant search/select, and highlights |
| `GET /api/v1/travel-consultants/{employee_id}/profile` | Consultant profile story sections with YoY context (including forecast section) | `period_type` (`monthly`/`year`/`rolling12`), `year`, `month`, `yoy_mode`, `currency_code` | `features/travel-consultant/profile/useTravelConsultantProfile.ts` | Powers profile sections and forecast/target timeline from a single request |
| `GET /api/v1/travel-consultants/{employee_id}/forecast` | Consultant forecast timeline and target gap | `horizon_months`, `currency_code` | Not consumed by current frontend UI | Available backend endpoint; UI now uses profile-embedded forecast payload |
| `GET /api/v1/travel-agents/leaderboard` | Trade travel-agent leaderboard (default Gross Profit current year) | `period_type` (`monthly`/`year`/`rolling12`), `year`, `month`, `top_n`, `sort_by`, `sort_order`, `currency_code` | `features/sales/useTravelAgentsLeaderboard.ts` | Powers `/travel-agencies` travel-agent ranking views + top-performance bars |
| `GET /api/v1/travel-agents/{agent_id}/profile` | Travel-agent profile (KPIs, YoY, consultant affinity, operational files) | `period_type` (`monthly`/`year`/`rolling12`), `year`, `month`, `top_n`, `currency_code` | `features/sales/useTravelAgentProfile.ts` | Powers `/travel-agencies/agents/[agentId]` |
| `GET /api/v1/travel-agencies/leaderboard` | Trade travel-agency leaderboard (default Gross Profit current year) | `period_type` (`monthly`/`year`/`rolling12`), `year`, `month`, `top_n`, `sort_by`, `sort_order`, `currency_code` | `features/sales/useTravelAgenciesLeaderboard.ts` | Powers `/travel-agencies` travel-agency ranking views + top-performance bars |
| `GET /api/v1/travel-agencies/{agency_id}/profile` | Travel-agency profile (KPIs, YoY, top agents) | `period_type` (`monthly`/`year`/`rolling12`), `year`, `month`, `top_n`, `currency_code` | `features/sales/useTravelAgencyProfile.ts` | Powers `/travel-agencies/agencies/[agencyId]` |
| `GET /api/v1/travel-trade/search` | Unified fuzzy/full-text travel trade search | `q`, `entity_type` (`all`/`agent`/`agency`), `limit` | `features/sales/useTravelTradeSearch.ts` | Single search bar for agent/agency/email/IATA/host lookup |
| `GET /api/v1/ai-insights/briefing` | Daily command-center AI briefing | `briefing_date` (optional) | `features/command-center/useAiBriefing.ts`, `features/ai-insights/useAiInsightsData.ts` | Source table: `ai_briefings_daily` |
| `GET /api/v1/ai-insights/feed` | Filterable AI event feed | `domain`, `insight_type`, `severity`, `status`, `entity_type`, `entity_id`, `page`, `page_size`, `include_totals` | `features/ai-insights/useAiInsightsData.ts`, `features/travel-consultant/leaderboard/useTravelConsultantTeamAiRecommendations.ts` | Source table: `ai_insight_events`; exact total counts are opt-in (`include_totals=true`) |
| `GET /api/v1/ai-insights/recommendations` | Prioritized recommendation queue | `domain`, `status`, `priority_min`, `priority_max`, `owner_user_id`, `entity_type`, `entity_id`, `page`, `page_size`, `include_totals` | `features/ai-insights/useAiInsightsData.ts`, `features/travel-consultant/leaderboard/useTravelConsultantTeamAiRecommendations.ts`, `features/travel-consultant/profile/useTravelConsultantEntityAi.ts` | Source table: `ai_recommendation_queue`; exact total counts are opt-in (`include_totals=true`) |
| `PATCH /api/v1/ai-insights/recommendations/{id}` | Recommendation lifecycle transition | request body: `status`, `owner_user_id`, `resolution_note` | `features/ai-insights/useAiInsightsData.ts` | Status contract: `new`, `acknowledged`, `in_progress`, `resolved`, `dismissed` |
| `GET /api/v1/ai-insights/history` | Historical AI event archive | `domain`, `insight_type`, `status`, `date_from`, `date_to`, `page`, `page_size`, `include_totals` | `features/ai-insights/useAiInsightsData.ts` | Source table: `ai_insight_events`; exact total counts are opt-in (`include_totals=true`) |
| `GET /api/v1/ai-insights/entities/{entity_type}/{entity_id}` | Entity-scoped insights (advisor/module context) | none | `features/travel-consultant/profile/useTravelConsultantEntityAi.ts`, `components/assistant/assistant-panel.tsx` | Entity anchor for advisor-level AI cards and assistant context |
| `POST /api/v1/ai-insights/run` | Manual on-demand generation trigger | Header: `x-ai-run-token` | No frontend route wired yet | Manual trigger is default until live sync automation is enabled; token must match backend `AI_MANUAL_RUN_TOKEN` |
| `GET /api/v1/fx/rates` | Live FX rates | `limit` | `app/fx-command/page.tsx` | Filtered to ZAR/USD/AUD/NZD pairs server-side |
| `GET /api/v1/fx/exposure` | Exposure rollup | none | `app/fx-command/page.tsx` | Returns empty if view missing; UI falls back to demo exposure |

---

## Backend Endpoints Not Consumed by UI

| Endpoint | Status | Notes |
|---|---|---|
| `GET /api/v1/revenue-bookings` | Deprecated and removed | Replaced by `/api/v1/itinerary-revenue/*` owner cockpit endpoints. |
| `GET /api/v1/revenue-bookings/{booking_id}` | Deprecated and removed | Replaced by owner cockpit and module-specific detail surfaces. |
| `GET /api/v1/itinerary-pipeline` | Deprecated and removed | Replaced by `/api/v1/itinerary-revenue/outlook` and `/conversion`. |
| `GET /api/v1/travel-consultants/{employee_id}/forecast` | Not consumed by current UI | Profile screen now uses embedded forecast from `/profile`; standalone forecast endpoint retained for API consumers. |

---

## Notes for Review
- All queries are sourced from `lib/api/*Service.ts` with typed mappings in `lib/types/*`.
- Itinerary revenue surfaces use `grossProfitAmount` as the primary Gross Profit metric, sourced from itinerary `gross_profit`.
- Closed-won groupings behind itinerary revenue actuals/channels use the allowlist statuses:
  `Deposited/Confirming`, `Amendment in Progress`, `Pre-Departure`, `eDocs Sent`, `Traveling`, `Traveled`, `Cancel Fees`.
- Numbers returned as strings are normalized in `lib/utils/parseNumber.ts` before rendering.
- Command Center uses staged loading (`primary` then `secondary`) so top sections render first while lower-priority sections load in the background.
- Travel Consultant profile now uses a single `/profile` request for both story sections and forecast/target timeline (duplicate `/forecast` UI call removed).
- Travel Consultant profile contract now includes `ytdVariancePct`, `threeYearPerformance` (`travelClosedFiles` + `leadFunnel`), and `funnelHealth.avgSpeedToBookDays`.
- Leaderboard contract now uses `avgSpeedToBookDays` (replacing prior median naming).
- AI Insights generation is manual-on-demand by default via backend script/API trigger; scheduled automation is deferred until live sync cadence is enabled.
- Employee-level consultant analytics opt-out is controlled by backend column `employees.analysis_disabled`; once toggled, rerun consultant/AI rollup refresh so consultant-focused surfaces reflect exclusion while company revenue/forecast reporting remains intact.
- Travel-trade endpoints are sourced from split rollups: lead metrics from `travel_trade_lead_monthly_rollup` and booked-production metrics from `travel_trade_booked_itinerary_monthly_rollup`, merged into `travel_agent_monthly_rollup` and `travel_agency_monthly_rollup`.

