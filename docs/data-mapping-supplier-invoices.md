# Supplier Invoices + Lines Data Mapping (Salesforce/Kaptio -> Supabase)

## Source Files
- Mapping source: `supplier_invoice_field_glossary.xlsx`, `supplier_invoice_lines_field_mapping.xlsx`
- Upsert datasets: `supplier_invoices_import.csv`, `supplier_invoice_bookings_import.csv`, `supplier_invoice_lines_import.csv`
- Sync scripts: `scripts/upsert_supplier_invoices.py`, `scripts/upsert_supplier_invoice_bookings.py`, `scripts/upsert_supplier_invoice_lines.py`

## Parent/Child Model
- Header/payables table: `public.supplier_invoices` (`external_id` conflict target, Salesforce `a2B`).
- Line parent table: `public.supplier_invoice_bookings` (`external_id` conflict target, Salesforce `a29`).
- Child table: `public.supplier_invoice_lines` (`external_id` conflict target).
- Parent linkage: `supplier_invoice_lines.supplier_invoice_booking_id` resolved from
  `supplier_invoice_external_id` -> `supplier_invoice_bookings.external_id`.
- Required integrity rule: each line must include at least one recognized parent key
  (booking-parent preferred; legacy invoice-parent fields retained for transition compatibility).

## Field Mapping (Import CSV -> Supabase `supplier_invoices`)

| Import Field | Supabase Field | Type | Required | Mapping Notes |
| --- | --- | --- | --- | --- |
| `external_id` | `external_id` | text | yes | Natural key for upsert conflict resolution |
| `invoice_name` | `invoice_name` | text | no | Canonical source invoice label |
| `invoice_number` | `invoice_number` | text | no | Fallback to `invoice_name` if missing |
| `supplier_external_id` | `supplier_id` (resolver input) | text -> uuid | no | Resolve from `suppliers.external_id` |
| `supplier_id` | `supplier_id` | uuid | no | Used when already pre-resolved |
| `amount` | `amount` | numeric | no | Numeric cast |
| `total_amount` | `total_amount` | numeric | no | Numeric cast |
| `commission_amount` | `commission_amount` | numeric | no | Numeric cast |
| `commission_amount_rollup` | `commission_amount_rollup` | numeric | no | Numeric cast |
| `tax_total` | `tax_total` | numeric | no | Numeric cast |
| `invoice_type` | `invoice_type` | text | no | Source type value |
| `invoice_date` | `invoice_date` | date | no | Parse to ISO date |
| `currency_code` | `currency_code` | text | no | ISO currency code |
| `brand_external_id` | `brand_external_id` | text | no | External brand reference |
| `external_code` | `external_code` | text | no | Source external code |
| `has_email_confirmation` | `has_email_confirmation` | boolean | no | Parse common true/false values |
| `sent_to_external_date` | `sent_to_external_date` | date | no | Parse to ISO date |
| `sent_to_external_system` | `sent_to_external_system` | boolean | no | Parse common true/false values |
| `supplier_sequence` | `supplier_sequence` | text | no | Source sequence marker |
| `is_deleted` | `is_deleted` | boolean | no | Soft-delete indicator |
| `created_at` | `created_at` | timestamptz | no | Source created timestamp |
| `updated_at` | `updated_at` | timestamptz | no | Source updated timestamp |

## Field Mapping (Import CSV -> Supabase `supplier_invoice_lines`)

