# Itinerary Data Mapping (Salesforce/Kaptio -> Supabase)

This document is the canonical mapping reference for itinerary ingestion and forecasting rollups.
It is intentionally ETL-focused and will be extended table-by-table as new mappings are introduced.

## Source Files
- `itineraries_supabase_import.csv` (primary import dataset)
- Kaptio/Salesforce itinerary fields (historical source naming retained in mapping below)

## Itinerary Status Mapping

| Status Value | Pipeline Category | Notes |
| --- | --- | --- |
| Lost | Closed - Lost | Lost opportunity |
| Traveled | Closed - Won | Trip completed |
| Cancelled | Closed - Lost | Cancelled before travel |
| Duplicate Itinerary | Filter Out | Exclude from analytics |
| Test Itinerary | Filter Out | Exclude from analytics |
| Rejected | Lost | Lost outcome |
| Sample Itinerary | Filter Out | Exclude from analytics |
| Pre-Departure | Closed - Won | Confirmed and preparing |
| Proposal Sent | Open | Pipeline potential |
| Cancel Fees | Closed - Won | Closed with cancellation fee outcome |
| Deposited/Confirming | Closed - Won | Deposit received; confirming |
| Amendment in Progress | Closed - Won | Amendment actively in progress and retained in closed-won allowlist |
| Holding | Holding | On hold |
| Traveling | Closed - Won | In-travel |
| eDocs Sent | Closed - Won | Travel docs sent |
| Amendment Merged | Closed - Lost | Amendment merged into prior opportunity and treated as lost for analytics |
| Snapshot Booking | Filter Out | Exclude from analytics |
| Amendment Rejected | Closed - Lost | Amendment lost |
| Assigned | Open | Being worked by consultant |
| Invoiced | Closed - Lost | Not in the closed-won allowlist; reclassified to closed-lost |
| Booked | Closed - Lost | Not in the closed-won allowlist; reclassified to closed-lost |
| Confirmed | Closed - Lost | Not in the closed-won allowlist; reclassified to closed-lost |
| Closed | Closed - Lost | Closed/lost record class |
| Draft | Open | Early pipeline |
| Pending | Open | Awaiting action |

## Field Mapping (Itineraries)

| Salesforce / Kaptio Field | Supabase Field | ETL Notes |
| --- | --- | --- |
| `Id` | `external_id` | Required natural key for upsert conflict resolution |
| `KaptioTravel__BookingNumber__c` | `itinerary_number` | Booking/reference number |
| `(optional)` | `itinerary_name` | Use when source provides title/name |
| `KaptioTravel__Status__c` | `itinerary_status` | Normalize via status reference table |
| `KaptioTravel__Start_Date__c` | `travel_start_date` | Parse to `date` |
| `KaptioTravel__End_Date__c` | `travel_end_date` | Parse to `date` |
| `Itinerary_Countries__c` | `primary_country` | String field; may need normalization later |
| `(optional)` | `primary_region` | Nullable |
| `(optional)` | `primary_city` | Nullable |
| `(optional)` | `primary_latitude` | Numeric |
| `(optional)` | `primary_longitude` | Numeric |
| `KaptioTravel__Group_Size__c` | `pax_count` | Integer |
| `(optional)` | `adult_count` | Integer |
| `(optional)` | `child_count` | Integer |
| `KaptioTravel__Itinerary_Amount__c` | `gross_amount` | Numeric |
| `KaptioTravel__TotalAmountNet__c` | `net_amount` | Numeric |
| `KaptioTravel__CommissionTotal__c` | `commission_amount` | Numeric |
| `KaptioTravel__DepositAmount__c` | `deposit_received` | Numeric (fallback to total deposit paid) |
| `(optional)` | `balance_due` | Numeric |
| `CurrencyIsoCode` | `currency_code` | ISO code |
| `(optional)` | `agency_id` | UUID FK to `agencies` when pre-resolved |
| `KaptioTravel__Account__c` | `agency_external_id` | Preserve source agency reference for later FK resolution |
| `(optional)` | `primary_contact_id` | UUID FK to `contacts` |
| `KaptioTravel__Primary_Contact__c` | `primary_contact_external_id` | Preserve source contact reference for later FK resolution |
| `(optional)` | `primary_contact_type` | Direct/Trade classification |
| `CloseDateOutput__c` | `close_date` | Date used for close/booking trend analysis |
| `Commission_Due_Date__c` | `trade_commission_due_date` | Date |
| `Commission_Status__c` | `trade_commission_status` | Text enum-like values |
| `Consortia__c` | `consortia` | Consortia identifier/value |
| `KaptioTravel__FinalPaymentExpectedDate__c` | `final_payment_date` | Date |
| `KaptioTravel__GrossProfit__c` | `gross_profit` | Numeric |
| `KaptioTravel__Itinerary_Cost__c` | `cost_amount` | Numeric |
| `KaptioTravel__No_of_days__c` | `number_of_days` | Integer |
| `KaptioTravel__No_of_nights__c` | `number_of_nights` | Integer |
| `KaptioTravel__ResellerCommissionTotal__c` | `trade_commission_amount` | Numeric |
| `KaptioTravel__Outstanding__c` | `outstanding_balance` | Numeric |
| `OwnerId` | `owner_external_id` | Salesforce user owner id |
| `Lost_Date__c` | `lost_date` | Date |
| `Lost_Reason_Description__c` | `lost_comments` | Text |
| `CreatedDate` | `created_at` | Parse to timestamp |
| `LastModifiedDate` | `updated_at` | Parse to timestamp |
| `(optional)` | `synced_at` | Sync job timestamp |

