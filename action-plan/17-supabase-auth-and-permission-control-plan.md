# 🎯 Supabase Auth and Permission Control - Invite-Only Access Foundation

> **Version**: v1.0  
> **Status**: 🚀 READY TO IMPLEMENT  
> **Date**: 2026-03-10

**Target Components**: `SwianOS_Documentation/supabase/migrations/`, `SwainOS_BackEnd/src/core/`, `SwainOS_BackEnd/src/api/`, `SwainOS_BackEnd/src/services/`, `SwainOS_BackEnd/src/repositories/`, `SwainOS_BackEnd/src/schemas/`, `SwainOS_FrontEnd/apps/web/src/app/`, `SwainOS_FrontEnd/apps/web/src/components/layout/`, `SwainOS_FrontEnd/apps/web/src/features/settings/`, `SwainOS_FrontEnd/apps/web/src/lib/api/`, `SwainOS_FrontEnd/apps/web/src/lib/constants/`, `SwainOS_FrontEnd/apps/web/src/lib/types/`, `SwianOS_Documentation/docs/`  
**Primary Issues**: SwainOS currently has no formal authentication boundary, no invite-only login flow, no user access model for left-navigation modules, and no server-enforced authorization contract for protected routes or APIs.  
**Objective**: Implement a lean, production-ready Supabase authentication and authorization foundation for a small invite-only user base, with admin override, per-module permissions, route/API protection, and documentation-aligned contracts.

## 📋 **QUICK SUMMARY**

**What We're Building/Fixing**: A Supabase-powered, invite-only auth system where admins create users, users log in with secure cookie-based sessions, and each module in the left navigation is controlled by an explicit permission key.

**Critical Issues Being Addressed**:
- SwainOS is currently open inside the app shell -> establish authenticated entry, protected routes, and session-aware server rendering.
- Left-nav access is all-or-nothing -> introduce explicit per-feature permission keys that control visibility and access.
- UI-only hiding would be unsafe -> enforce the same permissions in frontend routing, backend APIs, and Supabase RLS.
- Admin access is currently informal -> define a first-class `admin` role with full-platform access and access-management capability.

**Success Metrics**:
- Invite-only users can sign in and sign out using Supabase auth without any self-registration flow.
- Every left-nav destination is backed by a stable permission key and hidden when access is not granted.
- Unauthorized users are blocked at UI, route, API, and database layers for protected modules.
- Admins can manage user access without direct database edits after the initial rollout.
- Documentation, payload contracts, and implementation naming stay aligned across frontend, backend, and schema.

---

## 🎯 **EXECUTION STATUS**

**Progress**: 0 of 1 execution track completed  
**Current Status**: Planning complete; implementation follows one coordinated big-bang cutover with strict readiness gates and full-stack validation.

| Track | Status | Priority | Notes |
|-------|--------|----------|-------|
| 🚀 One-Bang Auth + Authorization Cutover | 📋 PENDING | HIGH | Implement schema, backend authz, frontend auth shell/nav, admin access UX, and QA in one coordinated release |

---

## 🚨 **CRITICAL REQUIREMENTS**

### **⚠️ MANDATORY CHECKLIST - Must Complete Before Implementation**

- [ ] **Type Safety**: All new frontend contracts and backend schemas use explicit types; no `any`.
- [ ] **Naming Conventions**: New tables/columns use `snake_case`; frontend types and JSON use `camelCase`; modules/files follow SwainOS naming rules.
- [ ] **Cookie-Based Auth Only**: Supabase auth uses SSR-safe cookies; no token storage in `localStorage`.
- [ ] **Modern OAuth/OIDC Flow**: Use Supabase PKCE-based auth flow for browser login and callback handling.
- [ ] **Server Enforcement**: Hidden navigation alone is not considered security; all protected reads/writes require server-side authorization checks.
- [ ] **JWT Trust Boundary**: Frontend-to-backend API calls use bearer tokens from authenticated Supabase sessions; backend verifies JWT on protected endpoints.
- [ ] **RLS Required**: Supabase tables introduced for auth/access ship with explicit RLS policies.
- [ ] **Invite-Only Flow**: No public registration route or self-serve signup path.
- [ ] **Least Privilege Keys**: `service_role` key is backend-only and never shipped to frontend bundles.
- [ ] **Session Hardening**: Cookie settings, idle timeout expectations, and refresh behavior are documented and validated.
- [ ] **Auditability**: Access changes capture `updated_by`, `updated_at`, and reason metadata where practical.
- [ ] **Documentation Update**: `docs/swainos-code-documentation-frontend.md`, `docs/swainos-code-documentation-backend.md`, and supporting auth/query docs are updated with final contracts.
- [ ] **No Dead Code**: Remove placeholder auth scaffolding and unused access branches as the real implementation lands.

