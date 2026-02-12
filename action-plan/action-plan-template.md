# 🎯 [ACTION PLAN TITLE] - [Brief Description]

> **Version**: v1.0  
> **Status**: [📋 PLANNING | 🚀 READY TO IMPLEMENT | 🔄 IN PROGRESS | ✅ COMPLETED]  
> **Date**: [YYYY-MM-DD]  
> **Completion Date**: [YYYY-MM-DD] *(for completed plans)*

**Target Components**: [List files/directories being modified: `lib/api/`, `features/itinerary-builder/`, etc.]  
**Primary Issues**: [Brief description of problems being solved]  
**Objective**: [Clear, measurable goal of this action plan]  

## 📋 **QUICK SUMMARY**

**What We're Building/Fixing**: [One sentence summary of the main deliverable]

**Critical Issues Being Addressed**:
- [Issue 1] → [Solution]
- [Issue 2] → [Solution]
- [Issue 3] → [Solution]

**Success Metrics**: [List 3-5 key metrics that define success]

---

## 🎯 **EXECUTION STATUS**

**Progress**: [X of Y sections completed] *(update as work progresses)*  
**Current Status**: [Current phase and next steps]

| Phase | Status | Priority | Notes |
|-------|---------|----------|-------|
| 1️⃣ [Phase Name] | [📋 PENDING / 🔄 IN PROGRESS / ✅ COMPLETED] | [HIGH/MEDIUM/LOW] | [Brief status note] |
| 2️⃣ [Phase Name] | [📋 PENDING / 🔄 IN PROGRESS / ✅ COMPLETED] | [HIGH/MEDIUM/LOW] | [Brief status note] |
| 3️⃣ [Phase Name] | [📋 PENDING / 🔄 IN PROGRESS / ✅ COMPLETED] | [HIGH/MEDIUM/LOW] | [Brief status note] |

---

## 🚨 **CRITICAL REQUIREMENTS**

### **⚠️ MANDATORY CHECKLIST - Must Complete Before Implementation**

These requirements are **NON-NEGOTIABLE** for every action plan. Do not skip any item.

- [ ] **Type Safety**: All new code uses explicit TypeScript types — NO `any` types allowed
- [ ] **Naming Conventions**: All new files, functions, and variables follow TravelOS naming standards (see section below)
- [ ] **Import Organization**: All imports follow the standard grouping order
- [ ] **ESLint Clean**: Zero warnings, zero errors before PR submission
- [ ] **Documentation Update**: `code-documentation-frontend.md` and/or `code-documentation-backend.md` updated to reflect changes
- [ ] **No Dead Code**: No commented-out code, no unused imports, no unused variables

### **Documentation Update Requirement**

> **⚠️ IMPORTANT**: Every action plan that modifies code MUST include updates to the relevant documentation file(s):
> - Frontend changes → Update `docs/code-documentation-frontend.md`
> - Backend changes → Update `docs/code-documentation-backend.md`  
> - Schema changes → Update both documentation files AND migration notes

---

## 📐 **NAMING CONVENTION ALIGNMENT**

All code in this action plan MUST follow TravelOS naming conventions. Reference this section during implementation.

### **Files & Directories**

| Element | Convention | Example |
|---------|------------|---------|
| React component files | `kebab-case.tsx` | `itinerary-item-card.tsx` |
| UI primitive files | `lowercase.tsx` | `button.tsx` |
| Hook files | `useCamelCase.ts` | `useItinerary.ts` |
| Service files | `camelCaseService.ts` | `itineraryService.ts` |
| Store files | `camelCaseStore.ts` | `tripStore.ts` |
| Type definition files | `lowercase.ts` | `itinerary.ts` |
| API route files | `route.ts` | `route.ts` |
| Test files | `*.test.ts(x)` | `itinerary-card.test.tsx` |

### **TypeScript Naming**

| Element | Convention | Example |
|---------|------------|---------|
| Interfaces/Types | `PascalCase` | `ItineraryItem`, `TravelerProfile` |
| Enums | `PascalCase` | `TripStatus`, `ItemStatus` |
| Enum values | `SCREAMING_SNAKE_CASE` | `DRAFT`, `IN_PROGRESS` |
| Variables | `camelCase` | `itineraryItems`, `selectedTraveler` |
| Functions | `camelCase` with verb prefix | `calculateTotal()`, `fetchItinerary()` |
| Boolean variables | `is/has/can` prefix | `isLoading`, `hasError`, `canEdit` |
| Constants | `SCREAMING_SNAKE_CASE` | `MAX_TRAVELERS`, `DEFAULT_CURRENCY` |
| Event handlers | `handle` prefix | `handleSubmit`, `handleItemClick` |

