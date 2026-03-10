# Platform Ingestion Job Inventory v1

This is the canonical first-release inventory for the ingestion control plane cutover.
It freezes `jobKey`, `runnerKey`, schedule mode, default cadence, dependencies, and legacy deletion targets.

## Current State (Pre-Migration Execution)

- Control-plane code paths are implemented in backend and frontend.
- `0090_create_data_jobs_control_plane_v1.sql` is updated with recurring-job `next_run_at` seeding and can be run safely.
- Scheduler logic now computes true next-run values from cron expressions/timezones and bootstraps recurring jobs that still have null `next_run_at`.
- Remaining execution step: run migration `0090` in Supabase and perform post-cutover verification checks.

## Schedule Modes

- `recurring`: eligible for scheduler-driven execution.
- `manual_only`: operator-triggered only.
- `backfill_only`: explicit one-off backfill workflows.
- `system_managed`: triggered by orchestrator dependency flow, not directly by operators.

## Canonical Job Matrix

| jobKey | runnerKey | family | scheduleMode | defaultCadence | dependsOn | primarySupabaseTargets | legacyPathsToDelete |
|---|---|---|---|---|---|---|---|
| `marketing-ga4-sync` | `marketing.ga4.sync` | recurring_ingestion | recurring | `0 * * * *` | - | `marketing_web_analytics_*` facts | `POST /marketing/web-analytics/sync/run`; request-time sync in `marketing_web_analytics_service.py` |
| `marketing-gsc-sync` | `marketing.gsc.sync` | recurring_ingestion | recurring | `15 * * * *` | - | `marketing_search_console_*` facts | request-time GSC sync paths in `marketing_web_analytics_service.py` |
| `marketing-search-console-rollups-refresh` | `marketing.gsc.rollups.refresh` | downstream_rollup | system_managed | none | `marketing-ga4-sync,marketing-gsc-sync` | `search_console_*_rollup` outputs | ad hoc rollup refresh calls outside orchestrator |
| `fx-rates-pull` | `fx.rates.pull` | recurring_ingestion | recurring | `*/15 * * * *` | - | `fx_rates` | `POST /fx/rates/run`; frontend proxy `app/api/fx/rates/run/route.ts` |
| `fx-exposure-refresh` | `fx.exposure.refresh` | downstream_rollup | system_managed | none | `fx-rates-pull` | `mv_fx_exposure` refresh | standalone exposure refresh execution paths |
| `fx-signals-generate` | `fx.signals.generate` | downstream_rollup | recurring | `30 * * * *` | `fx-exposure-refresh` | `fx_signals`, `fx_signal_runs` | `POST /fx/signals/run` |
| `fx-intelligence-generate` | `fx.intelligence.generate` | recurring_ingestion | recurring | `0 */6 * * *` | `fx-rates-pull` | `fx_intelligence_items`, `fx_intelligence_runs` | `POST /fx/intelligence/run` |
| `salesforce-readonly-sync` | `salesforce.readonly.sync` | recurring_ingestion | recurring | `0 */2 * * *` | - | salesforce raw + canonical sync tables | direct script-only execution outside control plane |
| `travel-trade-rollups-refresh` | `salesforce.travel_trade.rollups.refresh` | downstream_rollup | system_managed | none | `salesforce-readonly-sync` | travel trade rollups | standalone rollup refresh execution paths |
| `consultant-ai-rollups-refresh` | `salesforce.consultant_ai.rollups.refresh` | downstream_rollup | system_managed | none | `salesforce-readonly-sync` | consultant AI rollups | standalone rollup refresh execution paths |
| `ai-insights-generate` | `ai.insights.generate` | downstream_rollup | recurring | `0 */6 * * *` | `consultant-ai-rollups-refresh` | `ai_insight_events`, AI read models | `POST /ai-insights/run` |
| `bookings-import` | `imports.bookings.upsert` | manual_import | manual_only | none | - | `bookings` + lineage | non-control-plane manual run flows |
| `customer-payments-import` | `imports.customer_payments.upsert` | manual_import | manual_only | none | - | `customer_payments` + lineage | non-control-plane manual run flows |
| `supplier-invoices-import` | `imports.supplier_invoices.upsert` | manual_import | manual_only | none | - | `supplier_invoices` + lineage | non-control-plane manual run flows |
| `supplier-invoice-bookings-import` | `imports.supplier_invoice_bookings.upsert` | manual_import | manual_only | none | `supplier-invoices-import` | `supplier_invoice_bookings` | non-control-plane manual run flows |
| `supplier-invoice-lines-import` | `imports.supplier_invoice_lines.upsert` | manual_import | manual_only | none | `supplier-invoices-import` | `supplier_invoice_lines` | non-control-plane manual run flows |
| `fx-rates-history-backfill` | `fx.rates.backfill` | maintenance_backfill | backfill_only | none | - | `fx_rates` | standalone `backfill_fx_rates_history.py` invocation |
| `salesforce-permission-validate` | `salesforce.permissions.validate` | maintenance_backfill | manual_only | none | - | validation-only diagnostics | standalone `validate_salesforce_readonly_permissions.py` invocation |
| `ai-insights-purge` | `ai.insights.purge` | maintenance_backfill | manual_only | none | - | `ai_*` retention targets | standalone `purge_ai_insights.py` invocation |
| `inactive-employees-cleanup` | `workforce.cleanup.inactive_employees` | maintenance_backfill | recurring | `0 3 * * 0` | - | employee state tables | standalone `cleanup_inactive_employees.py` invocation |
| `debt-schedule-precompute` | `debt.schedule.precompute` | maintenance_backfill | system_managed | none | - | `debt_service_schedules` | schedule generation-on-read in `debt_service_service.py` |

## Dependency Matrix

- `marketing-ga4-sync` -> `marketing-search-console-rollups-refresh`
- `marketing-gsc-sync` -> `marketing-search-console-rollups-refresh`
- `fx-rates-pull` -> `fx-exposure-refresh`
- `fx-exposure-refresh` -> `fx-signals-generate`
- `salesforce-readonly-sync` -> `travel-trade-rollups-refresh`
- `salesforce-readonly-sync` -> `consultant-ai-rollups-refresh`
- `consultant-ai-rollups-refresh` -> `ai-insights-generate`
- `supplier-invoices-import` -> `supplier-invoice-bookings-import`
- `supplier-invoices-import` -> `supplier-invoice-lines-import`

## Runner Implementation Sources

- Marketing: `scripts/sync_marketing_web_analytics.py`
- FX: `scripts/pull_fx_rates.py`, `scripts/refresh_fx_exposure.py`, `scripts/generate_fx_intelligence.py`
- Salesforce + rollups: `scripts/sync_salesforce_readonly.py`, `scripts/refresh_travel_trade_rollups.py`, `scripts/refresh_consultant_ai_rollups.py`
- AI: `scripts/generate_ai_insights.py`, `scripts/purge_ai_insights.py`
- Imports: `scripts/upsert_bookings.py`, `scripts/upsert_customer_payments.py`, `scripts/upsert_supplier_invoices.py`, `scripts/upsert_supplier_invoice_bookings.py`, `scripts/upsert_supplier_invoice_lines.py`
- Backfill/maintenance: `scripts/backfill_fx_rates_history.py`, `scripts/validate_salesforce_readonly_permissions.py`, `scripts/cleanup_inactive_employees.py`

## Big-Bang Notes

- All legacy run endpoints remain deletable only after control-plane route migration is complete.
- No job outside this matrix is schedulable in first release.
- Salesforce object-level scripts are internal runner steps for `salesforce-readonly-sync`, not top-level jobs.