### **Documentation Update Requirement**

> **⚠️ IMPORTANT**: This rollout must update:
> - `docs/swainos-code-documentation-frontend.md`
> - `docs/swainos-code-documentation-backend.md`
> - `docs/frontend-data-queries.md`
> - `docs/sample-payloads.md`
> - `docs/swainos-terminology-glossary.md` (if new auth/access terms are introduced)

---

## 📐 **NAMING CONVENTION ALIGNMENT**

All code in this plan follows current SwainOS conventions and should not introduce alternate auth/access naming patterns.

| Element | Convention | Example |
|---------|------------|---------|
| Backend modules | `snake_case.py` | `auth_access_service.py` |
| Frontend components | `kebab-case.tsx` | `user-access-table.tsx` |
| Frontend services | `camelCaseService.ts` | `authAccessService.ts` |
| Hook files | `useCamelCase.ts` | `useAuthenticatedUser.ts` |
| Database tables | `snake_case`, plural | `user_profiles`, `user_module_permissions` |
| Database columns | `snake_case` | `role`, `is_active`, `permission_key` |
| API endpoints | `kebab-case` | `/api/v1/auth/me`, `/api/v1/settings/user-access` |
| JSON properties | `camelCase` | `isAdmin`, `permissionKeys`, `canManageAccess` |
| Permission constants | `SCREAMING_SNAKE_CASE` or string-literal registry | `MODULE_PERMISSION_KEYS` / `"marketing_web_analytics"` |

---

## 🔧 **STRATEGIC APPROACH**

### **Implementation Philosophy**
Use the simplest durable model that fits a 10-12 user invite-only platform: one role boundary (`admin` vs `member`), one explicit permission registry for modules, one canonical user-access read path, and default-deny enforcement everywhere. Avoid enterprise IAM complexity, but do not leave any security decision to the client alone.

### **Key Architecture Decisions**
- **Invite-only Supabase Auth**: Users are provisioned by admins and authenticate through Supabase-managed identities; no registration flow exists in the app.
- **Role + permission model**: `role` handles admin override; `permissionKeys` handle module-level access for left-nav destinations and protected features.
- **Permission registry mirrors nav topology**: Each protected left-nav destination maps to one stable permission key so frontend filtering and backend enforcement share the same source-of-truth vocabulary.
- **Server-first session resolution**: Session and access context are loaded on the server and fed into the app shell instead of relying on client-only auth state.
- **Default deny**: If access data is missing, stale, or ambiguous, the user is treated as unauthorized until proven otherwise.
- **Single source of permission truth**: One `route-to-permission` matrix is maintained in code/docs and used by frontend filtering, route guards, and backend endpoint checks.
- **Explicit API auth boundary**: Protected FastAPI endpoints accept only verified Supabase bearer JWTs and derive caller identity from token claims, not from client-sent user identifiers.

### **Data Flow**

```
Admin invites user in Supabase
  -> auth.users identity created
  -> user profile/access rows created or synced

User signs in
  -> Supabase cookie session issued
  -> auth callback exchanges code and finalizes session
  -> Next.js middleware validates auth presence
  -> server-side access loader fetches profile + permissions

Access context
  -> filters left-nav items
  -> guards route entry
  -> attaches bearer token to protected backend calls

Backend + Supabase
  -> service/repository checks role + permission keys
  -> RLS policies restrict direct table access
```

### **Auth Security Baseline (Best Practice Standards)**

- **Session model**: SSR cookie sessions only, `HttpOnly` + `Secure` in production, with explicit same-site policy and controlled refresh behavior.
- **Protocol posture**: Use Supabase PKCE flow; avoid implicit or localStorage token patterns.
- **Key management**: Keep anon key only on frontend; keep `service_role` and JWT verification secrets server-side only.
- **Authorization layering**: UI filtering + route guard + backend permission checks + RLS; no single-layer trust.
- **Abuse protection**: Cloudflare rate limits + backend throttling on auth-sensitive endpoints (`/auth/me`, admin access mutations, invite-related operations).
- **State-changing call protection**: Ensure CSRF-safe interaction model for cookie-backed auth (origin checks and anti-CSRF strategy on mutation routes).
- **Backend identity verification**: FastAPI verifies Supabase JWTs and never trusts user identity values passed in payload/query for authorization decisions.
- **Identity lifecycle controls**: Immediate access revocation for inactive users; predictable session invalidation behavior for role/permission changes.
- **Observability**: Structured security logs for sign-in/sign-out, denied access, permission updates, and admin-role changes.
- **Deployment hygiene**: Strict HTTPS, callback allowlist per environment, and no wildcard redirect URLs.
- **Optional hardening**: Enforce MFA for admin accounts if operationally acceptable for your small user base.

