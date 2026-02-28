# Itinerary Items Data Mapping (Salesforce/Kaptio -> Supabase `itinerary_items`)

## Source Files
- Mapping source: `itinerary_items_field_mapping.xlsx` (`Itinerary Items Mapping` sheet)
- Upsert dataset: `itinerary_items_20260224_import.csv`
- Sync script: `scripts/upsert_itinerary_items.py`

## Field Mapping (Import CSV -> Supabase `itinerary_items`)

| Import Field | Supabase Field | Type | Required | Mapping Notes |
| --- | --- | --- | --- | --- |
| `external_id` | `external_id` | text | yes | Natural key for upsert conflict resolution |
| `itinerary_id` | `itinerary_id` | uuid | no | Use when pre-resolved |
| `itinerary_external_id` | `itinerary_id` (resolver input) | text -> uuid | no | Resolve from `itineraries.external_id` when UUID missing |
| `supplier_id` | `supplier_id` | uuid | no | Use when pre-resolved |
| `supplier_external_id` | `supplier_id` (resolver input) | text -> uuid | no | Resolve from `suppliers.external_id` when UUID missing |
| `item_name` | `item_name` | text | no | Service/item label |
| `description` | `item_description` | text | no | Detailed item description |
| `date_from` | `service_start_date` | date | no | Parse to ISO date |
| `date_to` | `service_end_date` | date | no | Parse to ISO date |
| `destination_country` | `location_country` | text | no | Country mapping |
| `(not in file)` | `location_region` | text | no | Leave null if unavailable |
| `location` | `location_city` | text | no | City/location mapping |
| `(not in file)` | `location_latitude` | numeric | no | Leave null if unavailable |
| `(not in file)` | `location_longitude` | numeric | no | Leave null if unavailable |
| `quantity` | `quantity` | integer | no | Numeric cast |
| `unit_cost` | `unit_cost` | numeric | no | Numeric cast |
| `total_cost` | `total_cost` | numeric | no | Numeric cast |
| `full_service_name` | `full_service_name` | text | no | Preserve full supplier/service label |
| `unit_price` | `unit_price` | numeric | no | Numeric cast |
| `total_price` | `total_price` | numeric | no | Numeric cast |
| `subtotal_price` | `subtotal_price` | numeric | no | Numeric cast |
| `subtotal_cost` | `subtotal_cost` | numeric | no | Numeric cast |
| `gross_margin` | `gross_margin` | numeric | no | Numeric cast |
| `profit_margin_percent` | `profit_margin_percent` | numeric | no | Numeric cast |
| `is_cancelled` | `is_cancelled` | boolean | no | Parse common true/false values |
| `cancelled_date` | `cancelled_date` | date | no | Parse to ISO date |
| `is_invoiced` | `is_invoiced` | boolean | no | Parse common true/false values |
| `is_deleted` | `is_deleted` | boolean | no | Parse common true/false values |
| `voucher_title` | `voucher_title` | text | no | Voucher/document label |
| `destination_continent` | `destination_continent` | text | no | Continent mapping when provided |
| `currency_code` | `currency_code` | text | no | ISO currency code |
| `voucher_reference` | `confirmation_number` | text | no | Confirmation/reference mapping |
| `confirmation_status` | `item_status` | text | no | Status mapping |
| `created_at` | `created_at` | timestamptz | no | Source created timestamp |
| `(not in file)` | `updated_at` | timestamptz | no | Null unless provided |
| `(not in file)` | `synced_at` | timestamptz | no | Sync-job timestamp when available |

## Remaining Excluded Source Fields Mapping

| Source Field | Mapping Status |
| --- | --- |
| _None_ | All currently requested source fields are mapped |

## Destination Booked Rollup Filter Rules

- Rollup source view: `mv_itinerary_destination_booked_monthly`.
- Time bucketing uses `itinerary_items.service_start_date` (month grain).
- Booked eligibility uses itinerary join plus status reference mapping:
  `itinerary_status_reference.is_filter_out = false` and
  (`itinerary_status_reference.pipeline_bucket = "closed_won"` OR `itineraries.itinerary_status = "Confirmed"`).
- Active booked metrics exclude item rows where `is_cancelled = true` or `is_deleted = true`.
- Cancellation/deletion counts are still retained in rollup quality fields for operational context.