## Forecasting Semantics
- Revenue recognition should use `travel_end_date` (not close date).
- Conversion/close trend should use `close_date`.
- Deposit expectations assume baseline `25%` of gross at confirmation, tracked against `deposit_received`.
- Passenger planning uses `pax_count` (plus adult/child when available).
- Reseller commission (`trade_commission_amount`) is amount paid to travel agents.
- Cost (`cost_amount`) represents supplier-side obligations where populated.
- Canonical owner-cockpit Gross Profit metric uses itinerary `gross_profit` (rollup field `gross_profit_amount`).
- Source `net_amount` remains available as source-system net itinerary amount but is not the primary owner-cockpit income metric.

## FK Resolver Notes (Import Script)
- `scripts/upsert_itineraries.py` resolves:
  - `agency_external_id` -> `agency_id` via `agencies.external_id`
  - `primary_contact_external_id` -> `primary_contact_id` via `contacts.external_id`
- `scripts/upsert_itineraries.py` also resolves:
  - `owner_external_id` -> `employee_id` via `employees.external_id`
- Resolver runs by default and prints resolved/unresolved counts.
- Use `--skip-fk-resolver` only for troubleshooting or partial-load scenarios.

## Travel Consultant Analytics Context
- Consultant attribution is single-owner per itinerary via `itineraries.employee_id`.
- Travel outcome analytics (booked revenue, margin, production) are grouped by `travel_end_date`.
- Funnel analytics (lead count, conversion, speed to book) are grouped by `created_at` and close lifecycle dates.
- Profile Hero KPI averages are derived from itinerary-level fields and period rollups:
  - `averageGrossProfit` -> itinerary `gross_profit` (via rollup `gross_profit_amount`)
  - `averageItineraryNights` -> weighted `avg_number_of_nights`
  - `averageGroupSize` -> `pax_count / itinerary_count`
  - `averageLeadTime` -> days from `created_at` to `travel_start_date` (closed-won itineraries)
  - `averageSpeedToClose` -> days from `created_at` to `close_date` (closed-won itineraries)
- UI speed-to-book is surfaced as **average speed to book** (`avgSpeedToBookDays`) derived from funnel monthly cohorts.
- Frontend consultant surfaces consume materialized rollups driven by this owner mapping:
  - `mv_travel_consultant_leaderboard_monthly`
  - `mv_travel_consultant_profile_monthly`
  - `mv_travel_consultant_funnel_monthly`
  - `mv_travel_consultant_compensation_monthly`

## Rollup Foundations Added
- `mv_itinerary_revenue_monthly`: monthly recognized financials by `travel_end_date` and pipeline category.
- `mv_itinerary_revenue_weekly`: weekly recognized financials by `travel_end_date`.
- `mv_itinerary_deposit_monthly`: monthly close-date deposit performance vs 25% target.
- `mv_itinerary_consortia_monthly`: monthly consortia breakdown by travel start date.
- `mv_itinerary_trade_agency_monthly`: monthly trade agency breakdown (trade/agent contact types + agency fallback resolution).
- `mv_itinerary_consortia_actuals_monthly`: closed-won consortia production by `travel_end_date`.
- `mv_itinerary_trade_agency_actuals_monthly`: closed-won trade-agency production by `travel_end_date`.
- `itinerary_status_reference`: canonical status classification used by rollups.

## Next Tables To Map (Suggested Priority)
1. `contacts` (for trade/direct segmentation and agency attribution)
2. `bookings` (service-level and supplier performance forecast drivers)
3. `customer_payments` (cash-in pacing and deposit conversion health)
4. `supplier_invoices` / `supplier_invoice_lines` (cash-out forecasting and FX exposure)
