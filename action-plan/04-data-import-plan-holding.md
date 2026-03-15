# 🎯 Data Import - Supabase Historical Load
> **Version**: v1.3  
> **Status**: 🔄 IN PROGRESS  
> **Date**: 2026-02-09  
> **Completion Date**: —

**Target Components**: `SwainOS_BackEnd/supabase/migrations/`, Supabase tables (`contacts`, `agencies`, `suppliers`, `itineraries`, `bookings`, `itinerary_items`, `customer_payments`, `supplier_invoices`, `supplier_invoice_lines`, `locations_raw`, `locations`)  
**Primary Issues**: Historical data must be loaded in a safe, repeatable order with external ID mapping.  
**Objective**: Import historical data into Supabase with stable `external_id` mapping, validated relationships, consistent `contact_type` usage, and agency consortia tagging.

## 📋 QUICK SUMMARY
**What We're Building/Fixing**: A sequenced import runbook for loading historical Salesforce/Kaptio data into Supabase.

**Critical Issues Being Addressed**:
- Foreign key dependencies → define strict import order.
- ID mapping → enforce `external_id`-based upserts.
- Location normalization → use existing raw→canonical location pipeline.
- Contact identity → standardize `contact_type` values.
- Consortia analytics → ensure agencies include `consortia` values.

**Success Metrics**:
- All core tables populated with valid FK references.
- `external_id` uniqueness preserved across imports.
- Cashflow and forecast endpoints return data without errors.

---

## 🎯 EXECUTION STATUS
**Progress**: 3 of 4 sections completed  
**Current Status**: Core entities complete (including bookings and itinerary items). Financials in progress (customer payments done; supplier invoices pending).

| Phase | Status | Priority | Notes |
|-------|---------|----------|-------|
| 1️⃣ Preflight & Environment | ✅ COMPLETED | HIGH | Migrations applied and service role ready |
| 2️⃣ Reference + Locations | ✅ COMPLETED | HIGH | Reference data seeded; locations deferred |
| 3️⃣ Core Entities | ✅ COMPLETED | HIGH | Agencies/contacts/suppliers/itineraries/bookings/items done |
| 4️⃣ Financials + Validation | 🔄 IN PROGRESS | HIGH | Customer payments done; supplier invoices pending |

---

## 🚨 CRITICAL REQUIREMENTS
### ⚠️ MANDATORY CHECKLIST - Must Complete Before Import
These requirements are NON-NEGOTIABLE for the import run.

- [x] **Migrations Applied**: All schema migrations through `0015_add_customer_payments_card_type.sql` are applied.
- [x] **External ID Standard**: Use `external_id` for upserts in every core table.
- [x] **Contact Type Standard**: Use `AGENT` for trade itineraries, `DIRECT` for direct itineraries.
- [x] **Consortia Standard**: Populate `agencies.consortia` for segmentation.
- [x] **Service Role**: Imports run with Supabase service role key (RLS-safe).
- [x] **FK Safety**: Imports follow the defined order to avoid FK failures.
- [ ] **Auditability**: Track counts inserted/updated per table.

---

## 1️⃣ Preflight & Environment
*Priority: High - confirm schema and permissions*

### 🎯 Objective
Ensure database schema and security settings are ready for bulk imports.

### ⚙️ Implementation
**Steps:**
1. Apply migrations up to `0015_add_customer_payments_card_type.sql`.
2. Confirm `external_id` exists on all core tables and is unique.
3. Confirm service role access for REST upserts.
4. Confirm `contact_type` values are restricted to `AGENT` and `DIRECT` in imports.
5. Prepare import logs (counts per table).

### ✅ Validation Checklist
- [ ] `external_id` columns present and unique.
- [ ] RLS policies allow service role inserts.

---

## 2️⃣ Reference + Locations
*Priority: High - establish canonical reference data*

### 🎯 Objective
Load reference data and normalize locations.

### ⚙️ Implementation
**Tables:**
- `currencies`
- `locations_raw` → `locations` via `upsert_locations_from_raw()`

**Steps:**
1. Load any additional reference data not already seeded.
2. Insert Salesforce/Kaptio locations into `locations_raw`.
3. Run `public.upsert_locations_from_raw()` to populate `locations`.

### ✅ Validation Checklist
- [ ] `locations` populated and de-duplicated.
- [ ] `location_mappings` created for all raw rows.

