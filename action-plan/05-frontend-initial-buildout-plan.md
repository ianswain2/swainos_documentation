# 🎯 Frontend Initial Buildout - Local App With Live Data

> **Version**: v1.0  
> **Status**: ✅ COMPLETED  
> **Date**: 2026-02-09  
> **Completion Date**: 2026-02-10

**Target Components**: `SwainOS_FrontEnd/apps/web/src/app/`, `components/`, `features/`, `lib/api/`, `lib/types/`, `lib/constants/`, `lib/utils/`  
**Primary Issues**: Local frontend must run against local backend with real Supabase-loaded data, full module navigation, and AI-forward UI.  
**Objective**: Deliver a local-running frontend that renders the primary modules with live data where available and graceful placeholders elsewhere, using the live API envelopes.

## 📋 **QUICK SUMMARY**

**What We're Building/Fixing**: A first-pass, production-quality frontend shell with real data views and AI-forward visuals aligned to TravelOS conventions.

**Critical Issues Being Addressed**:
- Data readiness → wire service clients to existing endpoints and sample payload shapes.
- Navigation completeness → align modules to `docs/scope-and-modules.md` and the spec.
- UI polish → consistent layout, empty/loading/error states, and visual hierarchy.

**Success Metrics**:
- Local backend + frontend run together without errors.
- Navigation renders all primary modules from `docs/scope-and-modules.md`.
- Core dashboards render live data from API envelopes where data exists.
- Non-implemented modules show clear placeholders without errors.
- No lint/type errors; naming conventions fully adhered to.
- No shortcut implementations: every module route is intentionally designed, contract-validated, and review-ready.

---

## 🎯 **EXECUTION STATUS**

**Progress**: 4 of 4 sections completed  
**Current Status**: Complete — lint clean and documentation current.

| Phase | Status | Priority | Notes |
|-------|---------|----------|-------|
| 1️⃣ Local Run + API Wiring | ✅ COMPLETED | HIGH | API wiring and resilient loading in place |
| 2️⃣ Core Data Views | ✅ COMPLETED | HIGH | Command Center, Cash Flow, Revenue & Bookings, FX Command |
| 3️⃣ Navigation + Layout Polish | ✅ COMPLETED | HIGH | Full module layout + consistent shells |
| 4️⃣ QA + Documentation | ✅ COMPLETED | HIGH | Lint clean and docs updated |

---

## 🚨 **CRITICAL REQUIREMENTS**

### **Precision Build Principles (No Shortcuts)**

- **Contract-first always**: Every API call and mapper must match backend schema + `docs/sample-payloads.md`.
- **TravelOS as design reference only**: Reuse style/layout language, never TravelOS domain behavior.
- **Typed end-to-end**: No `any`, no silent fallbacks, no unhandled envelope branches.
- **Beautiful UX baseline**: Each page must include polished loading, empty, error, and ready states.
- **Deterministic validation**: No phase marked complete before lint/type/manual checks pass.

### **⚠️ MANDATORY CHECKLIST - Must Complete Before Implementation**

These requirements are **NON-NEGOTIABLE** for every action plan. Do not skip any item.

- [ ] **Type Safety**: All new code uses explicit TypeScript types — NO `any` types allowed
- [ ] **Naming Conventions**: All new files, functions, and variables follow SwainOS/TravelOS naming standards
- [ ] **Import Organization**: All imports follow the standard grouping order
- [ ] **ESLint Clean**: Zero warnings, zero errors before PR submission
- [ ] **Documentation Update**: `swainos-code-documentation-frontend.md` updated to reflect current state
- [ ] **No Dead Code**: No commented-out code, no unused imports, no unused variables

### **Documentation Update Requirement**

> **⚠️ IMPORTANT**: Every action plan that modifies code MUST include updates to the relevant documentation file(s):
> - Frontend changes → Update `docs/swainos-code-documentation-frontend.md`
> - Backend changes → Update `docs/swainos-code-documentation-backend.md`

---

## 📐 **NAMING CONVENTION ALIGNMENT**

All code in this action plan MUST follow SwainOS naming conventions.

### **Files & Directories**

