# Supplier Data Mapping (Salesforce Accounts -> Supabase `suppliers`)

## Source Files
- Mapping source: `account_field_mapping.xlsx` (`Suppliers Mapping` sheet)
- Upsert dataset: `suppliers_upsert.csv`
- Sync script: `scripts/upsert_suppliers.py`

## Field Mapping (Suppliers)

| Salesforce Field | Supabase Field | Type | Required | Notes |
| --- | --- | --- | --- | --- |
| `Id` | `external_id` | text | yes | Natural key for upsert conflict resolution |
| `Name` | `supplier_name` | text | no | Supplier display name |
| `IATA_Number__c` | `supplier_code` | text | no | IATA number stored as supplier code |
| `(not in export)` | `supplier_type` | text | no | Left blank in current source file |
| `KaptioTravel__AccountCurrency__c` | `default_currency` | text | no | Default supplier currency |
| `(not in export)` | `payment_terms_days` | integer | no | Left blank in current source file |
| `Account_Email__c` | `contact_email` | text | no | Primary supplier email |
| `Phone` | `contact_phone` | text | no | Primary supplier phone |
| `(not in export)` | `address_country` | text | no | Left blank in current source file |
| `KaptioTravel__IsActive__c` | `is_active` | boolean | no | Parsed from true/false-like values |
| `CreatedDate` | `created_at` | timestamptz | no | Source created timestamp |
| `LastModifiedDate` | `updated_at` | timestamptz | no | Source last-modified timestamp |