### **Database & API**

| Element | Convention | Example |
|---------|------------|---------|
| Database tables | `snake_case`, plural | `itinerary_items` |
| Database columns | `snake_case` | `created_at`, `company_id` |
| API endpoints | `kebab-case` | `/api/v1/itineraries/:id/items` |
| JSON properties | `camelCase` | `startDate`, `companyId` |

---

## 🧹 **CLEAN CODE REQUIREMENTS**

### **Import Organization Standard**

All files MUST organize imports in this exact order with blank line separators:

```typescript
// 1. React and Next.js
import { useState, useMemo, useCallback } from 'react';
import { useRouter, useSearchParams } from 'next/navigation';

// 2. External libraries (alphabetized)
import { format } from 'date-fns';
import { z } from 'zod';

// 3. Internal: UI components
import { Button } from '@/components/ui/button';
import { StatusBadge } from '@/components/ui/status-badge';

// 4. Internal: Features and layouts
import { ItineraryCard } from '@/features/itinerary-builder/itinerary-card';

// 5. Internal: Lib (types, services, hooks, utils)
import type { Itinerary, ItineraryItem } from '@/lib/types';
import { itineraryService } from '@/lib/api/itineraryService';
import { useItineraryStore } from '@/lib/state/itineraryStore';
import { formatCurrency } from '@/lib/utils';

// 6. Relative imports (same feature/module)
import { ItemDetailPanel } from './item-detail-panel';
import type { LocalComponentProps } from './types';
```

### **Component File Structure**

Every React component file MUST follow this structure:

```typescript
// [imports - organized as above]

// Types/interfaces for this component
interface ComponentProps {
  // ... props definition
}

// Constants (if any)
const CONSTANT_VALUE = 'value';

// Component definition (exported)
export function ComponentName({ prop1, prop2 }: ComponentProps) {
  // 1. Hooks (useState, useEffect, custom hooks)
  const [state, setState] = useState<Type>(initial);
  const router = useRouter();
  
  // 2. Derived state / memoized values
  const computedValue = useMemo(() => /* ... */, [deps]);
  
  // 3. Event handlers
  const handleClick = useCallback(() => {
    // ...
  }, [deps]);
  
  // 4. Effects (if needed)
  useEffect(() => {
    // ...
  }, [deps]);
  
  // 5. Early returns (loading, error states)
  if (isLoading) return <LoadingSkeleton />;
  if (error) return <ErrorState error={error} />;
  
  // 6. Render
  return (
    // JSX
  );
}
```

### **Code Quality Gates**

| Gate | Requirement |
|------|-------------|
| TypeScript | No `any` types. Strict mode enabled. |
| ESLint | Zero warnings. Zero errors. |
| Unused Code | No dead code. No commented-out code. |
| Console Logs | No `console.log` in production code. Use proper error handling. |
| Magic Numbers | Extract to named constants. |
| Type Assertions | Avoid `as` casts. Use type guards instead. |

---

## 🚨 **CRITICAL ISSUES DISCOVERED** *(if applicable)*

### **[Issue Category 1]**

```typescript
// ❌ PROBLEM: [Description of the issue]
// File: path/to/file.ts
function problematicFunction(data: any) {
  return data.items.map((i: any) => i.name);
}

// ✅ SOLUTION: [Description of the fix]
interface DataResponse {
  items: Array<{ name: string }>;
}

function fixedFunction(data: DataResponse): string[] {
  return data.items.map((item) => item.name);
}
```

### **[Issue Category 2]**
- **Problem**: [Clear description]
- **Impact**: [Business/technical impact]
- **Solution**: [How it will be fixed]

---

## 🔧 **STRATEGIC APPROACH**

### **Implementation Philosophy**
[Paragraph explaining the overall approach and why this strategy was chosen]

### **Key Architecture Decisions**
- **[Decision 1]**: [Rationale and benefits]
- **[Decision 2]**: [Rationale and benefits]
- **[Decision 3]**: [Rationale and benefits]

### **Data Flow** *(if applicable)*

```
┌─────────────────────────────────────────────────────────┐
│ Component Layer                                          │
│   └── Uses hooks from stores                            │
├─────────────────────────────────────────────────────────┤
│ State Layer (Zustand Store)                             │
│   └── Calls service layer                               │
├─────────────────────────────────────────────────────────┤
│ Service Layer (lib/api/*)                               │
│   └── Uses httpClient with automatic JWT injection      │
├─────────────────────────────────────────────────────────┤
│ API Layer (/api/v1/*)                                   │
│   └── Validates with Zod, queries Supabase              │
└─────────────────────────────────────────────────────────┘
```

