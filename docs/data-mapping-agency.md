# Agency Data Mapping (Salesforce Accounts -> Supabase `agencies`)

## Source Files
- Mapping source: `account_field_mapping.xlsx` (`Agencies Mapping` sheet)
- Upsert dataset: `agencies_upsert.csv`
- Sync script: `scripts/upsert_agencies.py`

## Field Mapping (Agencies)

| Salesforce Field | Supabase Field | Type | Required | Notes |
| --- | --- | --- | --- | --- |
| `Id` | `external_id` | text | yes | Natural key for upsert conflict resolution |
| `Name` | `agency_name` | text | no | Agency display name |
| `IATA_Number__c` | `agency_code` | text | no | IATA number stored as agency code |
| `Account_Email__c` | `contact_email` | text | no | Primary agency email |
| `KaptioTravel__IsActive__c` | `is_active` | boolean | no | Parsed from true/false-like values |
| `CreatedDate` | `created_at` | timestamptz | no | Source created timestamp |
| `LastModifiedDate` | `updated_at` | timestamptz | no | Source last-modified timestamp |
| `Consortia__c` | `consortia` | text | no | Consortia affiliation value |
