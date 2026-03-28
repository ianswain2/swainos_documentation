# Item Data Mapping (Salesforce/Kaptio `KaptioTravel__Item__c` -> Supabase `supplier_items`)

## Source Files
- Field glossary: `item_field_glossary.xlsx` (`Field Glossary`, `Field Analysis`, `Record Types`)
- Seed import dataset: `items_supabase_import.csv`
- Seed sync script: `SwainOS_BackEnd/scripts/upsert_supplier_items.py`

## Canonical Object and Naming
- Source object API name: `KaptioTravel__Item__c`
- Kaptio UI label: **Service** (same physical object)
- SwainOS canonical warehouse/table naming for this object: **Supplier Item** (`supplier_items`)

## Parent/Child Link Model
- Parent supplier identity: `supplier_items.supplier_id` (resolved from `supplier_external_id` -> `suppliers.external_id`)
- Itinerary instance linkage: `itinerary_items.supplier_item_id` and `itinerary_items.supplier_item_external_id`
- Location enrichment path (phase 1): item-level fields on `supplier_items` (`location_name`, `location_external_id`, lat/long)
- Optional canonical location FK: `supplier_items.location_id` -> `locations.id` (nullable until location resolver path is finalized)

## Field Mapping (Seed CSV -> Supabase `supplier_items`)

| Seed Import Field | Supabase Field | Type | Required | Mapping Notes |
| --- | --- | --- | --- | --- |
| `external_id` | `external_id` | text | yes | Natural key for upsert conflict resolution (`KaptioTravel__Item__c.Id`) |
| `supplier_external_id` | `supplier_id` (resolver input) | text -> uuid | no | Resolve from `suppliers.external_id` in strict resolver mode |
| `supplier_id` | `supplier_id` | uuid | no | Use when pre-resolved |
| `supplier_external_id` | `supplier_external_id` | text | no | Preserve source supplier reference |
| `item_name` | `item_name` | text | no | Item/service display name |
| `item_type` | `item_type` | text | no | Item record type/category (`Activity`, `Accommodation`, `Transfer`, etc.) |
| `external_name` | `external_name` | text | no | External-facing item name |
| `short_description` | `short_description` | text | no | Short summary text |
| `item_code` | `item_code` | text | no | Item code/reference |
| `currency_code` | `currency_code` | text | no | ISO currency code |
| `star_rating` | `star_rating` | numeric | no | Star/property rating when available |
| `is_active` | `is_active` | boolean | no | Parsed from `1/0`, `true/false`, etc. |
| `is_multiday` | `is_multiday` | boolean | no | Multiday indicator |
| `allocation_type` | `allocation_type` | text | no | Allocation behavior (e.g., booking/allotment/night) |
| `default_start_time` | `default_start_time` | text | no | Default service/check-in start time |
| `default_end_time` | `default_end_time` | text | no | Default service/check-out end time |
| `datetime_visibility` | `datetime_visibility` | text | no | Date/time display behavior |
| `time_setup` | `time_setup` | text | no | Time setup mode |
| `transfer_type` | `transfer_type` | text | no | Transfer type classification |
| `unit_of_measure` | `unit_of_measure` | text | no | Unit basis (`each day`, etc.) |
| `fallback_status` | `fallback_status` | text | no | Inventory fallback status |
| `inventory_setup` | `inventory_setup` | text | no | Inventory setup policy |
| `show_in_content` | `show_in_content` | boolean | no | Content visibility flag |
| `show_in_vouchers` | `show_in_vouchers` | boolean | no | Voucher visibility flag |
| `cancellation_policy` | `cancellation_policy` | text | no | Cancellation policy body text |
| `child_policy` | `child_policy` | text | no | Child policy body text |
| `commission_group` | `commission_group` | text | no | Commission group label |
| `address` | `address` | text | no | Physical/service address text |
| `service_address` | `service_address` | text | no | Service-specific address variant |
| `service_phone` | `service_phone` | text | no | Service-specific phone |
| `evoucher_address` | `evoucher_address` | text | no | E-voucher address text |
| `evoucher_phone` | `evoucher_phone` | text | no | E-voucher phone |
| `voucher_message` | `voucher_message` | text | no | Voucher message |
| `web_description` | `web_description` | text | no | Web description body |
| `location_name` | `location_name` | text | no | Kaptio item location label/name |
| `location_latitude` | `location_latitude` | numeric | no | Item-level location latitude |
| `location_longitude` | `location_longitude` | numeric | no | Item-level location longitude |
| `location_external_id` | `location_external_id` | text | no | Source location object external ID |
| `location_id` | `location_id` | uuid | no | Optional pre-resolved FK to `locations.id` |
| `contract_last_updated` | `contract_last_updated` | date | no | Contract maintenance date |
| `is_deleted` / `IsDeleted` | `is_deleted` | boolean | no | Soft-delete indicator from source |
| `created_at` | `created_at` | timestamptz | no | Source created timestamp/date |
| `updated_at` | `updated_at` | timestamptz | no | Source updated timestamp/date |
| `(optional)` | `synced_at` | timestamptz | no | Sync execution timestamp |