---

## 1️⃣ **[PHASE 1 NAME]**
*Priority: [High/Medium/Low] - [Brief description of phase focus]*

### **🎯 Objective**
[Clear statement of what this phase accomplishes]

### **🔍 Analysis / Discovery**
[Detailed analysis of current state, problems, or requirements]

### **⚙️ Implementation**

**Files to Create/Modify:**
| File | Action | Description |
|------|--------|-------------|
| `lib/api/newService.ts` | Create | New service for feature |
| `lib/types/feature.ts` | Modify | Add new types |
| `features/feature-name/component.tsx` | Create | New component |

**Implementation Steps:**
1. [Step 1 with specific details]
2. [Step 2 with specific details]
3. [Step 3 with specific details]

### **✅ Validation Checklist**
- [ ] TypeScript compiles without errors (`tsc --noEmit`)
- [ ] ESLint passes with zero warnings
- [ ] All new types are explicit (no `any`)
- [ ] Imports follow standard organization
- [ ] Component structure follows standard pattern

---

## 2️⃣ **[PHASE 2 NAME]**
*Priority: [High/Medium/Low] - [Brief description of phase focus]*

### **🎯 Objective**
[Clear statement of what this phase accomplishes]

### **🔄 Implementation**
[Core work being done in this phase]

### **✅ Validation Checklist**
- [ ] [Validation checkpoint 1]
- [ ] [Validation checkpoint 2]
- [ ] [Validation checkpoint 3]

---

## 3️⃣ **[PHASE 3 NAME]**
*Priority: [High/Medium/Low] - [Brief description of phase focus]*

### **🎯 Objective**
[Clear statement of what this phase accomplishes]

### **🧪 Testing**
[Testing approach and requirements]

### **📚 Documentation Updates**

**Required Documentation Changes:**

| Document | Section | Change Description |
|----------|---------|-------------------|
| `code-documentation-frontend.md` | [Section Name] | [What to add/update] |
| `code-documentation-backend.md` | [Section Name] | [What to add/update] |

### **✅ Validation Checklist**
- [ ] [Validation checkpoint 1]
- [ ] [Validation checkpoint 2]
- [ ] Documentation updated and accurate

---

## ⚠️ **RISK MANAGEMENT**

### **High Priority Risks**
- **[Risk 1]**: [Description] → **Mitigation**: [Strategy]
- **[Risk 2]**: [Description] → **Mitigation**: [Strategy]

### **Medium Priority Risks**
- **[Risk 3]**: [Description] → **Mitigation**: [Strategy]

### **Rollback Strategy**
1. [Step 1 for reverting changes]
2. [Step 2 for reverting changes]
3. [Verification step]

---

## 📊 **SUCCESS CRITERIA**

### **Technical Success Metrics**

| Metric | Target | Verification Method |
|--------|--------|---------------------|
| TypeScript compilation | Zero errors | `tsc --noEmit` |
| ESLint | Zero warnings | `npm run lint` |
| Feature functionality | All acceptance criteria met | Manual testing |
| Performance | No regressions | Browser DevTools |

### **User Experience Success**

| Scenario | Expected Outcome |
|----------|------------------|
| [User action 1] | [Expected result] |
| [User action 2] | [Expected result] |
| [User action 3] | [Expected result] |

---

## 🔗 **RELATED DOCUMENTATION**

- **[Related Action Plan](./XX-RELATED-PLAN.md)** - [Brief description]
- **[Architecture Principles](../docs/01-ARCHITECTURE-PRINCIPLES.md)** - Architecture reference
- **[Naming Conventions](../docs/02-NAMING-CONVENTIONS.md)** - Naming standards
- **[TypeScript Standards](../docs/04-TYPESCRIPT-STANDARDS.md)** - TypeScript reference

---

## 📚 **TECHNICAL REFERENCE**

### **Type Definitions**

```typescript
// Define new types here with full documentation
/**
 * Description of what this type represents.
 */
interface NewFeatureType {
  id: string;
  name: string;
  status: FeatureStatus;
  createdAt: string; // ISO datetime string
}

enum FeatureStatus {
  DRAFT = 'DRAFT',
  ACTIVE = 'ACTIVE',
  ARCHIVED = 'ARCHIVED'
}
```

### **Service Implementation**