### **Recommended Access Model**

**Role values (`user_profiles.role`)**:
- `admin`: full platform access, access-management capability, bypasses per-module permission restrictions.
- `member`: standard invite-only user; access is granted only through explicit module permissions.

**Permission key set (`user_module_permissions.permission_key`)**:
- `command_center`
- `ai_insights`
- `itinerary_forecast`
- `itinerary_actuals`
- `destination`
- `travel_consultant`
- `travel_agencies`
- `marketing_web_analytics`
- `search_console_insights`
- `cash_flow`
- `debt_service`
- `fx_command`
- `operations`
- `settings_job_controls`
- `settings_run_logs`
- `settings_user_access`

**Admin bootstrap note**:
- Seed `ianswain2@gmail.com` as the initial `admin` account during rollout.
- Preserve an explicit bootstrap step so first-admin creation is deliberate and auditable.

### **Route-to-Permission Matrix (v1)**

| Route Surface | Permission Key |
|---------------|----------------|
| `/command-center` | `command_center` |
| `/ai-insights` | `ai_insights` |
| `/itinerary-forecast` | `itinerary_forecast` |
| `/itinerary-actuals` | `itinerary_actuals` |
| `/destination` | `destination` |
| `/travel-consultant` | `travel_consultant` |
| `/travel-agencies` | `travel_agencies` |
| `/marketing` (+ web analytics subroutes) | `marketing_web_analytics` |
| `/marketing/search-console-insights` (+ `/marketing/search-console-insights/pages/[...pagePath]`) | `search_console_insights` |
| `/cash-flow` (+ `forecast`, `ap-schedule`, `scenarios`) | `cash_flow` |
| `/debt-service` | `debt_service` |
| `/fx-command` | `fx_command` |
| `/operations` | `operations` |
| `/settings` | `settings_job_controls` |
| `/settings/run-logs` | `settings_run_logs` |
| `/settings/user-access` (new admin screen) | `settings_user_access` (`admin` role still required server-side) |

---

## 🚀 **ONE-BANG IMPLEMENTATION PATH**
*Priority: High - coordinated full-stack cutover in one release*

### **ACCESS MODEL AND CONTRACT FREEZE**

### **🎯 Objective**
Define the exact role, permission, and route-mapping contract so implementation stays simple and consistent across frontend, backend, and database.

### **🔍 Analysis / Discovery**
- Current frontend navigation is centralized in `apps/web/src/lib/constants/navigation.ts`, which is the right anchor for a permission registry.
- Current app shell (`apps/web/src/app/layout.tsx` + `components/layout/system-shell.tsx`) renders the full shell without auth context.
- There is no existing Supabase auth client, Next.js auth middleware, or backend user-access contract in the current codebase snapshot.
- Settings already exists as an operational admin surface, making it the best home for access-management UX.

### **⚙️ Implementation**

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `SwainOS_FrontEnd/apps/web/src/lib/constants/navigation.ts` | Modify | Add permission-key mapping metadata for protected nav items |
| `SwainOS_FrontEnd/apps/web/src/lib/types/auth.ts` | Create | Define typed auth/session/access contracts |
| `SwianOS_Documentation/docs/swainos-code-documentation-frontend.md` | Modify | Document auth shell and navigation authorization model |
| `SwianOS_Documentation/docs/swainos-code-documentation-backend.md` | Modify | Document backend auth/access service contracts |

**Implementation Steps:**
1. Freeze role values to `admin` and `member`; reject additional role tiers for v1.
2. Freeze a one-to-one permission registry for every protected left-nav destination and protected settings child route.
3. Decide whether settings children inherit a parent permission or remain independent; recommended v1 is independent keys so job controls and run logs can be granted separately.
4. Publish route-to-permission mapping in docs before implementation begins.
5. Freeze non-goals:
   - no public signup
   - no multi-tenant org hierarchy
   - no custom policy builder UI
   - no per-widget or per-chart permissions in v1

### **✅ Validation Checklist**
- [ ] `admin` vs `member` semantics documented.
- [ ] Every protected route has a named permission key.
- [ ] Nav labels and permission names are clearly distinguished.
- [ ] Non-goals are documented to prevent scope creep.

---

### **SUPABASE AUTH FOUNDATION AND RLS**

### **🎯 Objective**
Implement the Supabase schema, invite workflow assumptions, and RLS boundaries required for secure login and access storage.

### **🔄 Implementation**

**Schema objects (v1):**
- `user_profiles`
- `user_module_permissions`
- optional helper view: `user_access_summary_v1`

