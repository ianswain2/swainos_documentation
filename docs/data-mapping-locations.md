# Locations Data Mapping (Salesforce/Kaptio `KaptioTravel__Location__c` -> Supabase `locations`)

## Source Files
- Source object export: `location_export_20260401.csv`
- Upsert script: `SwainOS_BackEnd/scripts/upsert_locations.py`
- Lookup resolver script: `SwainOS_BackEnd/scripts/resolve_location_lookups.py`
- Ingestion orchestrator: `SwainOS_BackEnd/scripts/sync_salesforce_readonly.py`

## Captio Relationship Model (Location ERD)
- Canonical location object: `KaptioTravel__Location__c`.
- Hierarchy edge: self-lookup `KaptioTravel__Location__c` (parent location reference).
- Core usage from ERD:
  - `KaptioTravel__Item__c.KaptioTravel__Location__c` (supplier item location)
  - `KaptioTravel__Itinerary_Item__c` location lookups (pickup/dropoff/flight from/to)
  - Package and package-day location bridge objects
  - Itinerary location bridge object
- Practical consequence: location ingestion must preserve both identity and hierarchy (not only flattened city labels).

## Export Profile (2026-04-01)
- Rows: `2015`
- Core columns:
  - `Id` (Salesforce record ID)
  - `Name`
  - `KaptioTravel__FullLocationName__c`
  - `KaptioTravel__Location__c` (parent Salesforce ID)
  - `KaptioTravel__LocationType__c`
  - `SystemModstamp`
- Hierarchy shape from this export:
  - Root nodes include `Global`, continents/regions (e.g., `Oceania`, `Africa`, `Europe`)
  - Maximum observed depth: `4`
  - No missing parent references in export

## Supabase Location Contract
- Target table: `public.locations`
- Salesforce identity key: `locations.external_id` (maps from Salesforce `Id`)
- Parent edge fields:
  - `locations.parent_external_id` (raw Salesforce parent ID)
  - `locations.parent_location_id` (resolved FK to `locations.id`)
- Hierarchy metadata:
  - `locations.full_location_name`
  - `locations.hierarchy_depth`
  - `locations.location_type`
  - `locations.source_system`
  - `locations.source_updated_at`
  - `locations.is_deleted`

## Field Mapping (Import CSV -> Supabase `locations`)

| Import Field | Supabase Field | Type | Required | Mapping Notes |
| --- | --- | --- | --- | --- |
| `Id` / `external_id` | `external_id` | text | yes | Natural key for upsert conflict resolution |
| `KaptioTravel__Location__c` / `parent_external_id` | `parent_external_id` | text | no | Raw parent Salesforce ID |
| `(resolver)` | `parent_location_id` | uuid | no | Resolved from `parent_external_id` -> `locations.external_id` |
| `Name` / `location_name` | `city_name` | text | no | Canonical location label for current filtering/search display |
| `KaptioTravel__FullLocationName__c` / `full_location_name` | `full_location_name` | text | no | Fallback to `Name` when missing |
| `KaptioTravel__LocationType__c` / `location_type` | `location_type` | text | no | Source location classification |
| `(derived from parent chain)` | `hierarchy_depth` | integer | no | Root = 0, increments by parent level |
| `SystemModstamp` / `source_updated_at` | `source_updated_at` | timestamptz | no | Source incremental watermark |
| `(constant)` | `source_system` | text | no | Defaults to `salesforce_kaptio` |
| `IsDeleted` / `is_deleted` | `is_deleted` | boolean | no | Soft-delete indicator |
| `(not in export)` | `country_code` | text | no | Null unless separately enriched |
| `(not in export)` | `latitude` / `longitude` / `timezone` | numeric/text | no | Null unless separately enriched |
| `(not in export)` | `is_primary_destination` | boolean | no | Existing default retained |

## Lookup Resolver Contract
- Parent-location resolver:
  - `locations.parent_external_id` -> `locations.external_id` -> `locations.parent_location_id`
- Supplier-item resolver:
  - `supplier_items.location_external_id` -> `locations.external_id` -> `supplier_items.location_id`
- Resolver write safety:
  - Merge-upsert by `id` only (`on_conflict=id`)
  - Payload limited to `id` + lookup FK being set

## Strict Integrity Guardrails
- `upsert_locations.py` supports strict reconciliation enforcement via `--strict-reconciliation-check`.
- In strict mode, the script fails when legacy `locations` rows still exist with `external_id is null`.
- No heuristic country/region derivation is applied during upsert.

## Incremental Sync Contract
- Include `locations` in ingestion object scope before supplier item resolver stage.
- Default source object: `KaptioTravel__Location__c`.
- Minimum required fields for sync extraction:
  - `Id`
  - `Name`
  - `KaptioTravel__Location__c`
  - `KaptioTravel__FullLocationName__c`
  - `KaptioTravel__LocationType__c`
  - `IsDeleted`
  - `SystemModstamp`
- Stage order for location linkage:
  1. `locations` upsert
  2. `supplier_items` upsert
  3. `location_lookup_resolver`

## Cross-Database Alignment Notes
- `supplier_items` is the primary SwainOS table currently carrying direct location lineage from Salesforce item records.
- `itinerary_items` still carries denormalized location fields (`location_city`, `location_country`) and can be incrementally aligned to `locations` in a later phase if needed.
- Supplier analytics materialized views already join through `supplier_items.location_id -> locations.id`; this location resolver path directly improves those joins.

## Reconciliation SQL (Post-Import)
```sql
-- Location identity + hierarchy coverage.
select
  count(*) as locations_total,
  count(*) filter (where external_id is not null) as locations_with_external_id,
  count(*) filter (where parent_external_id is not null) as locations_with_parent_external,
  count(*) filter (where parent_external_id is not null and parent_location_id is not null) as locations_with_parent_fk
from public.locations;

-- Supplier-item location lookup coverage.
select
  count(*) as supplier_items_total,
  count(*) filter (where location_external_id is not null) as supplier_items_with_location_external,
  count(*) filter (where location_id is not null) as supplier_items_with_location_fk,
  count(*) filter (where location_external_id is not null and location_id is null) as unresolved_supplier_item_locations
from public.supplier_items;
```