| Element | Convention | Example |
|---------|------------|---------|
| React component files | `kebab-case.tsx` | `cashflow-card.tsx` |
| UI primitive files | `kebab-case.tsx` | `status-pill.tsx` |
| Hook files | `useCamelCase.ts` | `useCashflow.ts` |
| Service files | `camelCaseService.ts` | `cashflowService.ts` |
| Type definition files | `camelCase.ts` | `cashflow.ts` |
| API route files | `route.ts` | `route.ts` |
| Test files | `*.test.ts(x)` | `cashflow-card.test.tsx` |

### **TypeScript Naming**

| Element | Convention | Example |
|---------|------------|---------|
| Interfaces/Types | `PascalCase` | `CashflowSummary`, `BookingForecast` |
| Enums | `PascalCase` | `CashflowBucket` |
| Enum values | `SCREAMING_SNAKE_CASE` | `NEXT_30_DAYS` |
| Variables | `camelCase` | `cashflowSummary` |
| Functions | `camelCase` with verb prefix | `fetchCashflowSummary()` |
| Boolean variables | `is/has/can` prefix | `isLoading`, `hasError` |
| Constants | `SCREAMING_SNAKE_CASE` | `DEFAULT_CURRENCY` |
| Event handlers | `handle` prefix | `handleRetry` |

### **Database & API**

| Element | Convention | Example |
|---------|------------|---------|
| Database tables | `snake_case`, plural | `itinerary_items` |
| Database columns | `snake_case` | `created_at` |
| API endpoints | `kebab-case` | `/api/v1/cash-flow/summary` |
| JSON properties | `camelCase` | `timeWindow`, `totalItems` |

---

## 🧹 **CLEAN CODE REQUIREMENTS**

### **Import Organization Standard**

All files MUST organize imports in this exact order with blank line separators:

```typescript
// 1. React and Next.js
import { useMemo } from 'react';
import { useRouter } from 'next/navigation';

// 2. External libraries (alphabetized)
import { format } from 'date-fns';

// 3. Internal: UI components
import { Card } from '@/components/ui/card';

// 4. Internal: Features and layouts
import { PageShell } from '@/components/layout/page-shell';

// 5. Internal: Lib (types, services, hooks, utils)
import type { CashflowSummary } from '@/lib/types/cashflow';
import { cashflowService } from '@/lib/api/cashflowService';
import { formatCurrency } from '@/lib/utils/formatCurrency';
```

### **Code Quality Gates**

| Gate | Requirement |
|------|-------------|
| TypeScript | No `any` types. Strict mode enabled. |
| ESLint | Zero warnings. Zero errors. |
| Unused Code | No dead code. No commented-out code. |
| Console Logs | No `console.log` in production code. |
| Magic Numbers | Extract to named constants. |
| Type Assertions | Avoid `as` casts. Use type guards instead. |

---

## 1️⃣ **Local Run + API Wiring**
*Priority: High - establish local backend/frontend connectivity*

### **🎯 Objective**
Run backend and frontend locally, ensure API base URLs and env vars are correct, and confirm data envelopes match `docs/sample-payloads.md`.

### **🔍 Analysis / Discovery**
- Verify `NEXT_PUBLIC_API_BASE` and Supabase public config are set.
- Confirm local backend endpoints return `{ data, pagination, meta }`.

### **⚙️ Implementation**

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `apps/web/src/lib/api/httpClient.ts` | Verify | Ensure envelope parsing + error handling |
| `apps/web/src/lib/api/*Service.ts` | Verify | Confirm endpoints + types |
| `apps/web/src/lib/types/*` | Verify | Align to `sample-payloads.md` |
| `apps/web/src/app/*` | Update | Wire pages to services |

**Implementation Steps:**
1. Confirm envs for frontend/backend and local run scripts.
2. Validate envelope parsing against `docs/sample-payloads.md`.
3. Add robust error state handling for failed requests.

### **✅ Validation Checklist**
- [ ] Backend and frontend run locally without errors
- [ ] All service clients return typed data
- [ ] Envelope parsing passes for all endpoints
- [ ] Preflight checklist completed (env, API reachability, local startup flow)

---

