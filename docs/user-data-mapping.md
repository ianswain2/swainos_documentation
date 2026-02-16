# User Data Mapping (Employees / Travel Consultants)

This document is the canonical mapping reference for Travel Consultant identity and compensation data used by analytics and profile experiences.

## Source System and Sync Scope
- Source system: Salesforce users (consultant owner records).
- Sync target: Supabase `public.employees`.
- Sync mechanism: `scripts/upsert_employees.py` (CSV/import driven).
- Attribution link: `itineraries.owner_external_id` -> `employees.external_id` -> `itineraries.employee_id`.

## Field Mapping (Employees)

| Salesforce / Source Field | Supabase Field | Type | Required | Notes |
| --- | --- | --- | --- | --- |
| `Id` | `external_id` | text | yes | Natural key for upsert conflict resolution |
| `FirstName` | `first_name` | text | yes | Consultant first name |
| `LastName` | `last_name` | text | yes | Consultant last name |
| `Email` | `email` | text | yes | Unique; used for duplicate handling in import |
| `AnnualSalary` (or equivalent source) | `salary` | numeric | no | Stored as annual salary amount |
| `(defaulted in system)` | `commission_rate` | numeric | yes | Defaults to `0.15` (15% of margin) |
| `(system generated)` | `id` | uuid | yes | Primary key for internal joins |
| `(system generated)` | `created_at` | timestamptz | yes | Insert timestamp |
| `(system generated)` | `updated_at` | timestamptz | yes | Trigger-managed update timestamp |

## Owner Resolution Semantics
- Each itinerary has one owner for consultant attribution.
- Import resolves `owner_external_id` to `employees.id` and writes `itineraries.employee_id`.
- If owner resolution fails, itinerary keeps unresolved owner state unless strict-fail mode is used (`--fail-on-unresolved-employees`).
- Consultant analytics rollups only include itineraries that can be joined to an employee record.

## Compensation Defaults and Calculations
- Commission default: `employees.commission_rate = 0.15`.
- Salary input: annual salary (`employees.salary`).
- Period allocation (monthly view): `salary / 12`.
- Estimated commission: `commission_rate * margin_amount`.
- Estimated total pay: `salary_period_amount + estimated_commission_amount`.

## Rollup Dependencies
- `mv_travel_consultant_leaderboard_monthly`: rankings, conversion/close context, target variance.
- `mv_travel_consultant_profile_monthly`: travel-outcome profile totals by period.
- `mv_travel_consultant_funnel_monthly`: lead-to-book funnel metrics and speed-to-book cohort values (UI displays average speed-to-book).
- `mv_travel_consultant_compensation_monthly`: salary + commission + estimated total pay.

## Advisor Effectiveness Inputs (Profile UI)
- Effectiveness summary blends:
  - YTD revenue variance vs prior-year baseline
  - Conversion rate
  - Margin %
  - Avg speed to book
- Hero KPI averages use:
  - average gross profit per itinerary
  - average itinerary nights
  - average group size
  - average lead time (`created_at` -> `travel_start_date`)
  - average speed to close (`created_at` -> `close_date`)

## Frontend Contract Dependencies
- `GET /api/v1/travel-consultants/leaderboard`
- `GET /api/v1/travel-consultants/{employee_id}/profile`
- `GET /api/v1/travel-consultants/{employee_id}/forecast`

These endpoints drive `apps/web/src/app/travel-consultant/page.tsx` and `apps/web/src/app/travel-consultant/[employeeId]/page.tsx`.