**Migration location and numbering:**
- Add new migrations in `SwianOS_Documentation/supabase/migrations/`.
- Start from the next available sequence after `0095`.
- Keep filename pattern aligned to existing standards (example: `0096_create_auth_access_domain_v1.sql`).

**Recommended table design:**
- `user_profiles`
  - `user_id uuid primary key references auth.users(id)`
  - `email text not null unique`
  - `role text not null check (role in ('admin', 'member'))`
  - `is_active boolean not null default true`
  - `created_at timestamptz not null default now()`
  - `updated_at timestamptz not null default now()`
  - `created_by uuid null`
  - `updated_by uuid null`
- `user_module_permissions`
  - `id uuid primary key default gen_random_uuid()`
  - `user_id uuid not null references user_profiles(user_id) on delete cascade`
  - `permission_key text not null`
  - `created_at timestamptz not null default now()`
  - `created_by uuid null`
  - unique `(user_id, permission_key)`

**RLS policy shape:**
- Users can read only their own `user_profiles` and `user_module_permissions`.
- Admins can read/write all access-management rows.
- Service-role workflows can bootstrap and maintain access rows.
- Deny all unauthenticated direct reads.

**Supabase project settings requirements:**
- Disable public signup in Auth settings.
- Configure invite/recovery redirect URLs for local and Vercel domains.
- Confirm invite email template and sender setup for delivery reliability.
- Validate cookie/JWT configuration against Cloudflare-proxied domain behavior.
- Restrict redirect URL allowlist to exact trusted domains only (no broad wildcards).

**Trigger/helper expectations:**
- Keep `updated_at` synchronized automatically.
- Consider a secure function or trigger path to initialize `user_profiles` after invite acceptance if manual row creation is not guaranteed.
- Avoid relying on mutable email as the primary join; `auth.users.id` is the canonical key.

### **✅ Validation Checklist**
- [ ] Migration creates constrained tables with indexes and uniqueness guarantees.
- [ ] RLS is enabled on all auth/access tables.
- [ ] Policies support self-read, admin-manage, and service bootstrap only.
- [ ] Initial admin bootstrap path is documented and tested.
- [ ] Invite acceptance does not leave orphaned authenticated users without an access row.
- [ ] Public signup is disabled in Supabase project settings.
- [ ] Redirect URLs are configured for all active environments.

---

### **BACKEND AUTHORIZATION SERVICES AND ADMIN APIS**

### **🎯 Objective**
Expose stable backend contracts for current-user access reads and admin-only user access management.

### **🔄 Implementation**

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `SwainOS_BackEnd/src/core/config.py` | Modify | Add auth-related environment variables and config validation |
| `SwainOS_BackEnd/src/core/supabase.py` | Modify | Support authenticated/service access helpers for auth-aware reads |
| `SwainOS_BackEnd/src/repositories/auth_access_repository.py` | Create | Read/write user profile and permission rows |
| `SwainOS_BackEnd/src/services/auth_access_service.py` | Create | Role/permission evaluation and access summaries |
| `SwainOS_BackEnd/src/schemas/auth_access.py` | Create | Typed request/response schemas |
| `SwainOS_BackEnd/src/api/auth.py` | Create | Current-user auth/access endpoints |
| `SwainOS_BackEnd/src/api/settings_user_access.py` | Create | Admin-only access management endpoints |
| `SwainOS_BackEnd/src/core/auth.py` | Create | Supabase JWT verification and caller context helpers |
| `SwainOS_BackEnd/src/main.py` | Modify | Register auth/access routers |

**Planned endpoint family (`/api/v1/auth` and `/api/v1/settings/user-access`):**
- `GET /api/v1/auth/me`
- `POST /api/v1/auth/sign-out` (optional if handled entirely in frontend via Supabase)
- `GET /api/v1/settings/user-access`
- `GET /api/v1/settings/user-access/{user_id}`
- `PUT /api/v1/settings/user-access/{user_id}`
- `POST /api/v1/settings/user-access/{user_id}/deactivate`
- `POST /api/v1/settings/user-access/{user_id}/reactivate`

**Authorization rules:**
- `GET /api/v1/auth/me` returns current user profile + effective access summary.
- Admin-only endpoints require `role=admin`.
- Feature endpoints that own sensitive platform data should accept a shared authorization helper so access logic is not duplicated ad hoc per route.
- For v1, backend permission checks should guard module families such as marketing, settings, operations, finance, AI, and sales surfaces where direct API calls could bypass hidden navigation.
- `settings_user_access` routes remain admin-only even if the permission key is present for non-admin users.
- Protected endpoints must resolve caller identity from verified JWT claims (`sub` as canonical user id) before role/permission checks.

