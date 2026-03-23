# SwainOS Vercel Guidelines

> **Version**: v1.1
> **Status**: Active standard
> **Date**: 2026-03-23

## Purpose

This document defines the deployment, security, environment, and cost-control standards for hosting the SwainOS frontend on Vercel.

Vercel is the preferred frontend hosting platform for the SwainOS Next.js application when fast deployment, minimal operational overhead, and strong Next.js support are the priority. These guidelines aim to preserve security and predictability for a small, private, invite-only user base.

## Scope

These guidelines apply to:

- the SwainOS frontend under `SwainOS_FrontEnd/apps/web`
- Vercel project configuration for production and preview environments
- Vercel domains assigned to `swainos.com` and related hostnames
- Vercel environment-variable handling
- Vercel billing, deployment, and operational safeguards

## Core Principles

- Vercel hosts the frontend application, not the full trust boundary of the system.
- Cloudflare remains the public edge and DNS authority for `swainos.com`.
- Supabase Auth remains the primary user authentication system.
- Backend APIs remain separate from the Vercel-hosted frontend unless explicitly documented otherwise.
- Production should favor simple, stable deployment behavior over clever platform-specific tricks.
- Avoid accidental metered usage by default.
- Every environment variable, domain, and deployment target should have a clear purpose.

## Intended Role In The Stack

Vercel is responsible for:

- building and serving the Next.js frontend
- handling production and preview deployments
- serving static assets and app-rendered frontend responses
- exposing the frontend at the approved production hostname

Vercel is not responsible for:

- primary DNS authority
- application identity management
- backend data jobs
- database access control
- replacing Cloudflare WAF and rate limiting

## Project And Naming Standards

- The Vercel project should map to the SwainOS frontend app only.
- The project name should be durable and obvious, such as `swainos-web`.
- Avoid vague names like `frontend2`, `new-web`, or `prod-ui`.
- One production Vercel project should correspond to one canonical production frontend.

Environment naming rules:

- `production` means the live SwainOS frontend
- `preview` means branch or pull-request deployments
- `development` remains local-only and should not be treated as hosted staging

## Domain Standards

- Production frontend hostname is `app.swainos.com`.
- `swainos.com` should not be treated as the canonical frontend hostname.
- Root-domain behavior should be explicit and documented as:
  - redirecting to `swaindestinations.com`, or
  - serving a minimal noindex holding page outside the main app flow until redirect cutover
- `www` handling should be explicit and documented.
- Preview deployments should use Vercel-generated preview domains unless there is a strong reason to expose custom preview hostnames.
- Preview URLs should never become the canonical public URL for production users.

## Environment Variable Standards

Only the environment variable names required by the frontend should be configured in Vercel.

Expected frontend-facing variables include:

- `NEXT_PUBLIC_API_BASE`
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `NEXT_PUBLIC_MAPBOX_TOKEN` when needed
- `NEXT_PUBLIC_TURNSTILE_SITE_KEY` when Cloudflare Turnstile is required on `/login` (must align with Supabase Auth CAPTCHA / Turnstile settings; removing the key disables the widget and server-side captcha requirement)

Server-only variables should be used only if the frontend app explicitly needs them at build time or during secure server execution.

Rules:

- Never store secrets in source control.
- Never put backend-only secrets in `NEXT_PUBLIC_*` variables.
- Keep production, preview, and local values intentionally separated.
- Remove stale environment variables when integrations are retired.

## Security Standards

- Production deploys should only be reachable through approved hostnames.
- HTTPS must be enforced for all production traffic.
- Authentication and authorization decisions must not rely on Vercel deployment protection alone.
- Application access remains invite-only through Supabase Auth and app-level enforcement.
- Vercel protection features may supplement access control, but must not replace the app's own auth model.
- Generated Vercel deployment URLs should be treated as real exposure surfaces, not harmless implementation details.

## Deployment Protection Standards

Use Vercel deployment protection as a supplemental guardrail for non-production environments.

Recommended usage:

- protect preview and generated deployment URLs by default unless there is a clear reason not to
- use customer-facing production domains intentionally and keep them separate from deployment URLs
- do not rely on deployment protection as the only safeguard for production

Protection model note:

- Vercel-generated deployment URLs are publicly accessible by default unless protected.
- `Standard Protection` is the preferred default for most SwainOS use cases because it protects preview and generated deployment URLs without forcing the production domain private.
- `All Deployments` should only be enabled if the production domain itself is intentionally meant to be private.
- Advanced Deployment Protection features may require an additional paid add-on on Pro.

## Cost-Control And Spend Management Standards

Vercel usage must be treated as metered infrastructure.

