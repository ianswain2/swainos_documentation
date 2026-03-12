# SwainOS Cloudflare Guidelines

> **Version**: v1.0
> **Status**: Active standard
> **Date**: 2026-03-11

## Purpose

This document defines the Cloudflare standards for `swainos.com` and all SwainOS public subdomains.

Cloudflare is the public edge for SwainOS. It owns DNS, terminates public traffic, enforces baseline abuse protection, applies rate limits and WAF rules, and protects origin infrastructure from unnecessary exposure.

These guidelines are intended for a small, private, invite-only application with a very limited user base. The goal is strong protection, simple operations, and predictable billing rather than maximum platform complexity.

## Scope

These guidelines apply to:

- `swainos.com`
- `www.swainos.com`
- `api.swainos.com`
- preview or staging subdomains that may be added later
- any future public subdomain routed through Cloudflare

## Core Principles

- Cloudflare is the canonical DNS authority for the SwainOS zone.
- All public production traffic should pass through Cloudflare unless there is a documented exception.
- Cloudflare protects the edge; it does not replace application authentication or backend authorization.
- Supabase Auth remains the user identity boundary for the product.
- Default-deny is preferred over broad public exposure.
- Prefer a small number of high-value, understandable rules over a sprawling ruleset.
- Every Cloudflare rule must have a clear owner and purpose.
- No bypass paths should exist that expose the origin directly to the public internet unless explicitly approved.

## Intended Role In The Stack

Cloudflare is responsible for:

- DNS for `swainos.com`
- proxying and TLS termination
- DDoS mitigation and managed edge protection
- WAF and rate limiting
- bot and abuse filtering
- edge caching for safe static assets
- redirect normalization and hostname enforcement

Cloudflare is not responsible for:

- primary application auth
- user/session management
- business authorization
- backend job execution
- database access control

## Domain And Subdomain Conventions

- Canonical frontend hostname: `app.swainos.com`
- Canonical backend hostname: `api.swainos.com`
- Root domain: `swainos.com`
- Preferred root behavior: redirect to `swaindestinations.com`
- Acceptable temporary root behavior: minimal noindex holding page
- Secondary root alias: `www.swainos.com`
- API hostname: `api.swainos.com`
- Optional staging hostname: `staging.swainos.com`
- Optional preview hostnames should follow a documented pattern and never be created ad hoc

Naming rules:

- Public hostnames use lowercase only
- Use `kebab-case` for multi-word subdomains if needed
- Avoid temporary names like `new-api`, `test2`, or `prod-final`
- Production names should be durable and meaning-based

## DNS Standards

- All production app hostnames should be proxied through Cloudflare unless there is a documented technical reason not to.
- DNS records should be minimal and intentional.
- Avoid leaving old verification, migration, or temporary records in place after cutover.
- Use `CNAME` or provider-recommended record types when integrating with Vercel.
- Use stable origin targets for the API and avoid changing records casually.
- TTL should remain on provider-managed defaults unless a migration requires temporary adjustment.
- If registrar or nameserver changes occur, DNSSEC should be re-enabled after the zone is stable.

## SSL And Transport Standards

- Enforce HTTPS for all public SwainOS hostnames.
- Use strict TLS settings appropriate for production.
- Use Full (strict) SSL whenever origin certificates support it.
- Redirect HTTP to HTTPS at the edge.
- Cookies and authenticated application traffic must only be served over HTTPS in production.
- Never leave production in Flexible SSL mode.
- Consider Authenticated Origin Pulls where the origin platform supports it and the operational overhead is justified.

## Edge Security Baseline

At minimum, the production zone should enable:

- Cloudflare proxy on production app and API records
- managed DDoS protection
- managed WAF rules
- bot protection features available on the active Cloudflare plan
- challenge or block rules for obviously abusive traffic
- security event visibility and review

## Rate Limiting Standards

Rate limiting should focus on expensive or abuse-sensitive paths first.

Required priority targets:

- auth-sensitive endpoints
- admin or access-management endpoints
- expensive analytics endpoints
- manual-run or job-trigger endpoints
- AI or report-generation endpoints

Rules should:

- target specific paths whenever possible
- distinguish read-heavy UI traffic from sensitive mutation paths
- avoid harming normal use by the small invite-only user base
- default to challenge before block when confidence is uncertain
- be documented with threshold, action, and purpose
- account for the active Cloudflare plan's rule-count and matching limitations
- assume a few excess requests may still reach the origin before mitigation is enforced

### SwainOS Required High-Cost API Rules

For `api.swainos.com`, reserve dedicated rules for these paths first:

- `POST /api/v1/ai-insights/run`
- `POST /api/v1/fx/signals/run`
- `POST /api/v1/data-jobs/scheduler/tick`
- `POST /api/v1/data-jobs/{job_key}/runs`

