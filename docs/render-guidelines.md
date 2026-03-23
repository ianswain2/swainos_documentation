# SwainOS Render Guidelines

> **Version**: v1.1
> **Status**: Active standard
> **Date**: 2026-03-23

## Purpose

This document defines the deployment, security, runtime, and cost-control standards for hosting SwainOS services on Render.

These guidelines are designed for a small, private, invite-only product with strict billing discipline and an offline-first rollout model.

## Scope

These guidelines apply to:

- SwainOS backend service under `SwainOS_BackEnd`
- optional SwainOS frontend hosting on Render if platform consolidation is chosen
- Render environment-variable handling
- Render health checks, scaling, and deployment behavior
- Render billing and usage controls

## Core Principles

- Keep setup simple and predictable over platform cleverness.
- Prefer fixed-size services and explicit scaling limits to avoid runaway usage.
- Treat Render as service hosting, not the primary security boundary.
- Keep Cloudflare as the public edge and DNS authority for `swainos.com`.
- Keep Supabase Auth as the user identity boundary.
- Separate frontend-safe config from backend secrets with strict ownership.
- **Repository vs runtime:** commits on `main` do not automatically mean production Render is running or serving latest code—suspend/resume, failed deploys, and held releases all create gaps. Pair Render state with Vercel (frontend) and Supabase (migrations) when validating behavior.

## Intended Role In The Stack

Render is responsible for:

- running the FastAPI backend as a managed web service
- exposing health endpoints for zero-downtime-safe deploy checks
- hosting optional frontend if consolidation is chosen
- handling service-level runtime configuration and logs

Render is not responsible for:

- primary DNS authority
- edge WAF and country restrictions (Cloudflare role)
- application-level authz policy
- replacing database RLS and Supabase access controls

## Architecture Policy

Current default architecture:

- frontend host: Vercel (`app.swainos.com`)
- backend host: Render (`api.swainos.com`)

Allowed consolidation option:

- Render hosts both frontend and backend only if explicitly approved

Rule:

- frontend and backend remain separate Render services even when both are on Render
- do not combine both runtimes into one service

## Service Naming Standards

- Backend service name: `swainos-api`
- Optional frontend service name: `swainos-web`
- Keep names durable and meaning-based; avoid temporary suffixes

## Runtime Standards (Backend / FastAPI)

Service root:

- repository root should map to `SwainOS_BackEnd`

Required app command model:

- start command should run FastAPI app at `src.main:app`
- bind to `0.0.0.0` and Render-provided `$PORT`

Recommended start command:

- `uvicorn src.main:app --host 0.0.0.0 --port $PORT`

Health checks:

- liveness endpoint: `/healthz`
- readiness endpoint: `/health/ready`

Deployment behavior:

- require health checks before routing traffic to new instances
- if health checks fail, keep prior healthy instance serving
- do not bypass failed health checks for convenience

## Environment Variable Standards

Backend required variables must include:

- `ENVIRONMENT=production` (for production service only)
- `TRUSTED_HOSTS` including non-local production hosts
- `SUPABASE_URL`
- `SUPABASE_SERVICE_ROLE_KEY` (preferred) or `SUPABASE_ANON_KEY`
- `AI_MANUAL_RUN_TOKEN`
- `FX_MANUAL_RUN_TOKEN`
- `DATA_JOBS_SCHEDULER_TOKEN`

Feature-dependent backend variables:

- AI model/runtime keys (`OPENAI_*`)
- FX provider keys (`FX_*`, `MACRO_*`, `NEWS_*`)
- marketing ingestion keys (`GOOGLE_*`, `MARKETING_*`)

Rules:

- Never store secrets in source control.
- Never place backend secrets in frontend projects.
- Keep production values separate from development values.
- Add only required keys; remove stale keys after integration retirement.

## Security Standards