## Itinerary Item Lookup Extension (Phase 1)
- Add `itinerary_items.supplier_item_external_id` to store the source lookup value from itinerary-item exports (`KaptioTravel__Item__c` / `item_external_id` / `supplier_item_external_id`).
- Add `itinerary_items.supplier_item_id` as UUID FK resolved from `supplier_items.external_id`.
- This preserves source lineage while enabling stable relational joins.

## Seed Import Procedure
1. Apply migration: `0126_create_supplier_items_and_itinerary_item_lookup.sql`.
2. Import supplier items seed CSV (strict resolver):
   - `python3 scripts/upsert_supplier_items.py "/Users/ianswain/Documents/Swain Destinations/Acquisition Documents/Data Analysis/Data Exports/items_supabase_import.csv" --strict-fk-resolver --skip-unresolved-fks --export-unresolved-csv ./tmp/unresolved_supplier_item_suppliers.csv`
3. Re-run itinerary items import with supplier-item lookup support:
   - `python3 scripts/upsert_itinerary_items.py "<path-to-itinerary-items-csv>" --strict-fk-resolver --skip-unresolved-fks --export-unresolved-csv ./tmp/unresolved_itinerary_item_refs.csv`

## Incremental Sync Contract (Phase 2)
- Salesforce read-only sync now treats supplier items as a first-class object (`KaptioTravel__Item__c`) in the scheduled incremental window (`SystemModstamp` + `Id` cursor semantics).
- Default extraction is intentionally minimal/schema-safe (no optional field toggles): `Id`, `KaptioTravel__Supplier__c`, `Name`, `CurrencyIsoCode`, `IsDeleted`, `CreatedDate`, `LastModifiedDate`, `SystemModstamp`.
- Required itinerary-item lookup field in each incremental pull:
  - `KaptioTravel__Itinerary_Item__c.KaptioTravel__Item__c` -> `itinerary_items.supplier_item_external_id`
- Pipeline load order for supplier-item linkage:
  1. `suppliers`
  2. `supplier_items`
  3. `itineraries`
  4. `itinerary_items`
  5. post-load resolver (`itinerary_items.supplier_item_external_id` -> `supplier_items.external_id` -> `itinerary_items.supplier_item_id`) only when the current incremental run extracted itinerary-item rows
- Resolver write safety contract:
  - merge-upsert by `id` only (`on_conflict=id`)
  - payload limited to `id` + `supplier_item_id`
  - no other itinerary-item columns are modified during resolver execution
- Per-run resolver diagnostics emitted in sync metrics:
  - `with_external`
  - `with_fk`
  - `unresolved`
  - `unresolved_pct`
- Historical reconciliation is intentionally separated from the incremental hot path:
  - DB-native function: `backfill_itinerary_item_supplier_links_v1()`
  - data job: `itinerary-item-supplier-links-backfill`

## Post-Import Linkage SQL (ready to run)
```sql
-- Run DB-native backfill function (no itinerary-item re-export required).
-- Note: execute as service-role context.
select public.backfill_itinerary_item_supplier_links_v1();

-- Coverage diagnostics.
select
  count(*) as itinerary_items_with_supplier_item_external_id,
  count(*) filter (where supplier_item_id is not null) as itinerary_items_linked_supplier_item_id,
  count(*) filter (where supplier_item_external_id is not null and supplier_item_id is null) as unresolved_supplier_item_links
from public.itinerary_items;

-- Supplier-item row counts and location coverage.
select
  count(*) as supplier_items_total,
  count(*) filter (where supplier_id is not null) as supplier_items_with_supplier_fk,
  count(*) filter (where location_external_id is not null) as supplier_items_with_location_external_id
from public.supplier_items;
```

## Relationship Coverage Notes (Phase 1)
- Primary analytics linkage path:
  `itinerary_items` -> `supplier_items` -> `suppliers`
- Existing payable line path remains unchanged:
  `supplier_invoice_lines` -> `itinerary_items` (via `itinerary_item_id`) -> `supplier_items`
- No direct payable-line FK to `supplier_items` is required in phase 1.
- `supplier_items.location_id` remains optional in phase 1. Until location-object resolver wiring is added, keep `location_external_id` + `location_name` as canonical item-location fields and track null `location_id` coverage in diagnostics.