Suggested baseline for this private workload:

- start with low minute-level thresholds per IP
- challenge first, then block if repeated
- use separate mutation thresholds for scheduler/manual-run routes vs normal read APIs
- keep one spare rule slot for emergency abuse controls

Header trust model:

- trust client IP from Cloudflare edge only
- do not trust arbitrary client-provided forwarding headers without Cloudflare validation
- keep origin direct access minimized so Cloudflare remains the observable choke point

Plan-awareness note:

- On lower-tier plans, Cloudflare rate limiting supports only a small number of rules and fewer matching/counting dimensions than higher tiers.
- SwainOS should reserve those limited rules for the highest-risk endpoints first rather than trying to rate-limit every route.

## WAF And Firewall Rule Standards

- Prefer Cloudflare managed protections before writing custom rules.
- Custom rules should be added only for known needs.
- Every custom rule must include a short reason in documentation or dashboard notes.
- Avoid rules that are so broad they create hard-to-debug app failures.
- Review new rules against app auth routes, API callbacks, and provider webhooks before enabling block actions.

Recommended protected categories:

- credential abuse
- obvious bot scraping
- path probing
- repeated failed auth behavior
- suspicious access to admin-only endpoints

## Origin Protection Standards

- Public traffic should reach the frontend and API through Cloudflare-hosted DNS records.
- Do not publicize raw origin URLs after cutover.
- Restrict direct access to origins where the host platform supports it.
- If a backend origin must stay public, use Cloudflare rules and host-level protections together.
- Preview and staging origins should be treated as sensitive and not indexed publicly.
- Email-related DNS records must remain `DNS only` and should never be proxied.

## Authentication Boundary

- Supabase Auth is the source of truth for user identity.
- Cloudflare may add pre-auth protection, but it must not become the only gate for application access.
- Cloudflare Access is optional for staging, admin tools, or emergency lockdown scenarios.
- Cloudflare rules must never break Supabase auth callbacks, cookie flows, or verified application traffic.

## Caching Standards

Safe to cache aggressively:

- static assets
- public non-personalized files
- immutable build artifacts

Do not edge-cache by default:

- authenticated HTML
- authenticated JSON API responses
- user-specific dashboards
- session-sensitive redirects

Caching changes must be deliberate and tested against auth/session behavior.

## Redirect Standards

- Enforce one canonical frontend hostname.
- Document all redirect rules.
- Avoid redirect chains.
- Avoid wildcard redirect behavior that can interfere with auth callbacks or preview URLs.

Preferred redirect model:

- `swainos.com` -> `swaindestinations.com`
- `www.swainos.com` -> `swaindestinations.com` or `swainos.com`, but choose one durable rule and keep it consistent

## Observability And Operations

- Review Cloudflare security and traffic events after launch and after any major rule changes.
- Investigate unusual spikes in challenged, blocked, or bot traffic.
- Keep a short written record of major zone-level changes.
- Treat WAF, DNS, and rate-limit changes as production changes, not casual dashboard tweaks.

### Security and Cost Alert Thresholds

Define alerts for:

- sudden spike in challenged/blocked requests on run endpoints
- sustained increase in `429` responses for run/mutation paths
- repeated attempts against token-protected manual-run routes
- abnormal burst volume on `api.swainos.com` outside normal operator windows

## Billing And Cost-Control Principles

- Start on the smallest Cloudflare plan that provides the required WAF and rate-limit features for production.
- Add only the features that have a clear protection or reliability benefit.
- Avoid piling on paid add-ons before traffic or operational need justifies them.
- Use Cloudflare to reduce unwanted origin traffic, not to create an overly complex billable edge architecture.

## Change Management

- DNS cutovers, SSL mode changes, firewall changes, and rate-limit changes should be planned and reversible.
- Make one high-risk Cloudflare change at a time when possible.
- Validate production login, API access, and key dashboard routes after each meaningful change.

## Minimum Production Checklist

- `swainos.com` is active in Cloudflare and serving as the authoritative DNS zone
- production hostnames are proxied
- `app.swainos.com` is the canonical frontend hostname
- `api.swainos.com` is the canonical backend hostname
- `swainos.com` is configured as a redirect to `swaindestinations.com` (or temporarily as a minimal noindex holding page until redirect cutover)
- HTTPS is enforced
- SSL mode is Full (strict)
- DNSSEC is enabled after any registrar or nameserver transition is complete
- managed WAF protections are enabled
- high-value rate limits are configured
- canonical hostname redirects are configured
- auth flows work through Cloudflare without loops
- API traffic is reachable only through the intended public hostname
- origin exposure is minimized and documented