### **✅ Validation Checklist**
- [ ] Backend access logic is centralized in service/repository layers, not copy-pasted in route files.
- [ ] Authenticated current-user endpoint returns stable `camelCase` payloads.
- [ ] Admin mutation endpoints reject non-admin callers consistently.
- [ ] Shared permission helper exists for protecting future endpoint families.
- [ ] Tests cover role override, missing permission, inactive user, and missing-profile cases.
- [ ] API behavior is explicit: `401` for unauthenticated and `403` for authenticated-but-forbidden.
- [ ] JWT verification is enforced on all protected endpoints and covered by tests (valid token, expired token, invalid signature).

---

### **FRONTEND SESSION BOOTSTRAP AND PROTECTED NAVIGATION**

### **🎯 Objective**
Add Supabase auth clients, protected app entry, permission-filtered navigation, and route-level denial behavior without introducing fragile client-only synchronization.

### **🔄 Implementation**

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `SwainOS_FrontEnd/apps/web/package.json` | Modify | Add Supabase auth dependencies |
| `SwainOS_FrontEnd/apps/web/src/app/layout.tsx` | Modify | Resolve authenticated session and access summary server-side |
| `SwainOS_FrontEnd/apps/web/src/middleware.ts` | Create | Redirect unauthenticated users away from protected routes |
| `SwainOS_FrontEnd/apps/web/src/app/auth/callback/route.ts` | Create | Handle PKCE code exchange and finalize session cookies |
| `SwainOS_FrontEnd/apps/web/src/components/layout/system-shell.tsx` | Modify | Accept authenticated user/access context |
| `SwainOS_FrontEnd/apps/web/src/components/layout/side-nav.tsx` | Modify | Filter items by effective permission set |
| `SwainOS_FrontEnd/apps/web/src/lib/constants/navigation.ts` | Modify | Attach permission metadata to nav items |
| `SwainOS_FrontEnd/apps/web/src/lib/supabase/server.ts` | Create | Server Supabase client using cookies |
| `SwainOS_FrontEnd/apps/web/src/lib/supabase/browser.ts` | Create | Browser Supabase client for login/logout actions |
| `SwainOS_FrontEnd/apps/web/src/lib/auth/getAuthenticatedUser.ts` | Create | Server helper to resolve session + access |
| `SwainOS_FrontEnd/apps/web/src/lib/api/httpClient.ts` | Modify | Attach bearer token for authenticated protected API calls |
| `SwainOS_FrontEnd/apps/web/src/app/login/page.tsx` | Create | Invite-only login screen |
| `SwainOS_FrontEnd/apps/web/src/app/unauthorized/page.tsx` | Create | Permission denied state |

**Frontend behavior requirements:**
- Root shell must not render full protected navigation before access context resolves.
- Middleware protects all authenticated app routes except explicit public auth pages.
- Navigation filtering reads the same permission keys used by backend contracts.
- Admin users see all protected nav items without per-item toggles.
- Unauthorized direct route entry redirects or renders a dedicated `unauthorized` page rather than exposing partial module content.
- Avoid `useEffect`-driven auth orchestration where server loaders or action-triggered logic can handle the flow more cleanly.
- Protected API calls from frontend include bearer token from current Supabase session; backend permission checks run against verified token identity.

**Required environment variables (frontend):**
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `NEXT_PUBLIC_API_BASE`
- Optional for clarity in redirects: `NEXT_PUBLIC_SITE_URL`

**Required environment variables (backend):**
- `SUPABASE_URL`
- `SUPABASE_SERVICE_ROLE_KEY`
- `SUPABASE_JWT_SECRET` (or equivalent verified JWT strategy aligned to project settings)

**Operational auth settings (recommendation):**
- Access token TTL: short-lived (for example, <= 60 minutes).
- Refresh/session max duration: explicit and documented (for example, 7-30 days based on your risk tolerance).
- Admin role changes and user deactivation: enforce near-immediate permission effect on next request.

### **✅ Validation Checklist**
- [ ] Unauthenticated users are redirected to login before protected content renders.
- [ ] Authenticated users with limited permissions only see permitted nav destinations.
- [ ] Direct URL entry to unauthorized pages is blocked.
- [ ] Login/logout behavior works with cookie sessions on Vercel.
- [ ] No auth token is stored in `localStorage`.
- [ ] Middleware matcher excludes static assets and public auth routes correctly.

---

### **ACCESS MANAGEMENT UX**

### **🎯 Objective**
Provide a small, admin-only Settings surface for viewing users, assigning permissions, and deactivating access without requiring manual SQL changes.

### **🔄 Implementation**