Required safeguards:

- enable Spend Management on the Vercel team
- set a deliberate spend threshold
- enable notifications for threshold crossings
- enable the option to pause production deployments if the threshold is reached

Important note:

- Spend Management helps limit runaway cost, but it does not prevent every bad application behavior instantly.
- Spend checks are periodic, not per-request circuit breakers.

## Runaway Usage Prevention

Vercel does not automatically solve request loops caused by application code.

SwainOS frontend standards must therefore minimize unnecessary server calls:

- avoid accidental polling
- avoid repeated SSR fetches for the same data in the same render path
- prefer stable caching for safe GET requests
- use explicit cache bypass only when fresh data is required
- avoid recursive or self-triggering route behavior
- validate auth redirects carefully to prevent bounce loops

The frontend's API client and route loaders should be treated as cost-sensitive code paths.

Generated-URL fetch rule:

- Do not use `VERCEL_URL`, branch URLs, or generated deployment URLs as the default internal fetch target for user-facing requests.
- Prefer relative client-side fetches and request-origin-aware server-side fetches so deployment protection and domain routing behave correctly.

Polling and refresh guidance:

- user-visible polling surfaces should use adaptive intervals (faster only when active work is running)
- background polling should pause when tabs are hidden
- prefer bounded refresh loops over fixed high-frequency polling

## Rendering And Data-Fetching Principles

- Prefer simple server-first loaders already used by the codebase.
- Avoid adding server-side work to layouts or shared routes without clear need.
- Avoid expensive revalidation patterns for pages used by a tiny invite-only user base.
- Only use dynamic rendering where it materially improves the product.
- Keep preview deployments representative, but not overly expensive.

## Preview Deployment Standards

- Preview deploys are for validation, not permanent internal environments.
- Preview env vars should never accidentally point at unsafe production-only secrets.
- Preview URLs should not be indexed or treated as public app entry points.
- Use preview deploys for QA, not as a substitute for stable production routing.
- If preview deployments can expose live or sensitive data, they should remain protected by default.

## Cloudflare Coordination Standards

- Cloudflare should front the production domain.
- DNS cutovers should follow provider-recommended Vercel integration patterns.
- Do not make conflicting redirect or caching decisions independently in both Cloudflare and Vercel without documenting precedence.
- When behavior is edge-sensitive, prefer Cloudflare as the public-traffic control plane and keep Vercel config simpler.

## Backend Integration Standards

- `NEXT_PUBLIC_API_BASE` must point to `https://api.swainos.com` in production, not an ad hoc origin URL.
- Frontend requests should target the documented API hostname consistently.
- Do not scatter backend hostnames across multiple environment variables or code paths.

## Operations And Verification

After any production deployment or hosting configuration change, verify:

- login works
- when Turnstile is enabled: widget completes, `POST /api/auth/login` succeeds with token, and `invalid_captcha` paths behave as expected
- auth callback works
- protected app routes render
- core dashboard routes load without request loops
- API calls resolve against the intended backend hostname
- no unexpected redirect chain exists between Cloudflare and Vercel
- **admin-only Settings:** non-admin users do not see Settings in nav and receive `/unauthorized` on direct `/settings*` URLs

## Merged code, not yet on production

Common during cost holds or manual deploy pauses:

- `main` may contain features (banners, auth hardening, admin routing) that **Vercel production has not built yet**. Confirm the **Production** deployment’s commit SHA and deploy time in the Vercel dashboard.
- Preview deployments use preview env vars; do not assume they match production Turnstile or API bases.
- Coordinate with **Render** (`render-guidelines.md`): a live frontend with a suspended API will surface auth or API errors—check both sides.

## Operational Alert Thresholds

Frontend hosting alerts should include:

- sudden growth in server-rendered request volume without matching user growth
- abnormal request bursts from preview URLs
- spikes in backend run-trigger UI traffic from a single source
- unexpected increase in data-refresh polling traffic

## Minimum Production Checklist

- the SwainOS frontend Vercel project is linked to `apps/web`
- production env vars are set correctly
- preview env vars are separated from production
- `app.swainos.com` is configured as the production domain intentionally
- `swainos.com` root behavior is documented and implemented intentionally
- Spend Management is enabled
- threshold notifications are enabled
- pause-on-threshold behavior is enabled
- preview and generated deployment URLs are appropriately protected
- login and callback routes are verified after deploy
- frontend talks only to the canonical API hostname

## Related documentation

- [Frontend code documentation](swainos-code-documentation-frontend.md)
- [Render guidelines](render-guidelines.md) — API availability and resume checklist
- [Cloudflare guidelines](cloudflare-guidelines.md)