```typescript
// Service method implementation example
import type { NewFeatureType } from '@/lib/types';
import { apiFetchEntity, apiFetchList } from './httpClient';

const base = process.env.NEXT_PUBLIC_API_BASE;

export const featureService = {
  async getById(id: string): Promise<NewFeatureType> {
    return apiFetchEntity<NewFeatureType>(`${base}/features/${id}`);
  },

  async create(data: CreateFeatureInput): Promise<NewFeatureType> {
    return apiFetchEntity<NewFeatureType>(`${base}/features`, {
      method: 'POST',
      body: JSON.stringify(data),
    });
  },
};
```

### **Component Implementation**

```typescript
// Component implementation example
import { useState, useCallback } from 'react';

import { Button } from '@/components/ui/button';

import type { NewFeatureType } from '@/lib/types';

interface FeatureCardProps {
  feature: NewFeatureType;
  onSelect: (id: string) => void;
}

export function FeatureCard({ feature, onSelect }: FeatureCardProps) {
  const [isHovered, setIsHovered] = useState(false);

  const handleClick = useCallback(() => {
    onSelect(feature.id);
  }, [feature.id, onSelect]);

  return (
    <div
      className="rounded-lg border p-4"
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
    >
      <h3 className="font-medium">{feature.name}</h3>
      <Button onClick={handleClick}>Select</Button>
    </div>
  );
}
```

---

## 🎯 **COMPLETION CHECKLIST**

### **Pre-Implementation**
- [ ] Read and understand all related existing code
- [ ] Verify requirements with stakeholders
- [ ] Review naming conventions and code standards

### **Implementation Quality Gates**
- [ ] All TypeScript types are explicit (NO `any`)
- [ ] All imports follow standard organization
- [ ] All components follow standard structure
- [ ] All naming follows TravelOS conventions
- [ ] ESLint passes with zero warnings
- [ ] No dead code or commented-out code
- [ ] No console.log statements

### **Testing**
- [ ] Core functionality tested manually
- [ ] Edge cases considered and handled
- [ ] Error states display correctly
- [ ] Loading states display correctly

### **Documentation** *(MANDATORY)*
- [ ] `code-documentation-frontend.md` updated (if frontend changes)
- [ ] `code-documentation-backend.md` updated (if backend changes)
- [ ] Inline code comments added where logic is complex
- [ ] Action plan status updated to ✅ COMPLETED

### **Final Review**
- [ ] All phases completed
- [ ] All validation checklists passed
- [ ] Documentation current and accurate
- [ ] No TODOs left in code

---

## 📝 **REVISION HISTORY**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.0 | [YYYY-MM-DD] | [Name/Team] | Initial action plan |

---

## 📋 **TEMPLATE USAGE NOTES**

### **How to Use This Template**

1. **Copy this template** for new action plans
2. **Replace all bracketed placeholders** with specific information
3. **Remove unused sections** that don't apply to your plan
4. **Keep all CRITICAL REQUIREMENTS** — they are non-negotiable
5. **Update status** as work progresses
6. **ALWAYS update documentation files** when implementation is complete

### **Template Sections Guide**

| Section | Purpose |
|---------|---------|
| Quick Summary | Executive overview for quick understanding |
| Execution Status | Track progress through phases |
| Critical Requirements | NON-NEGOTIABLE quality gates |
| Naming Convention Alignment | Reference during implementation |
| Clean Code Requirements | Import order, component structure |
| Phases | Logical progression of work (2-5 phases) |
| Risk Management | Identify and mitigate risks |
| Success Criteria | Measurable outcomes |
| Technical Reference | Code examples and implementation details |
| Completion Checklist | Final validation before marking complete |

### **Status Indicators**

| Indicator | Meaning |
|-----------|---------|
| 📋 PLANNING | Initial planning phase |
| 🚀 READY TO IMPLEMENT | Planning complete, ready to start |
| 🔄 IN PROGRESS | Active development |
| ✅ COMPLETED | All objectives achieved |
| ⚠️ CRITICAL | Urgent issues requiring attention |
| 🎯 OBJECTIVE | Goals and targets |
| 🔧 IMPLEMENTATION | Technical work sections |

### **Best Practices**

- **First principles thinking**: Focus on simplest path forward
- **Written planning first**: Explain approach before code examples
- **Consistency**: Follow patterns from existing action plans
- **No time estimates**: Focus on logical phases, not deadlines
- **Developer-friendly**: Structure for easy implementation
- **Never over-engineer**: Keep solutions simple and practical
- **Documentation is mandatory**: No PR without doc updates

---

*Last updated: Template v2.0 - Aligned with TravelOS Next.js/TypeScript standards*
