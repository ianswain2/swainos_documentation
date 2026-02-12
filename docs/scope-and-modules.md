SwainOS Scope and Modules

Scope
- Unified data layer with scheduled ETL from Salesforce, QuickBooks, FX, and manual inputs.
- One-way, read-only data sync from sources into SwainOS.
- Internal data model that supports historical retention and computed fields.
- Supabase-backed data layer with RLS and realtime updates.

Included application modules
- Command Center (dashboard)
- Cash Flow
- Debt Service
- Revenue and Bookings
- FX Command
- Operations
- AI Insights
- Settings and Administration

Data sources and integrations
- Salesforce (bookings, itineraries, contacts, suppliers)
- QuickBooks (transactions, P&L, AP/AR)
- FX rate provider (live and historical)
- News and macro data sources for sentiment inputs
- Manual inputs for budgets and debt schedules

Alignment note
- Build progress is tracked in `SwianOS_Documentation/action-plan/action-log`.