| Import Field | Supabase Field | Type | Required | Mapping Notes |
| --- | --- | --- | --- | --- |
| `external_id` | `external_id` | text | yes | Natural key for upsert conflict resolution |
| `supplier_invoice_external_id` | `supplier_invoice_booking_id` (resolver input) | text -> uuid | yes | Resolve from `supplier_invoice_bookings.external_id` |
| `supplier_invoice_booking_external_id` | `supplier_invoice_booking_external_id` | text | no | Hydrated from `supplier_invoice_external_id` for explicit parent lineage |
| `supplier_invoice_booking_id` | `supplier_invoice_booking_id` | uuid | no | Parent booking UUID |
| `supplier_invoice_id` | `supplier_invoice_id` | uuid | no | Legacy invoice-parent UUID retained for compatibility only |
| `itinerary_item_external_id` | `itinerary_item_id` (resolver input) | text -> uuid | no | Resolve from `itinerary_items.external_id` |
| `itinerary_item_id` | `itinerary_item_id` | uuid | no | Use when pre-resolved |
| `itinerary_external_id` | `itinerary_id` (resolver input) | text -> uuid | no | Resolve from `itineraries.external_id` |
| `itinerary_id` | `itinerary_id` | uuid | no | Use when pre-resolved |
| `supplier_external_id` | `supplier_id` (resolver input) | text -> uuid | no | Resolve from `suppliers.external_id` |
| `supplier_id` | `supplier_id` | uuid | no | Use when pre-resolved |
| `line_name` | `line_name` | text | no | Line label |
| `item_name` | `item_name` | text | no | Product/service label |
| `line_description` | `line_description` | text | no | Detailed description |
| `booking_date` | `booking_date` | date | no | Parse to ISO date |
| `due_date` | `due_date` | date | no | Parse to ISO date |
| `balance_due` | `balance_due` | numeric | no | Numeric cast |
| `balance` | `balance` | numeric | no | Numeric cast |
| `cost_due` | `cost_due` | numeric | no | Numeric cast |
| `cost_invoiced` | `cost_invoiced` | numeric | no | Numeric cast |
| `original_cost_due` | `original_cost_due` | numeric | no | Numeric cast |
| `commission_due` | `commission_due` | numeric | no | Numeric cast |
| `commission_invoiced` | `commission_invoiced` | numeric | no | Numeric cast |
| `commission_balance_due` | `commission_balance_due` | numeric | no | Numeric cast |
| `currency_code` | `currency_code` | text | no | ISO currency code |
| `payment_rule_type` | `payment_rule_type` | text | no | Source payment-rule type |
| `payment_rule_key` | `payment_rule_key` | text | no | Source payment-rule key |
| `adjustment_type` | `adjustment_type` | text | no | Source adjustment marker |
| `custom_type` | `custom_type` | text | no | Source custom type |
| `is_cancelled` | `is_cancelled` | boolean | no | Parse common true/false values |
| `is_complete` | `is_complete` | boolean | no | Parse common true/false values |
| `is_credit_line` | `is_credit_line` | boolean | no | Parse common true/false values |
| `is_manual_adjustment` | `is_manual_adjustment` | boolean | no | Parse common true/false values |
| `skip_processing` | `skip_processing` | boolean | no | Parse common true/false values |
| `comment` | `comment` | text | no | Operational notes |
| `is_deleted` | `is_deleted` | boolean | no | Soft-delete indicator |
| `created_at` | `created_at` | timestamptz | no | Source created timestamp |
| `updated_at` | `updated_at` | timestamptz | no | Source updated timestamp |
| `synced_at` | `synced_at` | timestamptz | no | Source sync timestamp |

## Field Mapping (Import CSV -> Supabase `supplier_invoice_bookings`)

