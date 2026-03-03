# Frontend Data Queries

Purpose: canonical list of frontend-to-backend query contracts in active use.

## Conventions
- API envelope: `{ data, pagination, meta }`
- Query params: `snake_case`
- Response JSON fields: `camelCase`
- Numeric payloads are normalized in `apps/web/src/lib/utils/parseNumber.ts`
- Frontend GET cache is disabled in non-production (`NODE_ENV !== "production"`)

## Live Queries In Use

| Endpoint | Used In | Purpose |
|---|---|---|
| `GET /api/v1/cash-flow/summary` | `lib/api/cashFlowService.ts`, `features/command-center/command-center-server-loader.ts` | Net liquidity summary for command center snapshot |
| `GET /api/v1/cash-flow/risk-overview` | `lib/api/cashFlowService.ts`, `features/cash-flow/cash-flow-risk-server-loader.ts` | Decision-first risk status, first risk date, and risk drivers by currency |
| `GET /api/v1/cash-flow/forecast` | `lib/api/cashFlowService.ts`, `features/cash-flow/cash-flow-risk-server-loader.ts` | Forecast points by currency for 13-week (`3m`) and rolling 12-month (`12m`) horizons |
| `GET /api/v1/cash-flow/ap-schedule` | `lib/api/cashFlowService.ts`, `features/cash-flow/cash-flow-risk-server-loader.ts` | Upcoming AP schedule rows by due date/currency |
| `GET /api/v1/cash-flow/scenarios` | `lib/api/cashFlowService.ts`, `features/cash-flow/cash-flow-risk-server-loader.ts` | Read-only scenario compare snapshots by currency |
| `GET /api/v1/deposits/summary` | `lib/api/depositsService.ts`, `features/command-center/command-center-server-loader.ts` | Customer receivable/deposit liability posture |
| `GET /api/v1/payments-out/summary` | `lib/api/paymentsOutService.ts`, `features/command-center/command-center-server-loader.ts` | AP outstanding summary and near-term due pressure |
| `GET /api/v1/ap/summary` | `lib/api/apService.ts` | AP open-line/booking/supplier liquidity rollup |
| `GET /api/v1/ap/aging` | `lib/api/apService.ts` | AP aging buckets by currency |
| `GET /api/v1/ap/payment-calendar` | `lib/api/apService.ts` | AP payment schedule timeline |
| `GET /api/v1/booking-forecasts` | `features/command-center/command-center-server-loader.ts` | Short-horizon booking projection |
| `GET /api/v1/debt-service/overview` | `features/debt-service/debt-service-server-loader.ts`, `features/command-center/command-center-server-loader.ts` | Debt KPI overview and scheduled 30/60/90 obligations |
| `GET /api/v1/debt-service/facilities` | `features/debt-service/debt-service-server-loader.ts` | Facility list and identifiers for debt slices |
| `GET /api/v1/debt-service/schedule` | `features/debt-service/debt-service-server-loader.ts`, `features/debt-service/debt-service-page.tsx` | Amortization timeline rows |
| `GET /api/v1/debt-service/payments` | `features/debt-service/debt-service-server-loader.ts`, `features/debt-service/debt-service-page.tsx` | Logged payment ledger entries |
| `POST /api/v1/debt-service/payments` | `features/debt-service/debt-service-page.tsx` | Principal/interest posting workflow |
| `GET /api/v1/debt-service/scenarios` | `features/debt-service/debt-service-server-loader.ts` | Persisted scenario result summaries |
| `POST /api/v1/debt-service/scenarios/run` | `features/debt-service/debt-service-page.tsx` | Deterministic payoff/interest scenario compare run |
| `GET /api/v1/itinerary-lead-flow` | `features/command-center/command-center-server-loader.ts`, `features/itinerary-actuals/useItineraryActualsYoy.ts` | Lead-flow trend by booking semantics |
| `GET /api/v1/itinerary-revenue/outlook` | `features/command-center/command-center-server-loader.ts`, `features/itinerary-forecast/useItineraryForecastOutlook.ts` | Forward outlook timeline |
| `GET /api/v1/itinerary-revenue/deposits` | `features/command-center/command-center-server-loader.ts`, `features/itinerary-forecast/useItineraryForecastOutlook.ts` | Deposit health timeline |
| `GET /api/v1/itinerary-revenue/conversion` | `features/itinerary-forecast/useItineraryForecastOutlook.ts` | Conversion projections |
| `GET /api/v1/itinerary-revenue/channels` | `features/itinerary-forecast/useItineraryForecastOutlook.ts` | Consortia/trade channel performance |
| `GET /api/v1/itinerary-revenue/actuals-yoy` | `features/command-center/command-center-server-loader.ts`, `features/itinerary-actuals/useItineraryActualsYoy.ts`, `features/itinerary-forecast/useItineraryForecastOutlook.ts` | Jan-Dec matrix and current-year booked total |
| `GET /api/v1/itinerary-revenue/actuals-channels` | `features/itinerary-actuals/useItineraryActualsYoy.ts` | Actuals channel production |
| `GET /api/v1/travel-consultants/leaderboard` | `features/travel-consultant/leaderboard/useTravelConsultantLeaderboard.ts` | Consultant ranking and highlights |
| `GET /api/v1/travel-consultants/{employee_id}/profile` | `features/travel-consultant/profile/useTravelConsultantProfile.ts` | Consultant detail and embedded forecast section |
| `GET /api/v1/travel-agents/leaderboard` | `features/sales/useTravelAgentsLeaderboard.ts` | Agent ranking |
| `GET /api/v1/travel-agents/{agent_id}/profile` | `features/sales/useTravelAgentProfile.ts` | Agent profile |
| `GET /api/v1/travel-agencies/leaderboard` | `features/sales/useTravelAgenciesLeaderboard.ts` | Agency ranking |
| `GET /api/v1/travel-agencies/{agency_id}/profile` | `features/sales/useTravelAgencyProfile.ts` | Agency profile |
| `GET /api/v1/travel-trade/search` | `features/sales/useTravelTradeSearch.ts` | Unified trade search |
| `GET /api/v1/itinerary-destinations/summary` | `features/sales/destination-server-loader.ts`, `features/sales/destination-page.tsx` | Current-year booked destination KPI summary and top-country ranking |
| `GET /api/v1/itinerary-destinations/trends` | `features/sales/destination-server-loader.ts`, `features/sales/destination-page.tsx` | Monthly destination trend by selected country/city scope |
| `GET /api/v1/itinerary-destinations/breakdown` | `features/sales/destination-server-loader.ts`, `features/sales/destination-page.tsx` | Country and city booked production breakdown |
| `GET /api/v1/itinerary-destinations/matrix` | `features/sales/destination-server-loader.ts`, `features/sales/destination-page.tsx` | Horizontal Jan-Dec country/city matrix with monthly and annual YoY for Gross Revenue, Booked Cost, and Gross Profit |
| `GET /api/v1/ai-insights/briefing` | `features/command-center/command-center-server-loader.ts`, `features/ai-insights/useAiInsightsData.ts` | Daily briefing |
| `GET /api/v1/ai-insights/feed` | `features/ai-insights/useAiInsightsData.ts` | AI events feed |
| `GET /api/v1/ai-insights/recommendations` | `features/ai-insights/useAiInsightsData.ts`, consultant AI hooks | Recommendation queue |
| `PATCH /api/v1/ai-insights/recommendations/{id}` | `features/ai-insights/useAiInsightsData.ts` | Recommendation status transition |
| `GET /api/v1/ai-insights/history` | `features/ai-insights/useAiInsightsData.ts` | Historical AI events |
| `GET /api/v1/ai-insights/entities/{entity_type}/{entity_id}` | `features/travel-consultant/profile/useTravelConsultantEntityAi.ts`, `components/assistant/assistant-panel.tsx` | Entity-scoped AI context |
| `GET /api/v1/fx/rates` | `app/fx-command/page.tsx`, `features/fx-command/useFxCommandData.ts` | FX rate series and metadata |
| `GET /api/v1/fx/exposure` | `app/fx-command/page.tsx`, `features/fx-command/useFxCommandData.ts` | Exposure by currency |
| `GET /api/v1/fx/signals` | `app/fx-command/page.tsx`, `features/fx-command/useFxCommandData.ts` | Buy/wait signals |
| `GET /api/v1/fx/holdings` | `app/fx-command/page.tsx`, `features/fx-command/useFxCommandData.ts` | Holdings snapshot |
| `GET /api/v1/fx/transactions` | `app/fx-command/page.tsx`, `features/fx-command/useFxCommandData.ts` | Ledger feed |
| `POST /api/v1/fx/transactions` | `features/fx-command/fx-command-page.tsx` | Ledger write path |
| `GET /api/v1/fx/intelligence` | `app/fx-command/page.tsx`, `features/fx-command/useFxCommandData.ts` | Intelligence feed |
| `GET /api/v1/fx/invoice-pressure` | `lib/api/fxServerService.ts`, `lib/api/fxService.ts`, `features/fx-command/useFxCommandData.ts` | Due-window payable pressure sourced from AP rollups |
| `GET /api/v1/marketing/web-analytics/overview` | `lib/api/marketingService.ts`, `features/marketing/marketing-server-loader.ts` | Strategic web analytics KPIs and trend context from GA4 |
| `GET /api/v1/marketing/web-analytics/page-activity` | `lib/api/marketingService.ts`, `features/marketing/marketing-server-loader.ts` | Page-level behavior (best/worst pages, itinerary-focused breakdown, quality scoring) |
| `GET /api/v1/marketing/web-analytics/geo` | `lib/api/marketingService.ts`, `features/marketing/marketing-server-loader.ts` | Geographic performance by country/region/city |
| `GET /api/v1/marketing/web-analytics/events` | `lib/api/marketingService.ts`, `features/marketing/marketing-server-loader.ts` | Event catalog with plain-language definitions and conversion classification |
| `GET /api/v1/marketing/web-analytics/search` | `lib/api/marketingService.ts`, `features/marketing/marketing-server-loader.ts` | Search and landing-page opportunity surface (GA4-first; GSC-aware) |
| `GET /api/v1/marketing/web-analytics/ai-insights` | `lib/api/marketingService.ts`, `features/marketing/marketing-server-loader.ts` | Ranked marketer/sales recommendations from website behavior signals |
| `POST /api/fx/rates/run` (frontend route) | `features/fx-command/useFxCommandData.ts` | Server-side proxy for manual FX pull token handling |