## 2️⃣ **Core Data Views**
*Priority: High - surface real data across key modules*

### **🎯 Objective**
Render real data in the most critical modules: Command Center, Cash Flow, Revenue & Bookings, and show placeholders for Debt Service, FX Command, Operations, AI Insights, and Settings where data is not yet available.

### **🔄 Implementation**
- Cashflow summary + timeseries cards.
- Bookings list + key metrics.
- Forecast widgets using booking forecasts.
- Deposits/payments-out summaries.
- Debt Service and FX Command placeholder sections with “data pending” states.
- Operations, AI Insights, and Settings placeholder views to complete navigation.

### **✅ Validation Checklist**
- [ ] All primary pages render with live data
- [ ] Empty/loading/error states are present
- [ ] Pagination fields respected in list views
- [ ] No hardcoded financial totals remain in live-data modules unless explicitly labeled as placeholder

---

## 3️⃣ **Navigation + Layout Polish**
*Priority: High - TravelOS-aligned navigation and visuals*

### **🎯 Objective**
Finalize full navigation map and consistent layout primitives with AI-forward polish.

### **🔄 Implementation**
- Ensure SideNav map covers all primary modules.
- Standardize cards, tables, and stat blocks.
- Keep assistant launcher/panel visible and consistent.

### **✅ Validation Checklist**
- [ ] Navigation includes all primary modules
- [ ] Layout components consistent across pages
- [ ] Visual hierarchy matches TravelOS style
- [ ] Assistant surfaces remain consistent and non-disruptive across all routes

---

## 4️⃣ **QA + Documentation**
*Priority: High - verify and document current state*

### **🎯 Objective**
Run lint checks, verify page flows, and update documentation to match current state.

### **🧪 Testing**
- `npm run lint`
- Manual navigation across all primary modules

### **📚 Documentation Updates**

| Document | Section | Change Description |
|----------|---------|-------------------|
| `docs/swainos-code-documentation-frontend.md` | Overview + Data & Services | Current state of live data wiring |

### **✅ Validation Checklist**
- [ ] ESLint passes with zero warnings
- [ ] Manual UX walkthrough completed
- [ ] Documentation current and accurate
- [ ] No console errors across full route walkthrough
- [ ] No unlabeled placeholders in primary module routes

---

## 🧭 **PRECISION EXECUTION PROTOCOL**

- Build in vertical slices: service → types → UI → states → documentation.
- Validate after every slice before moving forward.
- Prefer typed mapping utilities over ad-hoc transforms inside components.
- Treat all fallback UI as intentional product decisions, not silent failures.
- If contract uncertainty appears, pause implementation and resolve against backend schemas/docs first.

---

## ⚠️ **RISK MANAGEMENT**

### **High Priority Risks**
- **API mismatch**: sample payloads diverge from live responses → **Mitigation**: validate against `docs/sample-payloads.md`.
- **Partial data**: supplier invoices pending; QuickBooks/FX not yet loaded → **Mitigation**: graceful empty states and placeholder modules.

### **Rollback Strategy**
1. Revert UI wiring for unstable endpoints.
2. Fall back to placeholder cards.
3. Re-validate endpoints once data load completes.

---

## 📊 **SUCCESS CRITERIA**

### **Technical Success Metrics**

| Metric | Target | Verification Method |
|--------|--------|---------------------|
| TypeScript compilation | Zero errors | `tsc --noEmit` |
| ESLint | Zero warnings | `npm run lint` |
| Feature functionality | All acceptance criteria met | Manual testing |

### **User Experience Success**

| Scenario | Expected Outcome |
|----------|------------------|
| Navigate core modules | Pages load quickly with consistent UI |
| View cashflow dashboard | Real data and charts appear |
| View bookings & forecasts | Lists and summaries match API data |

---

## 🔗 **RELATED DOCUMENTATION**

- `./04-data-import-plan.md` - Historical data ingestion status
- `../docs/sample-payloads.md` - API payload shapes
- `../docs/swainos-code-documentation-frontend.md` - Frontend reference

---

## 📝 **REVISION HISTORY**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.0 | 2026-02-09 | SwainOS Team | Initial frontend buildout plan |