- Restrict public access via Cloudflare-first routing.
- Keep direct origin URLs non-public in docs and operator workflow.
- Enforce HTTPS end-to-end with Cloudflare + Render TLS.
- Keep production host allowlists (`TRUSTED_HOSTS`) accurate.
- Apply app-layer token gates for expensive/manual run routes.

## Scaling And Cost-Control Standards

For setup and low-volume production:

- start backend on `Starter` web service instance unless load proves otherwise
- keep instance count fixed to `1` during setup/offline phases
- do not enable autoscaling until baseline traffic and costs are measured
- do not enable additional paid services unless tied to explicit need

Build/deploy discipline:

- no production deploys without explicit gate approval
- avoid frequent no-op deploys during setup
- keep preview environments disabled unless explicitly needed

Operational monitoring:

- watch service restarts and health-check failures
- track traffic spikes on expensive run endpoints
- monitor monthly usage before changing instance class

## Recommended Plan For SwainOS Goals

For a small invite-only workload with strong cost discipline:

- **Workspace**: start on `Hobby` if a single operator and minimal collaboration are acceptable
- **Backend web service**: `Starter` (`$7/month` baseline compute)
- move to `Professional` workspace only when you need collaboration features (protected environments, preview environments, team workflow, higher included bandwidth)

Guiding rule:

- scale plan/features only when there is an observed operational need, not in anticipation

## Cloudflare Coordination Standards

- Render services should be reachable through Cloudflare hostnames, not raw service URLs.
- DNS cutover timing is controlled by offline-first rollout gates.
- Keep `api.swainos.com` routing disabled until backend service passes readiness checks.
- Apply Cloudflare WAF/rate limits before go-live routing.

## Operations And Verification

After any backend hosting configuration change, verify:

- `/healthz` returns healthy
- `/health/ready` validates dependencies
- auth-protected routes still enforce expected behavior
- expensive run routes still require expected tokens
- Cloudflare routing and TLS behavior remain correct

## Minimum Production Checklist

- Render backend service exists as `swainos-api`
- start command uses `src.main:app` and `$PORT`
- health check path is configured to `/health/ready`
- required production env vars are present
- service is reachable only through intended Cloudflare hostname
- Cloudflare security baseline is enabled before public cutover
- instance sizing and scaling settings match current traffic reality
- no unapproved autoscaling or surprise paid add-ons are active

## Suspended service and resume (reversal startup) — checklist

Use when bringing the API back after **suspend**, failed deploy, or long idle period. Execute only when deliberately going live; do not treat as routine.

1. **Supabase:** confirm required migrations are applied on the target project (especially MV/RPC changes referenced in `swainos-code-documentation-backend.md`). Run or schedule rollup refresh jobs if stale.
2. **Render:** resume/unsuspend `swainos-api` (or equivalent); wait for deploy to **live** with passing `/health/ready`.
3. **Environment:** verify production env vars (tokens, `TRUSTED_HOSTS`, Supabase keys) match the intended project—no drift between “docs” and dashboard.
4. **DNS / Cloudflare:** confirm `api.swainos.com` points at the intended target and matches proxy vs DNS-only decision (see `cloudflare-guidelines.md`).
5. **Smoke:** `GET /healthz`, `GET /health/ready`, then authenticated `GET /api/v1/auth/me` from the app; spot-check one analytics read used by Command Center.
6. **Frontend:** ensure Vercel production has `NEXT_PUBLIC_API_BASE` aimed at the live API and that login + Turnstile (if enabled) still work end-to-end.

## When code is merged but not yet deployed

- Treat `main` as the **contract** source; production may lag until Render (and Vercel) promote a build.
- Document any intentional **deploy hold** in the action log; operators should not assume feature parity until both app and API deploys complete.

## Related documentation

- [Backend code documentation](swainos-code-documentation-backend.md)
- [Vercel guidelines](vercel-guidelines.md) — paired frontend deploy state
- [Cloudflare guidelines](cloudflare-guidelines.md)
