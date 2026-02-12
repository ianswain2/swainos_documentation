# 🎯 FX COMMAND BACKEND ACTION PLAN - Live Rates, Exposure, Trade Logging, AI Insights

> **Version**: v1.0  
> **Status**: 📋 PLANNING  
> **Date**: 2026-02-10  
> **Completion Date**: [YYYY-MM-DD]

**Target Components**: `SwainOS_BackEnd/src/api/`, `SwainOS_BackEnd/src/repositories/`, `SwainOS_BackEnd/src/services/`, `SwainOS_BackEnd/src/schemas/`, `SwianOS_Documentation/supabase/migrations/`  
**Primary Issues**: Live FX data ingestion, supplier exposure aggregation, trade logging (buy/sell), AI insights storage, forecasting pipeline  
**Objective**: Deliver a complete FX backend that provides live rates, exposure rollups for ZAR/USD/AUD/NZD, durable trade logging, AI insights history, and forecast-ready datasets.

## 📋 **QUICK SUMMARY**

**What We're Building/Fixing**: A backend FX platform that ingests live rates, aggregates exposure, supports buy/sell logging, and stores AI insights + forecasts.

**Critical Issues Being Addressed**:
- **Live rates feed** → Automated ingestion to `fx_rates` with filters for ZAR/USD/AUD/NZD
- **Exposure accuracy** → `mv_fx_exposure` tied to supplier invoices/lines with deterministic refresh
- **Trade logging** → Buy/sell entries across any currency with clear audit trail and linkage to invoices/signals
- **AI insights + forecasts** → Persistent storage for model output and reasoning, not just live computation

**Success Metrics**:
- FX rates endpoint returns live ZAR/USD/AUD/NZD pairs with timestamps < 1 hour old
- Exposure endpoint returns accurate confirmed/estimated exposure per currency
- Buy/sell trades can be logged, retrieved, and reconciled against holdings
- AI insights are stored and auditable with model versions and timestamps
- Forecast endpoints return stable, reproducible outputs

---

## 🎯 **EXECUTION STATUS**

**Progress**: 0 of 4 sections completed  
**Current Status**: Planning and architecture definition

| Phase | Status | Priority | Notes |
|-------|---------|----------|-------|
| 1️⃣ Data Architecture & Schema | 📋 PENDING | HIGH | Define storage vs live, tables, and migrations |
| 2️⃣ Ingestion & Aggregation | 📋 PENDING | HIGH | Live rates feed + exposure rollups |
| 3️⃣ Trade Logging & Insights | 📋 PENDING | HIGH | Buy/sell logging + AI insights + forecasts |
| 4️⃣ API Contracts & Validation | 📋 PENDING | HIGH | Endpoints + docs + QA |

---

## 🚨 **CRITICAL REQUIREMENTS**

### **⚠️ MANDATORY CHECKLIST - Must Complete Before Implementation**

- [ ] **Type Safety**: All new code uses explicit types — NO `any`
- [ ] **Naming Conventions**: All files, functions, and variables follow SwainOS standards
- [ ] **Import Organization**: Standard grouping and order
- [ ] **ESLint Clean**: Zero warnings, zero errors before PR
- [ ] **Documentation Update**: `docs/swainos-code-documentation-backend.md` and `docs/sample-payloads.md` updated
- [ ] **No Dead Code**: No unused imports, no commented-out code

### **Documentation Update Requirement**
- Backend changes → Update `docs/swainos-code-documentation-backend.md`
- Schema changes → Update documentation + migration notes

---

## 📐 **NAMING CONVENTION ALIGNMENT**

Follow SwainOS backend standards:
- Files/modules: `snake_case.py`
- Classes: `PascalCase`
- Functions/variables: `snake_case`
- Constants: `SCREAMING_SNAKE_CASE`
- Tables/columns: `snake_case`
- API endpoints: `kebab-case`

---

## 🧹 **CLEAN CODE REQUIREMENTS**

- No magic numbers without constants
- No silent error swallowing; log + return safe envelopes
- Consistent response envelopes `{ data, pagination, meta }`
- All Supabase queries parameterized via repository layer

---

## 🔧 **STRATEGIC APPROACH**

### **Implementation Philosophy**
Keep FX backend deterministic and auditable. Live data is ingested into durable storage (`fx_rates`), exposure uses materialized views, and AI insights + forecasts are persisted to allow retrospective analysis.

