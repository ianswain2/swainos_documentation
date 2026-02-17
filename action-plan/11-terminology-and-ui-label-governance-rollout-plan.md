# 🎯 Terminology and UI Label Governance Rollout Plan - Cross-Platform Vocabulary Standardization

> **Version**: v1.0  
> **Status**: 🚀 READY TO IMPLEMENT  
> **Date**: 2026-02-17  
> **Completion Date**: N/A

**Target Components**: `SwainOS_FrontEnd/apps/web/src/**`, `SwainOS_BackEnd/src/**`, `SwianOS_Documentation/docs/**`, `SwianOS_Documentation/action-plan/**`  
**Primary Issues**: User-facing naming variance across titles, metric labels, filters, toggles, and backend terminology references (for example: gross profit vs commission income vs income, margin amount vs margin percent).  
**Objective**: Define one canonical SwainOS terminology standard and enforce it across frontend UI, backend contracts/metadata, and documentation so users see consistent labels everywhere.

## 📋 **QUICK SUMMARY**

**What We're Building/Fixing**: A durable terminology governance system with a canonical glossary, rollout plan, and implementation checkpoints across frontend, backend, and documentation.

**Critical Issues Being Addressed**:
- Multiple terms for the same KPI (for example, gross profit vs commission income) → one canonical display vocabulary
- Ambiguous metric variants (for example, margin amount vs margin %) → explicit label rules and formatting standards
- Inconsistent UI controls and section naming by module → shared title/filter/toggle naming baseline
- Drift between UI wording and backend/source contracts → documented mapping rules and optional backend alias strategy

**Success Metrics**:
- One canonical glossary document exists and is linked from both frontend and backend code documentation
- Frontend labels for common metrics are aligned to canonical terms across major modules
- Backend contract documentation explicitly maps internal field names to canonical display terms
- New UI/endpoint additions follow glossary-first naming review before merge

---

## 🎯 **EXECUTION STATUS**

**Progress**: 0 of 4 sections completed  
**Current Status**: Plan approved; begin glossary lock and inventory baseline

| Phase | Status | Priority | Notes |
|-------|---------|----------|-------|
| 1️⃣ Canonical Glossary Lock | 📋 PENDING | HIGH | Finalize approved definitions and aliases |
| 2️⃣ Frontend Label Alignment | 📋 PENDING | HIGH | Standardize titles, labels, filters, toggles in all active modules |
| 3️⃣ Backend Contract + Documentation Alignment | 📋 PENDING | HIGH | Preserve API stability while aligning terminology references |
| 4️⃣ Governance and Regression Controls | 📋 PENDING | MEDIUM | Add checklist and review guardrails for future consistency |

---

## 🚨 **CRITICAL REQUIREMENTS**

### **⚠️ MANDATORY CHECKLIST - Must Complete Before Implementation**

- [ ] **Canonical Definitions Approved**: Leadership/stakeholders approve glossary terms before broad refactor
- [ ] **Frontend Consistency**: Shared metrics use one exact display label across all pages
- [ ] **Contract Safety**: Existing backend response field names remain compatible unless explicitly approved for migration
- [ ] **Documentation Synced**: Frontend and backend code documentation both link to and reference the glossary
- [ ] **Quality Gates**: Lint/type/build checks remain clean after implementation changes
- [ ] **No Dead Copy**: Legacy labels removed from active UI and documentation where replaced

### **Documentation Update Requirement**

> **Required docs for this rollout**:
> - `docs/swainos-terminology-glossary.md` (new canonical source)
> - `docs/swainos-code-documentation-frontend.md` (link + usage rule reference)
> - `docs/swainos-code-documentation-backend.md` (link + contract-mapping reference)
> - `docs/frontend-data-queries.md` and `docs/sample-payloads.md` updates in implementation phase if contract language references need harmonization

---

## 🔧 **STRATEGIC APPROACH**

### **Implementation Philosophy**
Terminology consistency is treated as product infrastructure, not cosmetic copy cleanup. We standardize definitions first, then map every surface (frontend UI, backend contract language, docs, and AI narrative labels) to that single source of truth.

### **Key Architecture Decisions**
- **Glossary-first governance**: Every shared metric/title/filter term is defined in one canonical glossary document.
- **Display vs contract separation**: UI labels can standardize to user-friendly language while backend field keys remain stable for compatibility.
- **Centralized frontend constants**: Shared labels/toggle options should move into common constants to prevent drift across feature modules.
- **Backward-safe backend updates**: If backend terminology updates are needed, add aliases/metadata before replacing any external contract key.

---

## 1️⃣ **CANONICAL GLOSSARY LOCK**
*Priority: High - Establish unambiguous shared vocabulary before code-wide changes*

### **🎯 Objective**
Finalize and publish canonical metric/title/filter definitions with alias history and explicit usage examples.

### **🔍 Analysis / Discovery**
- Validate current terms used in:
  - Command Center
  - Itinerary Forecast / Itinerary Actuals
  - Travel Consultant leaderboard/profile
  - AI Insights panels and recommendation cards
- Confirm known canonical rule already documented:
  - UI uses **Gross Profit**
  - API field remains `commissionIncomeAmount`
- Identify all ambiguous terms requiring hard definitions:
  - Margin (amount) vs Margin %
  - Conversion Rate vs Close Rate
  - YoY Variance vs Target Variance
  - Liability/Payables wording

