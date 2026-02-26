# User Data Mapping (Employees / Travel Consultants)

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