**Recommended UX location:**
- Add an admin-only Settings child route such as `/settings/user-access`.
- Keep the UI table-driven and operational, not “enterprise IAM”.

**Recommended UX elements:**
- User list with email, role, active state, and last updated timestamp.
- `Admin` checkbox/toggle.
- Permission checkbox group based on the centralized module registry.
- Save action with optimistic or explicit success/error feedback.
- Deactivate/reactivate controls.
- Clear badge or copy indicating that admins automatically receive all permissions.
- Invite lifecycle state where available (`pending_invite`, `active`, `deactivated`).

**Important UX rules:**
- Prevent an admin from accidentally removing the last active admin without an explicit safeguard.
- Disable module checkboxes when `role=admin` if the backend treats admin as full override.
- Reflect settings child permissions separately if `settings_job_controls` and `settings_run_logs` remain independently grantable.

### **✅ Validation Checklist**
- [ ] Access-management screen is visible only to admins.
- [ ] Permission registry is rendered from shared constants, not duplicated strings.
- [ ] Last-admin lockout safeguard exists.
- [ ] Save flows surface validation errors clearly.
- [ ] Deactivated users lose route/API access immediately on next request/session refresh.

---

### **QA, ROLLOUT, AND DOCUMENTATION CLOSEOUT**

### **🎯 Objective**
Validate the full invite-only auth flow, confirm permission enforcement across layers, and align documentation to the final shipped implementation.

### **🧪 Testing**
- Backend tests:
  - current-user access summary
  - admin vs member authorization
  - missing permission rejection
  - inactive user rejection
  - RLS-sensitive repository behavior where applicable
  - JWT verification paths (valid, expired, malformed/invalid signature)
- Frontend checks:
  - login redirect behavior
  - auth callback route behavior after invite/login
  - logout behavior
  - filtered left-nav rendering
  - unauthorized route redirect/page behavior
  - admin-only Settings access-management surface
  - authenticated API calls include bearer token for protected endpoints
- Deployment checks:
  - Vercel environment variables configured correctly
  - Supabase redirect URLs aligned to deployed domain(s)
  - cookie/session behavior works behind Cloudflare proxying

### **Cutover and Rollout Sequence**
1. Apply schema + RLS migrations in non-production.
2. Bootstrap initial `admin` (`ianswain2@gmail.com`) and one constrained `member`.
3. Deploy backend auth/access endpoints and verify `GET /api/v1/auth/me`.
4. Deploy frontend login/middleware/nav filtering and validate permission matrix.
5. Remove any temporary rollout bypasses and rerun end-to-end checks.
6. Promote to production and verify both admin and constrained-member journeys.

### **End-to-End User Journey Validation (Must Pass)**

1. **Admin invite + first login**
   - Admin invites a new user from Supabase.
   - User receives invite, completes login via PKCE callback route, lands in app shell.
   - `GET /api/v1/auth/me` returns active profile + permission set.
2. **Constrained member experience**
   - Member with limited permissions sees only authorized left-nav items.
   - Direct visit to unauthorized route returns redirect/unauthorized state.
   - Protected API request without permission returns `403`.
3. **Unauthenticated user protection**
   - Visiting protected route while signed out redirects to `/login`.
   - Backend protected API without token returns `401`.
4. **Admin access management**
   - Admin opens `/settings/user-access`, updates member permissions.
   - Permission change is effective on next request.
   - Last-admin removal/deactivation is blocked by backend guardrails.
5. **Revocation and recovery**
   - Deactivated user is denied route/API access.
   - Re-activated user regains access according to assigned permissions.
6. **Operational safety checks**
   - Session/cookie behavior remains correct on Vercel behind Cloudflare.
   - Security logs capture access denials and permission changes.
   - Docs and sample payloads reflect final contracts.

### **📚 Documentation Updates**

| Document | Section | Change Description |
|----------|---------|-------------------|
| `docs/swainos-code-documentation-frontend.md` | Structure, Route Surface, UX notes, Environment | Add login route, middleware, auth shell flow, permission-aware navigation, Supabase envs |
| `docs/swainos-code-documentation-backend.md` | Architecture, Active Endpoint Families, Contract Rules | Add auth/access endpoint family, repository/service layer, role/permission model |
| `docs/frontend-data-queries.md` | Auth/settings query inventory | Add current-user access query and admin access-management endpoints |
| `docs/sample-payloads.md` | Auth + user-access examples | Add canonical `GET /api/v1/auth/me` and admin update payloads |
| `docs/swainos-terminology-glossary.md` | Access-control terms | Add terms only if needed (`admin`, `member`, `permission key`, `inactive user`) |
| `action-plan/action-log` | Milestones | Append implementation milestones and closeout notes |