### **⚙️ Implementation**

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `docs/swainos-terminology-glossary.md` | Create | Canonical terms, definitions, aliases, display rules |
| `docs/swainos-code-documentation-frontend.md` | Modify | Add glossary reference and adoption rule |
| `docs/swainos-code-documentation-backend.md` | Modify | Add glossary reference and contract/display mapping rule |

**Implementation Steps:**
1. Publish glossary with metric families, canonical display names, and prohibited legacy variants.
2. Include backend contract mapping section (field key -> canonical UI term).
3. Add change-control rules for introducing any new metric terminology.

### **✅ Validation Checklist**
- [ ] Glossary reviewed and approved
- [ ] Existing duplicate terms captured as aliases/deprecated variants
- [ ] Frontend and backend docs both link to glossary

---

## 2️⃣ **FRONTEND LABEL ALIGNMENT**
*Priority: High - Remove user-visible wording variance across UI*

### **🎯 Objective**
Apply canonical terminology across all active frontend modules and controls.

### **🔄 Implementation**
- Standardize:
  - Page titles
  - Section headers
  - KPI labels
  - Filter names
  - Toggle option labels
  - Empty-state messaging where terminology appears
- Introduce shared label constants for common KPI/filter terms used in multiple modules.
- Ensure the same metric always renders with the same display string.

### **Frontend Scope (initial wave)**
- `features/command-center/*`
- `features/itinerary-forecast/*`
- `features/itinerary-actuals/*`
- `features/itinerary-shared/*`
- `features/travel-consultant/*`
- `features/ai-insights/*`
- Shared UI primitives using metric/title props where hardcoded labels exist

### **✅ Validation Checklist**
- [ ] Canonical labels match glossary exactly
- [ ] No mixed synonyms for shared KPIs on active screens
- [ ] Manual UX pass confirms consistency across modules

---

## 3️⃣ **BACKEND CONTRACT + DOCUMENTATION ALIGNMENT**
*Priority: High - Keep contracts stable while improving terminology clarity*

### **🎯 Objective**
Align backend naming semantics to glossary without unsafe API breakage.

### **🧪 Backend Strategy**
- Keep existing response keys stable unless migration is explicitly approved.
- Prefer contract-safe alignment methods:
  - Response metadata notes
  - Documentation mapping tables
  - Optional alias fields for new consumers when needed
- Ensure AI context/recommendation generation references canonical terminology for user-facing narratives.

### **Potential Backend Touchpoints**
- `src/services/*` and `src/repositories/*` where user-visible text is generated
- `src/schemas/*` where descriptions can clarify canonical term meaning
- AI orchestration prompts/output normalization to follow glossary terms

### **✅ Validation Checklist**
- [ ] No unplanned API contract breakage
- [ ] Backend docs clearly map field keys to canonical display terms
- [ ] AI-generated user-facing copy uses canonical vocabulary

---

## 4️⃣ **GOVERNANCE AND REGRESSION CONTROLS**
*Priority: Medium - Prevent terminology drift after rollout*

### **🎯 Objective**
Make terminology consistency enforceable in future changes.

### **📚 Documentation and Process Controls**
- Add a PR checklist item: "Terminology aligns to `swainos-terminology-glossary.md`."
- Require glossary update for any new cross-module KPI/filter/title terms.
- Add periodic consistency audit pass across frontend labels and AI output copy.

### **✅ Validation Checklist**
- [ ] Review checklist updated
- [ ] Ownership assigned for glossary governance
- [ ] Drift detection audit cadence defined

---

## ⚠️ **RISK MANAGEMENT**

### **High Priority Risks**
- **Contract confusion**: UI terms diverge from API keys in developer understanding → **Mitigation**: explicit field-to-display mapping table in glossary and backend docs
- **Partial rollout inconsistency**: some pages updated while others lag → **Mitigation**: phased checklist by module with completion tracking

### **Medium Priority Risks**
- **AI narrative drift**: generated recommendation copy uses legacy terms → **Mitigation**: prompt/output normalization against glossary terminology set

### **Rollback Strategy**
1. Revert only label copy/constants changes if UX confusion appears.
2. Keep glossary as baseline artifact and iterate terminology set with stakeholder review.
3. Re-run consistency audit before re-release.

---

## 📊 **SUCCESS CRITERIA**

### **Technical Success Metrics**

| Metric | Target | Verification Method |
|--------|--------|---------------------|
| Canonical glossary coverage | 100% of shared KPI/filter/title terms defined | Glossary audit checklist |
| Frontend terminology consistency | No conflicting labels for same concept in active modules | UI string inventory pass |
| Backend terminology mapping clarity | Every canonical metric mapped to contract field(s) | Backend docs review |
| Documentation linkage | Glossary linked in FE + BE docs | TOC and section verification |

### **User Experience Success**

| Scenario | Expected Outcome |
|----------|------------------|
| User moves between Command Center and Travel Consultant pages | Shared metrics use identical names and meaning |
| User checks Itinerary Forecast vs Itinerary Actuals | Metric labels remain consistent and unambiguous |
| User reads AI recommendation and then KPI surfaces | Terminology matches exactly (no synonym confusion) |

---

## 🔗 **RELATED DOCUMENTATION**

- `./action-plan-template.md` - Standard action-plan format baseline
- `../docs/swainos-code-documentation-frontend.md` - Frontend architecture and module references
- `../docs/swainos-code-documentation-backend.md` - Backend contract and service references
- `../docs/frontend-data-queries.md` - Current frontend-to-backend query contract inventory

---

## 📝 **REVISION HISTORY**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.0 | 2026-02-17 | AI Agent + Ian | Initial terminology governance rollout plan |