### **Key Architecture Decisions**
- **Store live rates** in `fx_rates` (time-series) rather than only real-time fetches.
- **Scope currencies** to ZAR/USD/AUD/NZD at every layer.
- **Trade logging** uses `fx_transactions` with normalized base/quote fields.
- **AI insights** stored in a dedicated table with model/version metadata.

### **Data Flow**
```
Rate Provider -> Ingestion Job -> fx_rates
Supplier Invoices -> mv_fx_exposure -> /api/v1/fx/exposure
fx_transactions -> holdings + exposure adjustments
AI Model -> fx_ai_insights + fx_forecasts
```

---

## 1️⃣ **PHASE 1: Data Architecture & Schema**
*Priority: HIGH - Define source of truth for rates, trades, and AI insights*

### **🎯 Objective**
Establish DB structure to support live rates, buy/sell logging, AI insights history, and forecasting.

### **🔍 Analysis / Discovery**
Current tables:
- `fx_rates` (time series)
- `fx_holdings` (positions)
- `fx_signals` (signal outputs)
- `fx_transactions` (trades)

Gaps:
- No normalized base/quote fields for trades
- No AI insights history table
- No forecast outputs table
- Exposure view uses invoices but not invoice lines

### **⚙️ Implementation**

**Schema Updates (Migrations):**
- Add columns to `fx_transactions`:
  - `base_currency_code`, `quote_currency_code`, `trade_side` (BUY/SELL), `notional_amount`, `executed_rate`
  - `execution_source` (manual, AI, external)
- New table `fx_ai_insights`:
  - `currency_code`, `insight_type`, `insight_strength`, `current_rate`, `avg_30d_rate`, `recommended_action`, `recommended_amount`, `reasoning`, `model_version`, `generated_at`, `expires_at`
- New table `fx_forecasts`:
  - `currency_code`, `forecast_horizon`, `forecast_date`, `forecast_rate`, `confidence`, `model_version`, `generated_at`
- Optional: `fx_rate_snapshots` or materialized view for daily OHLC (if needed for forecasting)

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `supabase/migrations/00xx_fx_transactions_ext.sql` | Create | Add normalized trade fields |
| `supabase/migrations/00xx_fx_ai_insights.sql` | Create | Add AI insights table |
| `supabase/migrations/00xx_fx_forecasts.sql` | Create | Add forecast table |
| `docs/swainos-code-documentation-backend.md` | Update | Document FX schema and storage decisions |

### **✅ Validation Checklist**
- [ ] Migrations apply cleanly
- [ ] New tables/columns exist and are queryable
- [ ] RLS policies updated for new tables

---

## 2️⃣ **PHASE 2: Ingestion & Aggregation**
*Priority: HIGH - Live rates + exposure aggregation*

### **🎯 Objective**
Implement ingestion pipelines for FX rates and exposure rollups.

### **⚙️ Implementation**

**Live Rates Ingestion:**
- Add provider integration (Oanda/Fixer/ECB/etc.)
- Scheduled ingestion job:
  - Fetch rates for ZAR/USD/AUD/NZD pairs only
  - Upsert into `fx_rates`
  - Retain time-series history

**Exposure Aggregation:**
- Update/extend `mv_fx_exposure`:
  - Confirmed exposure based on supplier invoices (or invoice lines once available)
  - Estimated exposure tied to itinerary status weights
  - Holdings offset to compute net exposure
- Define refresh cadence for materialized views (e.g., hourly)

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `src/integrations/fx_rates/*` | Create | Provider client + ingestion job |
| `supabase/migrations/00xx_mv_fx_exposure.sql` | Modify | Adjust exposure logic if needed |
| `src/repositories/fx_repository.py` | Modify | Support filters for currency scope |

### **✅ Validation Checklist**
- [ ] Rates ingestion inserts new rows into `fx_rates`
- [ ] `mv_fx_exposure` returns ZAR/USD/AUD/NZD only
- [ ] Exposure numbers reconcile with invoices

---

## 3️⃣ **PHASE 3: Trade Logging & AI Insights**
*Priority: HIGH - Buy/sell logging + AI insights storage*

### **🎯 Objective**
Enable durable buy/sell logging and storage of AI insights and forecasts.

