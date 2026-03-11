# Platform Ingestion Big-Bang Cutover Checklist

Use this checklist as the final go/no-go gate for one-time cutover.

## Current State Snapshot

- Implementation status: control-plane backend/frontend wiring is complete.
- Migration status: control-plane and follow-on runtime migrations are applied in Supabase (`0090`, `0091`, `0092`, `0093`).
- Scheduler readiness update: cron/timezone next-run computation, recurring null-`next_run_at` bootstrap, stale-run expiration, conflict blocking, and retry backoff are active.

## Global Gates

- [x] Canonical control-plane schema exists (`data_jobs`, dependencies, runs, steps, health).
- [x] `0090_create_data_jobs_control_plane_v1.sql` has been executed in target Supabase environment.
- [x] Canonical backend API family exists (`/api/v1/data-jobs*`, `/api/v1/data-job-runs/{run_id}`).
- [x] Scheduler tick endpoint exists (`POST /api/v1/data-jobs/scheduler/tick`).
- [x] Settings and Operations surfaces are wired to control-plane endpoints.
- [x] Legacy manual-run endpoints are removed (`/marketing/web-analytics/sync/run`, `/fx/*/run`, `/ai-insights/run`).
- [x] Frontend FX proxy route is removed.

## Domain Deletion Checklist

### Marketing
- [x] New runner path registered (`marketing.ga4.sync`, `marketing.gsc.sync`, rollup refresh).
- [x] Legacy run endpoint removed (`/marketing/web-analytics/sync/run`).
- [x] Request-time sync triggers removed.
- [x] Docs updated to canonical data-jobs trigger path.

### FX
- [x] New runner path registered (`fx.rates.pull`, `fx.exposure.refresh`, `fx.signals.generate`, `fx.intelligence.generate`).
- [x] Legacy run endpoints removed (`/fx/rates/run`, `/fx/signals/run`, `/fx/intelligence/run`).
- [x] Legacy frontend proxy removed (`app/api/fx/rates/run/route.ts`).
- [x] FX frontend manual run now calls `/api/v1/data-jobs/fx-rates-pull/runs`.

### AI
- [x] New runner path registered (`ai.insights.generate`, `ai.insights.purge`).
- [x] Legacy run endpoint removed (`/ai-insights/run`).
- [x] AI frontend manual run now calls `/api/v1/data-jobs/ai-insights-generate/runs`.

### Salesforce + Rollups
- [x] Parent sync runner registered (`salesforce.readonly.sync`).
- [x] Rollup runners registered (`salesforce.travel_trade.rollups.refresh`, `salesforce.consultant_ai.rollups.refresh`).
- [x] Dependency chain enforced in control-plane dependency table.

### Debt Service
- [x] Precompute runner registered (`debt.schedule.precompute`).
- [x] Schedule generation-on-read removed from service read path.
- [x] Payment posting path now requires precomputed schedule presence.

### Imports + Backfills
- [x] Import runners registered under `imports.*`.
- [x] Maintenance/backfill runners registered under `fx.*`, `salesforce.*`, `workforce.*`, `ai.*`.

## Post-Cutover Verification

- [x] Execute one scheduler tick in staging and confirm only due recurring jobs dispatch.
- [x] Verify dependency-blocked jobs produce `blocked` runs with readable reasons.
- [x] Trigger manual runs from `/settings` for at least one job per domain.
- [x] Verify `/operations` renders health + run history with live statuses.
- [x] Verify no removed endpoint is reachable in staging.