| Import Field | Supabase Field | Type | Required | Mapping Notes |
| --- | --- | --- | --- | --- |
| `external_id` | `external_id` | text | yes | Natural key for upsert conflict resolution (Salesforce `a29`) |
| `booking_name` | `booking_name` | text | no | Source booking label |
| `balance_due` | `balance_due` | numeric | no | Numeric cast |
| `commission_balance_due` | `commission_balance_due` | numeric | no | Numeric cast |
| `commission_due` | `commission_due` | numeric | no | Numeric cast |
| `commission_invoiced` | `commission_invoiced` | numeric | no | Numeric cast |
| `cost_due` | `cost_due` | numeric | no | Numeric cast |
| `cost_invoiced` | `cost_invoiced` | numeric | no | Numeric cast |
| `currency_code` | `currency_code` | text | no | ISO currency code |
| `is_complete` | `is_complete` | boolean | no | Parse common true/false values |
| `is_deleted` | `is_deleted` | boolean | no | Soft-delete indicator |
| `created_at` | `created_at` | timestamptz | no | Source created timestamp |
| `updated_at` | `updated_at` | timestamptz | no | Source updated timestamp |
| `synced_at` | `synced_at` | timestamptz | no | Source sync timestamp |
| `itinerary_external_id` | `itinerary_id` (resolver input) | text -> uuid | no | Resolve from `itineraries.external_id` |
| `supplier_external_id` | `supplier_id` (resolver input) | text -> uuid | no | Resolve from `suppliers.external_id` |
| `itinerary_id` | `itinerary_id` | uuid | no | Use when pre-resolved |
| `supplier_id` | `supplier_id` | uuid | no | Use when pre-resolved |

## Resolver + Upsert Rules
- Resolver execution happens in scripts only (no Supabase resolver functions).
- In strict mode (`--strict-fk-resolver`), scripts always derive FK UUIDs from external IDs.
- In skip mode (`--skip-unresolved-fks`), rows with unresolved strict FK targets are skipped and counted.
- Optional unresolved export (`--export-unresolved-csv`) writes unresolved entity external IDs for reconciliation.
- `--start-row` and `--max-rows` support resumable/staged loads.

## Operational Import Sequence
1. Run `supplier_invoices` import first with strict resolver mode.
2. Run `supplier_invoice_bookings` import with strict resolver mode.
3. Confirm booking-parent (`a29`) resolution metrics and unresolved output.
4. Run `supplier_invoice_lines` import with strict resolver mode.
5. Reconcile unresolved parent/related external IDs, then re-run staged batches as needed.

## Current-State Resolver Policy (Static Itinerary Data Lag)
- `supplier_invoice_lines` parent booking linkage is treated as required and must resolve (`a29` -> `supplier_invoice_bookings.external_id`).
- `itinerary_item_external_id` and `itinerary_external_id` are currently tolerated as optional unresolved references in strict mode when source invoice data lands ahead of itinerary sync.
- Keep unresolved optional references exported and tracked for follow-up backfill after itinerary refreshes.

## Reconciliation SQL (Post-Import)
```sql
-- Core row counts.
select count(*) as supplier_invoices_total from public.supplier_invoices;
select count(*) as supplier_invoice_bookings_total from public.supplier_invoice_bookings;
select count(*) as supplier_invoice_lines_total from public.supplier_invoice_lines;

-- Hard integrity checks for booking parent linkage on lines.
select count(*) as lines_missing_all_parent_keys
from public.supplier_invoice_lines
where supplier_invoice_booking_id is null
  and supplier_invoice_booking_external_id is null
  and supplier_invoice_external_id is null
  and supplier_invoice_id is null;

select count(*) as lines_with_unresolved_booking_external
from public.supplier_invoice_lines
where supplier_invoice_booking_id is null
  and (
    supplier_invoice_booking_external_id is not null
    or supplier_invoice_external_id is not null
  );

-- Optional lagging references to clean up after itinerary refresh.
select count(*) as lines_with_unresolved_itinerary_item
from public.supplier_invoice_lines
where itinerary_item_external_id is not null
  and itinerary_item_id is null;

select count(*) as lines_with_unresolved_itinerary
from public.supplier_invoice_lines
where itinerary_external_id is not null
  and itinerary_id is null;

-- Constraint validation status check.
select conname, convalidated
from pg_constraint
where conname in (
  'supplier_invoice_bookings_itinerary_id_fkey',
  'supplier_invoice_bookings_supplier_id_fkey',
  'supplier_invoice_lines_itinerary_item_id_fkey',
  'supplier_invoice_lines_supplier_id_fkey',
  'supplier_invoice_lines_supplier_invoice_booking_id_fkey',
  'supplier_invoice_lines_parent_key_presence_check'
)
order by conname;
```