### **⚙️ Implementation**

**Trade Logging:**
- Implement POST `/api/v1/fx/transactions` to log BUY/SELL trades
- Link trades to `supplier_invoice_id` and `signal_id` when relevant
- Update holdings automatically after trade insert

**AI Insights + Forecasts:**
- Implement POST `/api/v1/fx/insights` and `/api/v1/fx/forecasts` to store AI outputs
- Ensure `model_version` is always stored

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `src/api/fx.py` | Modify | Add POST routes for trades + insights + forecasts |
| `src/services/fx_service.py` | Modify | Business rules for logging + validation |
| `src/schemas/fx.py` | Modify | Add create payload schemas |

### **✅ Validation Checklist**
- [ ] Trades can be logged and retrieved
- [ ] AI insights stored with model metadata
- [ ] Forecast outputs stored and queryable

---

## 4️⃣ **PHASE 4: API Contracts & Validation**
*Priority: HIGH - Ensure clean API contracts and docs*

### **🎯 Objective**
Finalize API contracts for FX and ensure consistent documentation + testing.

### **⚙️ Implementation**

**API Endpoints:**
- GET `/api/v1/fx/rates` (filtered pairs only)
- GET `/api/v1/fx/exposure`
- POST `/api/v1/fx/transactions`
- GET `/api/v1/fx/transactions`
- POST `/api/v1/fx/insights`
- GET `/api/v1/fx/insights`
- POST `/api/v1/fx/forecasts`
- GET `/api/v1/fx/forecasts`

**Documentation Updates:**
| Document | Section | Change Description |
|----------|---------|-------------------|
| `docs/swainos-code-documentation-backend.md` | FX section | Schema + ingestion + API behavior |
| `docs/sample-payloads.md` | FX endpoints | Add request/response examples |

### **✅ Validation Checklist**
- [ ] All endpoints return standard envelope
- [ ] Swagger/fastapi docs are clear
- [ ] Input validation errors are structured

---

## ⚠️ **RISK MANAGEMENT**

### **High Priority Risks**
- **Live rate provider failure** → **Mitigation**: fall back to last known rate, surface status in API meta
- **Exposure mismatch** → **Mitigation**: enforce reconciliation jobs + audit logs
- **Trade logging errors** → **Mitigation**: transactional inserts + validation checks

### **Rollback Strategy**
1. Disable ingestion job and revert to last known `fx_rates`
2. Roll back migrations if required
3. Verify API endpoints still respond with empty data instead of 500

---

## 📊 **SUCCESS CRITERIA**

### **Technical Success Metrics**
| Metric | Target | Verification Method |
|--------|--------|---------------------|
| FX rates ingestion | 100% success | Scheduled job logs |
| Exposure accuracy | Matches invoices | Audit query comparison |
| Trade logging | No failed inserts | API tests |
| AI insights storage | 100% persisted | DB audit |

### **User Experience Success**
| Scenario | Expected Outcome |
|----------|------------------|
| View FX rates | ZAR/USD/AUD/NZD only with timestamps |
| Log a trade | Immediate confirmation + holdings update |
| Review AI insights | History with reasoning + model version |

---

## 🔗 **RELATED DOCUMENTATION**

- **[Frontend Buildout Plan](./05-frontend-initial-buildout-plan.md)** - Frontend reference
- **[Supabase Schema](../supabase/migrations/0001_initial_schema.sql)** - FX table definitions
- **[Materialized Views](../supabase/migrations/0003_materialized_views.sql)** - Exposure view

---

## 📋 **COMPLETION CHECKLIST**

### **Pre-Implementation**
- [ ] Confirm rate provider and cadence
- [ ] Confirm supported currency pairs

### **Implementation Quality Gates**
- [ ] Types explicit, no `any`
- [ ] ESLint passes
- [ ] Naming conventions followed

### **Testing**
- [ ] Manual tests for all FX endpoints
- [ ] Error handling for missing data
- [ ] Exposure reconciliation

### **Documentation** *(MANDATORY)*
- [ ] `docs/swainos-code-documentation-backend.md` updated
- [ ] `docs/sample-payloads.md` updated

---

## 📝 **REVISION HISTORY**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.0 | 2026-02-10 | SwainOS Assistant | Initial action plan |
