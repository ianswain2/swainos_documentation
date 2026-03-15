# 🎯 Itinerary Status Pipeline - Status-Driven Trends + Filters

> **Version**: v1.0  
> **Status**: ✅ COMPLETED  
> **Date**: 2026-02-10  
> **Completion Date**: 2026-02-10

**Target Components**: `SwainOS_BackEnd/src/api/`, `SwainOS_BackEnd/src/repositories/`, `SwainOS_BackEnd/src/services/`, `SwainOS_BackEnd/src/schemas/`, `SwainOS_BackEnd/supabase/migrations/`, `SwainOS_FrontEnd/apps/web/src/features/revenue-bookings/`  
**Primary Issues**: Pipeline trends based on “days” are insufficient; need status-driven monthly pipeline views with date-range controls.  
**Objective**: Deliver a status-driven itinerary pipeline (monthly buckets + status counts) with 3/6/12-month filters and API contracts aligned to naming conventions.

## 📋 **QUICK SUMMARY**

**What We're Building/Fixing**: A status-first itinerary pipeline dashboard backed by a materialized view and API endpoints that support month-based trend analysis and filters.

**Critical Issues Being Addressed**:
- “Days” windows are too coarse → **Monthly status rollups**
- Sparse pipeline data → **Status buckets + monthly trend series**
- Filter needs → **3/6/12 month window toggles**

**Success Metrics**:
- Pipeline endpoints return correct counts by status
- UI can toggle 3/6/12 month windows without regression
- Status buckets match itinerary status values in Supabase
- Zero lint/type errors

---

## 🎯 **EXECUTION STATUS**

**Progress**: 3 of 3 sections completed  
**Current Status**: Complete — backend + UI aligned to pipeline status view.

| Phase | Status | Priority | Notes |
|-------|---------|----------|-------|
| 1️⃣ Data Modeling | ✅ COMPLETED | HIGH | `mv_itinerary_status_trends` migration added + applied |
| 2️⃣ API Contracts | ✅ COMPLETED | HIGH | Endpoint + schema + repo/service in place |
| 3️⃣ UI Integration | ✅ COMPLETED | HIGH | Status cards + stacked trend + detail table implemented |

---

## 🚨 **CRITICAL REQUIREMENTS**

### **⚠️ MANDATORY CHECKLIST - Must Complete Before Implementation**

- [ ] **Type Safety**: All new code uses explicit types — NO `any`
- [ ] **Naming Conventions**: Files, functions, variables follow SwainOS standards
- [ ] **Import Organization**: Standard grouping and order
- [ ] **ESLint Clean**: Zero warnings, zero errors before PR
- [ ] **Documentation Update**: `swainos-code-documentation-backend.md`, `swainos-code-documentation-frontend.md`, and `sample-payloads.md` updated
- [ ] **No Dead Code**: No unused imports, no commented-out code

---

## 📐 **NAMING CONVENTION ALIGNMENT**

- Files: `snake_case.py` (backend), `kebab-case.tsx` (components)
- Tables/columns: `snake_case`
- JSON properties: `camelCase`
- API endpoints: `kebab-case`

---

## 🔧 **STRATEGIC APPROACH**

### **Implementation Philosophy**
Status-driven pipeline analytics must be consistent and auditable. All trend logic should live in the database layer and surface through a typed API contract.

### **Key Architecture Decisions**
- **Materialized view** for monthly status rollups (`mv_itinerary_status_trends`)
- **API endpoint** that serves a combined response: summary buckets + timeline
- **Frontend filter controls** for 3/6/12 month windows

### **Data Flow**
```
itineraries -> mv_itinerary_status_trends -> /api/v1/itinerary-pipeline -> UI charts + tables
```

---

## 1️⃣ **PHASE 1: Data Modeling**
*Priority: HIGH - Build DB rollup for status trends*

### **🎯 Objective**
Create a materialized view that aggregates itineraries by month and status.

### **⚙️ Implementation**

**Migration**: `mv_itinerary_status_trends`
- Columns:
  - `period_start` (month)
  - `itinerary_status`
  - `itinerary_count`