## Backend Endpoints Not Used By Current Frontend Surfaces

| Endpoint | Note |
|---|---|
| `GET /api/v1/debt-service/covenants` | Endpoint is live; current debt page surfaces aggregate covenant status from overview and does not yet render per-covenant snapshot rows. |
| `GET /api/v1/travel-consultants/{employee_id}/forecast` | Profile endpoint includes forecast section; standalone endpoint is not called by current UI. |
| `POST /api/v1/ai-insights/run` | Manual/operator trigger; no direct frontend action in module UI. |
| `POST /api/v1/fx/signals/run` | Manual/operator trigger; no direct frontend action in module UI. |
| `POST /api/v1/fx/intelligence/run` | Manual/operator trigger; no direct frontend action in module UI. |
| `POST /api/v1/marketing/web-analytics/sync/run` | Manual/operator trigger; current Marketing UI auto-syncs via backend read path and does not expose a direct run action yet. |
| `GET /api/v1/marketing/web-analytics/health` | Endpoint remains available for operator diagnostics, but the current Marketing UI intentionally omits the dedicated Tracking Health tab. |

## Notes
- Itinerary revenue surfaces use `grossProfitAmount` as canonical Gross Profit metric.
- Itinerary actuals and channel rollups are aligned to travel-period reporting.
- Itinerary lead-flow panel is present on command center and itinerary actuals; itinerary forecast does not render lead-flow.
- Destination matrix API still carries passenger fields, but destination UI intentionally hides passenger metrics to avoid itinerary-item duplication inflation.
- Debt Service payment posting is user-confirmed from a prefilled prompt; frontend does not auto-post when opening the log-payment flow.
- Cash Flow module is split into subpages (`/cash-flow`, `/cash-flow/forecast`, `/cash-flow/ap-schedule`, `/cash-flow/scenarios`) and uses risk-first contracts before detail tables.
- Cash Flow renders per-currency risk/forecast/schedule rows and avoids mixed-currency totals labeled as a single currency.
- Marketing Web Analytics is GA4-first in v1; Search Console query datasets remain intentionally deferred until `GOOGLE_GSC_SITE_URL` is configured.