### **✅ Validation Checklist**
- [ ] Backend lint/tests pass for touched modules.
- [ ] Frontend lint/build pass for touched modules.
- [ ] Manual QA covers invite acceptance through protected-route access.
- [ ] Docs reflect shipped routes, env vars, and access contracts.
- [ ] Rollout includes initial admin bootstrap and at least one limited-permission user verification.
- [ ] Production cutover and rollback path are validated in staging.

---

## 🚦 **DEVELOPMENT READINESS CHECKLIST**

Use this gate before coding begins.

- [ ] Route-to-permission matrix is approved and frozen.
- [ ] Supabase project settings (signup disabled, redirects configured) are confirmed.
- [ ] Initial admin bootstrap method is approved.
- [ ] Migration sequence slot is reserved in `supabase/migrations`.
- [ ] Backend ownership is assigned (`auth`, `settings user access`, permission helper).
- [ ] Frontend ownership is assigned (`login`, `middleware`, shell/nav filtering).
- [ ] QA matrix includes admin, constrained-member, and inactive-user scenarios.
- [ ] Documentation update owners are assigned.

---

## ⚠️ **RISK MANAGEMENT**

### **High Priority Risks**
- **UI-only authorization drift**: users could still hit protected endpoints directly -> **Mitigation**: shared backend permission helper + RLS + middleware/route denial.
- **Missing access row after invite acceptance**: valid login without profile/permissions creates ambiguous state -> **Mitigation**: bootstrap trigger/function or explicit provisioning workflow; default deny when access row missing.
- **Permission sprawl**: route names and permission keys drift apart over time -> **Mitigation**: central permission registry tied to navigation constants and documented route mapping.
- **Admin lockout**: last active admin accidentally removed or deactivated -> **Mitigation**: backend guard preventing last-admin removal/deactivation.
- **Token/session misconfiguration**: weak cookie or redirect settings can create auth bypass or token leakage risk -> **Mitigation**: enforce HTTPS-only cookies, strict redirect allowlist, and pre-prod auth configuration checklist.

### **Medium Priority Risks**
- **Proxy/session edge cases on Vercel + Cloudflare**: cookies or redirects behave differently between environments -> **Mitigation**: validate auth callback URLs, cookie config, and end-to-end login in deployed staging/prod-like environment.
- **Over-engineering for a tiny user base**: too many roles or policies slow delivery -> **Mitigation**: keep v1 to `admin` + `member` and module-level permissions only.
- **Settings permission ambiguity**: parent/child settings routes may be inconsistent -> **Mitigation**: freeze child-route permission semantics before implementation and document them explicitly.

### **Rollback Strategy**
1. Disable access-management mutations and revert to known-good admin-only access if rollout breaks.
2. Temporarily grant admin access to the small trusted user group while preserving auth login enforcement.
3. Revert frontend nav filtering and route guards only in sync with backend/RLS rollback so behavior does not drift.
4. Re-run bootstrap verification for `ianswain2@gmail.com` before reopening the app to non-admin users.

---

## 📊 **SUCCESS CRITERIA**

### **Technical Success Metrics**

| Metric | Target | Verification Method |
|--------|--------|---------------------|
| Invite-only login flow | 100% functional | Manual auth QA in deployed environment |
| Permission enforcement | No unauthorized route/API access | Targeted backend tests + manual route checks |
| Navigation accuracy | 100% of protected left-nav items filtered correctly | Permission matrix QA |
| Admin access management | User permissions editable without SQL | Settings workflow QA |
| Session handling | Stable SSR cookie auth on Vercel | Deployed sign-in/sign-out validation |
| Security posture | Baseline controls verified | Auth hardening checklist + config review |

### **User Experience Success**

| Scenario | Expected Outcome |
|----------|------------------|
| Invited user signs in | Lands in the protected app with only approved modules visible |
| Limited-permission user pastes unauthorized URL | Receives redirect or unauthorized state, not partial data |
| Admin opens Settings user access | Can update role/activity/permissions safely |
| Admin user logs in | Sees full platform navigation without manual per-module assignment |

---

## 🔗 **RELATED DOCUMENTATION**

- **[Action Plan Template](./action-plan-template.md)** - Required planning structure
- **[Platform Ingestion Control Plane](./16-platform-ingestion-control-plane-and-job-operations-plan-completed.md)** - Current Settings/Operations architecture reference
- **[Backend Code Documentation](../docs/swainos-code-documentation-backend.md)** - Backend layering and endpoint conventions
- **[Frontend Code Documentation](../docs/swainos-code-documentation-frontend.md)** - App Router shell, route surface, and nav structure

---

## 📚 **TECHNICAL REFERENCE**