- Filters:
  - Exclude test/snapshot/duplicate itineraries if required

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `SwainOS_BackEnd/supabase/migrations/00xx_mv_itinerary_status_trends.sql` | Create | Materialized view for status trends |

### **✅ Validation Checklist**
- [ ] View builds and refreshes successfully
- [ ] Counts match manual SQL checks
- [ ] Status values match `itinerary_status` enum

---

## 2️⃣ **PHASE 2: API Contracts**
*Priority: HIGH - Expose pipeline via a stable endpoint*

### **🎯 Objective**
Serve pipeline status buckets + monthly trends with time-window filters.

### **⚙️ Implementation**

**Endpoint**: `GET /api/v1/itinerary-pipeline`

**Query Params**:
- `time_window` (e.g., `3m`, `6m`, `12m`)

**Response Shape**:
```json
{
  "data": {
    "summary": [
      { "itineraryStatus": "Deposited/Confirming", "itineraryCount": 42 }
    ],
    "timeline": [
      {
        "periodStart": "2026-01-01",
        "itineraryStatus": "Deposited/Confirming",
        "itineraryCount": 10
      }
    ]
  },
  "pagination": null,
  "meta": { "timeWindow": "6m", "asOfDate": "YYYY-MM-DD" }
}
```

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `src/schemas/itinerary_pipeline.py` | Create | Response + filter schemas |
| `src/repositories/itinerary_pipeline_repository.py` | Create | Query view with date filter |
| `src/services/itinerary_pipeline_service.py` | Create | Business logic + aggregation |
| `src/api/itinerary_pipeline.py` | Create | FastAPI route |
| `src/api/router.py` | Modify | Register route |

### **✅ Validation Checklist**
- [ ] Response uses `{ data, pagination, meta }`
- [ ] `time_window` respected
- [ ] Status buckets are correct

---

## 3️⃣ **PHASE 3: UI Integration**
*Priority: HIGH - Build pipeline dashboard with filters*

### **🎯 Objective**
Replace sparse revenue/bookings pipeline with status-driven charts and tables.

### **⚙️ Implementation**
- Add filter controls for 3/6/12 months
- Render:
  - Status summary cards
  - Monthly trend chart by status
  - Detail table by month/status

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `apps/web/src/lib/api/itineraryPipelineService.ts` | Create | API client |
| `apps/web/src/lib/types/itineraryPipeline.ts` | Create | Types |
| `apps/web/src/features/revenue-bookings/itinerary-pipeline.tsx` | Create | Pipeline UI |
| `apps/web/src/app/revenue-bookings/page.tsx` | Modify | Mount new pipeline view |

### **✅ Validation Checklist**
- [ ] Filter toggles update timeline
- [ ] Status labels match backend enum values
- [ ] Empty/loading/error states present

---

## ⚠️ **RISK MANAGEMENT**

### **High Priority Risks**
- **Status drift**: New status values appear → **Mitigation**: use enum values from Supabase and handle unknowns
- **Performance**: Large table scans → **Mitigation**: materialized view + refresh cadence

### **Rollback Strategy**
1. Remove new pipeline endpoint from router
2. Revert view migration
3. Fall back to existing trend view

---

## 📊 **SUCCESS CRITERIA**

### **Technical Success Metrics**
| Metric | Target | Verification Method |
|--------|--------|---------------------|
| View refresh time | < 2 min | DB logs |
| API response time | < 500ms | Backend logs |
| ESLint | Zero warnings | `npm run lint` |

### **User Experience Success**
| Scenario | Expected Outcome |
|----------|------------------|
| Toggle 3/6/12 months | Pipeline updates correctly |
| Review status buckets | Counts match expected statuses |
| Scan trend table | Month-over-month changes visible |

---

## 📚 **DOCUMENTATION UPDATES**

| Document | Section | Change Description |
|----------|---------|-------------------|
| `docs/sample-payloads.md` | Itinerary pipeline | Add request/response examples |
| `docs/swainos-code-documentation-backend.md` | FX/Revenue section | Add pipeline API + view |
| `docs/swainos-code-documentation-frontend.md` | Revenue & Bookings | Note pipeline UI + filters |

---

## 📝 **REVISION HISTORY**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.0 | 2026-02-10 | SwainOS Assistant | Initial action plan |