---

## 3️⃣ Core Entities
*Priority: High - load CRM + booking graph*

### 🎯 Objective
Import core CRM entities and booking graph with FK integrity.

### ⚙️ Implementation (strict order)
1. `agencies`
2. `contacts` (requires `agency_id` for trade contacts)
3. `suppliers`
4. `itineraries` (requires `agency_id`, `primary_contact_id`, `primary_contact_type`)
5. `bookings` (requires `itinerary_id`, `supplier_id`)
6. `itinerary_items` (requires `itinerary_id`, `supplier_id`)

**Rules:**
- Upsert on `external_id`.
- Use returned Supabase IDs to populate FK columns.
- Set `primary_contact_type` to `AGENT` for trade itineraries and `DIRECT` for direct.
- Set `contact_type` on `contacts` to the same `AGENT` or `DIRECT` values.
- Populate `agencies.consortia` for trade agencies when available.
- Set `bookings.is_deleted` from KaptioTravel__isDeleted__c and filter deleted rows in analytics.

### ✅ Validation Checklist
- [ ] No orphaned FK rows.
- [ ] `external_id` uniqueness preserved.

### ✅ Current Status
- Agencies, contacts, suppliers, itineraries, bookings, and itinerary items imported.

---

## 4️⃣ Financials + Validation
*Priority: High - load payments/invoices and verify outputs*

### 🎯 Objective
Import financial flows and verify analytics outputs.

### ⚙️ Implementation
**Tables:**
1. `customer_payments` (requires `itinerary_id`)
2. `supplier_invoices` (requires `supplier_id`)
3. `supplier_invoice_lines` (requires `supplier_invoice_id`, `booking_id`, `itinerary_id`)

**Validation:**
- Run cashflow endpoints: `/api/v1/cash-flow/summary` and `/api/v1/cash-flow/timeseries`
- Confirm deposits and payments out endpoints return data.

### ✅ Validation Checklist
- [ ] Financial tables populated with valid FK references.
- [ ] API endpoints return non-empty data.

---

## 🔧 REST Ingestion Plan (Remaining Loads)
*Use REST upserts for large CSVs and idempotent loads.*

### 🎯 Objective
Load remaining datasets via Supabase REST with `on_conflict=external_id`.

### ⚙️ Implementation
**Approach:**
1. Convert CSV → JSON and chunk (1k–5k rows).
2. POST to `/rest/v1/<table>?on_conflict=external_id`.
3. Use headers:
   - `apikey: <SERVICE_ROLE_KEY>`
   - `Authorization: Bearer <SERVICE_ROLE_KEY>`
   - `Prefer: resolution=merge-duplicates`

**Target tables:**
- `supplier_invoices`
- `supplier_invoice_lines`

### ✅ Validation Checklist
- [ ] Each batch returns 201/204 without errors.
- [ ] Row counts match CSV totals.
- [ ] No duplicate `external_id` values remain.

---

## ⚠️ RISK MANAGEMENT
### High Priority Risks
- **FK failures**: wrong insert order → **Mitigation**: enforce import order above.
- **External ID collisions**: duplicates overwrite data → **Mitigation**: pre-check uniqueness before load.

### Rollback Strategy
1. Roll back imported batches in reverse order.
2. Re-run imports using corrected mapping.
3. Validate endpoints and counts again.

---

## 📊 SUCCESS CRITERIA
### Technical Success Metrics
| Metric | Target | Verification Method |
|--------|--------|---------------------|
| External ID integrity | No duplicates | DB uniqueness checks |
| FK integrity | Zero orphaned rows | FK validation queries |
| API readiness | Non-empty results | Manual endpoint checks |

### User Experience Success
| Scenario | Expected Outcome |
|----------|------------------|
| View cashflow dashboard | Real historical data appears |
| View bookings | Bookings and suppliers align correctly |

---

## 🔗 RELATED DOCUMENTATION
- `../docs/success-criteria-and-phases.md`
- `../docs/swainos-code-documentation-backend.md`

---

## 📝 REVISION HISTORY
| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.0 | 2026-02-07 | SwainOS Team | Initial data import action plan |
| v1.1 | 2026-02-07 | SwainOS Team | Updated for contacts and primary_contact_id |
| v1.2 | 2026-02-09 | SwainOS Team | Marked progress and added REST ingestion plan |
