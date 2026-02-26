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
| `currency_code` | `currency_code` | text | no | ISO currency code |
| `voucher_reference` | `confirmation_number` | text | no | Confirmation/reference mapping |
| `confirmation_status` | `item_status` | text | no | Status mapping |
| `created_at` | `created_at` | timestamptz | no | Source created timestamp |
| `(not in file)` | `updated_at` | timestamptz | no | Null unless provided |
| `(not in file)` | `synced_at` | timestamptz | no | Sync-job timestamp when available |

## Excluded Source Fields Mapping

| Source Field | Mapping Status |
| --- | --- |
| `full_service_name` | Not mapped |
| `unit_price` | Not mapped |
| `total_price` | Not mapped |
| `subtotal_price` | Not mapped |
| `subtotal_cost` | Not mapped |
| `gross_margin` | Not mapped |
| `profit_margin_percent` | Not mapped |
| `is_cancelled` | Not mapped |
| `cancelled_date` | Not mapped |
| `is_invoiced` | Not mapped |
| `is_deleted` | Not mapped |
| `voucher_title` | Not mapped |
| `destination_continent` | Not mapped |
