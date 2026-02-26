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
| `GET /api/v1/cash-flow/summary` | `features/command-center/useCommandCenterData.ts`, `features/cash-flow/cash-flow-dashboard.tsx` | Cash totals |
| `GET /api/v1/cash-flow/timeseries` | `features/cash-flow/cash-flow-dashboard.tsx` | Cash trend series |
| `GET /api/v1/deposits/summary` | `features/command-center/useCommandCenterData.ts`, `features/deposits/deposits-summary.tsx` | Deposit totals |
| `GET /api/v1/payments-out/summary` | `features/command-center/useCommandCenterData.ts`, `features/payments-out/payments-out-summary.tsx` | Supplier invoice totals |
| `GET /api/v1/booking-forecasts` | `features/command-center/useCommandCenterData.ts` | Short-horizon booking projection |
| `GET /api/v1/itinerary-trends` | `features/command-center/useCommandCenterData.ts` | Itinerary trend context for command center |
| `GET /api/v1/itinerary-lead-flow` | `features/command-center/useCommandCenterData.ts`, `features/itinerary-actuals/useItineraryActualsYoy.ts` | Lead-flow trend by booking semantics |
| `GET /api/v1/itinerary-revenue/outlook` | `features/itinerary-forecast/useItineraryForecastOutlook.ts` | Forward outlook timeline |
| `GET /api/v1/itinerary-revenue/deposits` | `features/itinerary-forecast/useItineraryForecastOutlook.ts` | Deposit health timeline |
| `GET /api/v1/itinerary-revenue/conversion` | `features/itinerary-forecast/useItineraryForecastOutlook.ts` | Conversion projections |
| `GET /api/v1/itinerary-revenue/channels` | `features/itinerary-forecast/useItineraryForecastOutlook.ts` | Consortia/trade channel performance |
| `GET /api/v1/itinerary-revenue/actuals-yoy` | `features/itinerary-actuals/useItineraryActualsYoy.ts`, `features/itinerary-forecast/useItineraryForecastOutlook.ts` | Jan-Dec matrix and current-year booked total |
| `GET /api/v1/itinerary-revenue/actuals-channels` | `features/itinerary-actuals/useItineraryActualsYoy.ts` | Actuals channel production |
| `GET /api/v1/travel-consultants/leaderboard` | `features/travel-consultant/leaderboard/useTravelConsultantLeaderboard.ts` | Consultant ranking and highlights |
| `GET /api/v1/travel-consultants/{employee_id}/profile` | `features/travel-consultant/profile/useTravelConsultantProfile.ts` | Consultant detail and embedded forecast section |
| `GET /api/v1/travel-agents/leaderboard` | `features/sales/useTravelAgentsLeaderboard.ts` | Agent ranking |
| `GET /api/v1/travel-agents/{agent_id}/profile` | `features/sales/useTravelAgentProfile.ts` | Agent profile |
| `GET /api/v1/travel-agencies/leaderboard` | `features/sales/useTravelAgenciesLeaderboard.ts` | Agency ranking |
| `GET /api/v1/travel-agencies/{agency_id}/profile` | `features/sales/useTravelAgencyProfile.ts` | Agency profile |
| `GET /api/v1/travel-trade/search` | `features/sales/useTravelTradeSearch.ts` | Unified trade search |
| `GET /api/v1/ai-insights/briefing` | `features/command-center/useAiBriefing.ts`, `features/ai-insights/useAiInsightsData.ts` | Daily briefing |
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
| `GET /api/v1/fx/invoice-pressure` | `app/fx-command/page.tsx`, `features/fx-command/useFxCommandData.ts` | Due-window payable pressure |
| `POST /api/fx/rates/run` (frontend route) | `features/fx-command/useFxCommandData.ts` | Server-side proxy for manual FX pull token handling |

## Backend Endpoints Not Used By Current Frontend Surfaces

| Endpoint | Note |
|---|---|
| `GET /api/v1/travel-consultants/{employee_id}/forecast` | Profile endpoint includes forecast section; standalone endpoint is not called by current UI. |
| `POST /api/v1/ai-insights/run` | Manual/operator trigger; no direct frontend action in module UI. |
| `POST /api/v1/fx/signals/run` | Manual/operator trigger; no direct frontend action in module UI. |
| `POST /api/v1/fx/intelligence/run` | Manual/operator trigger; no direct frontend action in module UI. |

## Notes
- Itinerary revenue surfaces use `grossProfitAmount` as canonical Gross Profit metric.
- Itinerary actuals and channel rollups are aligned to travel-period reporting.
- Itinerary lead-flow panel is present on command center and itinerary actuals; itinerary forecast does not render lead-flow.