### **Current User Access Payload**

```json
{
  "data": {
    "userId": "2e4f6dc0-6b70-4aa0-b8d0-9f13c9c5f321",
    "email": "ianswain2@gmail.com",
    "role": "admin",
    "isAdmin": true,
    "isActive": true,
    "permissionKeys": [
      "command_center",
      "marketing_web_analytics",
      "search_console_insights",
      "settings_job_controls",
      "settings_run_logs"
    ],
    "canManageAccess": true
  },
  "pagination": null,
  "meta": {
    "source": "supabase_auth + user_access_summary_v1"
  }
}
```

### **Suggested Type Contracts**

```typescript
export type AppRole = "admin" | "member";

export type ModulePermissionKey =
  | "command_center"
  | "ai_insights"
  | "itinerary_forecast"
  | "itinerary_actuals"
  | "destination"
  | "travel_consultant"
  | "travel_agencies"
  | "marketing_web_analytics"
  | "search_console_insights"
  | "cash_flow"
  | "debt_service"
  | "fx_command"
  | "operations"
  | "settings_job_controls"
  | "settings_run_logs"
  | "settings_user_access";

export interface AuthenticatedUserAccess {
  userId: string;
  email: string;
  role: AppRole;
  isAdmin: boolean;
  isActive: boolean;
  permissionKeys: ModulePermissionKey[];
  canManageAccess: boolean;
}
```

### **Permission Registry Pattern**

```typescript
export const MODULE_PERMISSION_KEYS = {
  commandCenter: "command_center",
  aiInsights: "ai_insights",
  itineraryForecast: "itinerary_forecast",
  itineraryActuals: "itinerary_actuals",
  destination: "destination",
  travelConsultant: "travel_consultant",
  travelAgencies: "travel_agencies",
  marketingWebAnalytics: "marketing_web_analytics",
  searchConsoleInsights: "search_console_insights",
  cashFlow: "cash_flow",
  debtService: "debt_service",
  fxCommand: "fx_command",
  operations: "operations",
  settingsJobControls: "settings_job_controls",
  settingsRunLogs: "settings_run_logs",
  settingsUserAccess: "settings_user_access",
} as const;
```

---

## 🎯 **COMPLETION CHECKLIST**

### **Pre-Implementation**
- [ ] Confirm final protected route inventory.
- [ ] Confirm Supabase invite flow and redirect/callback requirements.
- [ ] Confirm first-admin bootstrap process for `ianswain2@gmail.com`.

### **Implementation Quality Gates**
- [ ] All new types and schemas are explicit.
- [ ] No auth state is trusted from client-only storage.
- [ ] All protected paths have server-side authorization checks.
- [ ] RLS and admin guards are in place for access tables.
- [ ] Frontend and backend naming follow current SwainOS conventions.
- [ ] Auth hardening baseline checklist is fully passed before production rollout.

### **Testing**
- [ ] Admin login verified.
- [ ] Limited-permission login verified.
- [ ] Unauthorized route access verified.
- [ ] Permission change propagation verified.
- [ ] Sign-out/session-expiry behavior verified.
- [ ] Expired/invalid token behavior verified (`401` path) and re-auth flow confirmed.

### **Documentation** *(MANDATORY)*
- [ ] `docs/swainos-code-documentation-frontend.md` updated
- [ ] `docs/swainos-code-documentation-backend.md` updated
- [ ] Query/payload/glossary docs updated where applicable
- [ ] `action-plan/action-log` updated with milestone entries
- [ ] Action plan status updated as implementation progresses

### **Final Review**
- [ ] One-bang execution track completed.
- [ ] Admin override behaves correctly.
- [ ] Limited users can see only approved modules.
- [ ] No public registration path exists.
- [ ] Deployed environment behaves correctly behind Cloudflare and on Vercel.

---

## 📝 **REVISION HISTORY**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.0 | 2026-03-10 | AI Agent + Ian | Initial execution-ready plan for invite-only Supabase auth, role + permission access control, protected navigation, and admin-managed user access |
| v1.1 | 2026-03-10 | AI Agent + Ian | Added modern auth/security baseline (PKCE, key handling, CSRF/rate-limit/session controls, strict redirect policy), expanded security success criteria, and aligned permission type/registry examples with `settings_user_access` |
| v1.2 | 2026-03-10 | AI Agent + Ian | Closed API trust-boundary gap by adding explicit frontend-to-backend bearer token propagation, PKCE callback route, backend JWT verification layer, and token-path testing requirements |
| v1.3 | 2026-03-10 | AI Agent + Ian | Refactored to single-track one-bang rollout (no phases), added explicit end-to-end user journey validation matrix, and removed remaining phase-based wording |
