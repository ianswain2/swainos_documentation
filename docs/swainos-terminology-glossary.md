# SwainOS Terminology Glossary

## Purpose
This glossary is the canonical source of truth for user-facing terminology across SwainOS frontend UI, backend documentation, and AI-generated narrative copy.

## Scope
- Page titles and section headers
- KPI and metric labels
- Filter and toggle labels
- Empty-state and helper copy when metric terms are referenced
- Backend field-to-display terminology mapping

## Core Rules
- Use one canonical display name per concept across all modules.
- API contract keys should align to canonical terminology.
- Prefer explicit labels that distinguish amount vs percent/rate.
- When backend key names differ from display names, document mapping here and in backend/frontend code documentation.

## Breaking Contract Targets
- This glossary defines breaking canonical targets for API and code symbols.
- Legacy naming variants are not used in active contracts or active code.
- Canonical financial metric family:
  - `grossAmount` (revenue)
  - `grossProfitAmount` (profit)
  - `marginAmount` (amount)
  - `marginPct` (ratio)

## Canonical Metrics

| Canonical Display Term | Definition | Preferred Format | Canonical API Field(s) | Deprecated/Synonym Terms |
|---|---|---|---|---|
| Gross Revenue | Total gross itinerary value in scope. | Currency | `grossAmount`, `bookedRevenue`, `expectedGrossAmount`, `forecastGrossAmount`, `targetGrossAmount` | Gross (ambiguous) |
| Gross Profit | Profit from itinerary activity. | Currency | `grossProfitAmount`, `expectedGrossProfitAmount`, `forecastGrossProfitAmount`, `targetGrossProfitAmount` | Commission Income, Income, Net |
| Booked Itineraries | Count of closed-won itineraries attributed to the selected travel period window. | Integer | `bookedItinerariesCount` | Traveled Itineraries, Traveled Files |
| Margin Amount | Absolute margin amount in currency. | Currency | `marginAmount`, `expectedMarginAmount` | Margin (when `%` is not explicit) |
| Margin % | Margin ratio relative to gross revenue. | Percent | `marginPct`, `expectedMarginPct` | Margin Ratio, Margin Percent |
| Booked Revenue | Closed-won revenue for selected period in consultant and leaderboard contexts. | Currency | `bookedRevenue` | Revenue (generic) |
| Booked Item Value | Total booked destination item value in scope, aligned to itinerary-item service start period. | Currency | `bookedTotalPrice` | Item Value, Destination Revenue |
| Destination Country | Country rollup dimension for destination analytics. | Text | `country`, `locationCountry` | Country (when entity context is unclear) |
| Destination City | City rollup dimension nested under destination country analytics. | Text | `city`, `locationCity` | City (when entity context is unclear) |
| Conversion Rate | Closed won divided by total leads. | Percent | `conversionRate` | Conversion (when not clearly rate) |
| Close Rate | Closed won divided by (closed won + closed lost). | Percent | `closeRate` | Win Rate (unless explicitly same formula) |
| Speed to Close (Days) | Average days from lead creation to booking close. | Days (`d`) | `avgSpeedToBookDays` | Avg Speed to Book, Lead Time |
| YoY Variance % | Year-over-year percent change vs comparable prior period. | Signed Percent | `yoyToDateVariancePct`, `ytdVariancePct`, derived YoY delta fields | YoY to-date (without %) |
| Target Variance % | Percent variance against strategic target trajectory. | Signed Percent | `growthTargetVariancePct`, `growthGapPct`, `totalGrowthGapPct` | Growth Gap (without %) |
| Supplier Liability | Outstanding supplier invoices/payables. | Currency | `outstandingAmount` | Supplier Payables, Open Supplier Liability |
| Deposit Liability | Outstanding customer deposit obligations. | Currency | `outstandingDeposits` | Open Deposit Liability |
| Deposit Coverage % | Deposits received divided by deposits targeted/required. | Percent | Derived from deposit timeline summary | Deposit Health (generic) |

## Debt Service Canonical Terms

| Canonical Display Term | Definition | Preferred Format | Canonical API Field(s) | Deprecated/Synonym Terms |
|---|---|---|---|---|
| Debt by Creditor | Facility-level breakdown of lender obligations and due timing. | Table section title | `facilities[]` in debt overview + `lenderName` in facilities endpoint | Loan Breakdown (ambiguous) |
| Outstanding Balance | Current unpaid principal balance for facility or aggregate debt view. | Currency | `outstandingBalanceAmount` | Remaining Debt (generic) |
| Next Due Date | Next contractual due date for a facility (null for standby debt). | Date | `nextDueDate` | Next Payment Date (facility-row context) |
| Next Due Amount | Next contractual payment amount for a facility (null for standby debt). | Currency | `nextDueAmount` | Next Payment (facility-row context) |
| Seller Note 1 | Seller-financed note with 2-year standby period and then repayment schedule. | Facility label | `facilityName` | Seller Note (ambiguous single-note label) |
| Seller Note 2 (Equity Injection) | Seller note counted toward SBA equity injection with full standby through SBA life. | Facility label | `facilityName` | Seller Equity Note (inconsistent variants) |
| Standby / N/A | Display state for obligations with no current due date/amount under standby terms. | Text state | `nextDueDate = null`, `nextDueAmount = null` | No Payment Due (inconsistent variants) |

## FX Canonical Terms

| Canonical Display Term | Definition | Preferred Format | Canonical API Field(s) | Deprecated/Synonym Terms |
|---|---|---|---|---|
| Funding Currency | Currency used to fund payable-currency buys in FX workflows. | ISO currency code | `baseCurrency` (`USD` in v1 policy) | Base Buy Currency (ambiguous) |
| Payable Currency | Non-USD currency used to settle supplier obligations. | ISO currency code | `currencyCode` (target set `AUD`, `NZD`, `ZAR`) | Tracked Currency (ambiguous) |
| FX Signal | Recommendation output for buy timing decisions. | Enum + rationale | `signalType`, `signalStrength`, `reasonSummary` | FX Alert (generic) |
| Invoice Pressure (30/60/90d) | Supplier due-date weighted payable amount pressure by currency window. | Currency | `invoicePressure30d`, `invoicePressure60d`, `invoicePressure90d` | Invoice Urgency Score |
| Data Health | Operator-visible data quality status for recommendation safety. | Enum | `meta.dataStatus` (`live`, `partial`, `degraded`) | Health Flag (generic) |
| Source Links | Clickable provenance links for macro/news evidence behind intelligence overlays. | URL list | `sourceLinks` | References (generic) |

## Canonical UI Title and Control Patterns

| Surface Type | Canonical Pattern | Example |
|---|---|---|
| Page title | Title Case, concise domain name | `Travel Consultant` |
| Section header | Title Case, noun-oriented | `Recommendation Queue` |
| Filter label | Singular noun or noun phrase | `Domain`, `Severity`, `Status` |
| Metric toggle label | Canonical metric names only | `Gross Profit`, `Gross Revenue`, `Margin Amount`, `PAX` |
| Time window toggle | Short window token or explicit period name | `3m`, `6m`, `12m`, `This Year` |
| Empty-state copy | Plain, deterministic, no metric synonym drift | `No recommendations matched current filters.` |

## Specific Mapping Guidance

### Gross Profit
- User-facing UI should display **Gross Profit**.
- API and code symbol target is `grossProfitAmount`.
- Data lineage remains sourced from itinerary `gross_profit`.

### Margin Naming
- Use **Margin Amount** for currency values.
- Use **Margin %** for ratios.
- Avoid standalone `Margin` where amount vs percent is unclear.

### Conversion Naming
- Use **Conversion Rate** and **Close Rate** as separate terms.
- Always keep formulas consistent with canonical definitions.

## Governance

### Change Control
- Any new cross-module metric/filter/title term must be added here before rollout.
- PRs that introduce new terminology should include:
  - proposed term
  - definition
  - API/data mapping
  - impacted surfaces

### Review Checklist
- Does the same concept use the same name everywhere?
- Is amount vs percent explicit?
- Is backend field naming mapped clearly when different from display text?
- Are AI-generated summaries using canonical display terms?

## Linked Standards
- `docs/swainos-code-documentation-frontend.md`
- `docs/swainos-code-documentation-backend.md`
- `docs/frontend-data-queries.md`
- `docs/sample-payloads.md`

## Rename Ledger

| Legacy Term | Canonical Term | Scope |
|---|---|---|
| `commissionIncomeAmount` | `grossProfitAmount` | API payloads, frontend types, backend schemas/services |
| `onBooksCommissionIncomeAmount` | `onBooksGrossProfitAmount` | Itinerary outlook contracts/UI |
| `potentialCommissionIncomeAmount` | `potentialGrossProfitAmount` | Itinerary outlook contracts/UI |
| `expectedCommissionIncomeAmount` | `expectedGrossProfitAmount` | Itinerary outlook contracts/UI |
| `forecastCommissionIncomeAmount` | `forecastGrossProfitAmount` | Itinerary outlook contracts/UI |
| `targetCommissionIncomeAmount` | `targetGrossProfitAmount` | Itinerary outlook contracts/UI |
| `totalExpectedCommissionIncomeAmount` | `totalExpectedGrossProfitAmount` | Itinerary outlook summary |
| `totalForecastCommissionIncomeAmount` | `totalForecastGrossProfitAmount` | Itinerary outlook summary |
| `totalTargetCommissionIncomeAmount` | `totalTargetGrossProfitAmount` | Itinerary outlook summary |
| `projectedCommissionIncomeExpected` | `projectedGrossProfitExpected` | Conversion contracts/UI |
| `projectedCommissionIncomeBestCase` | `projectedGrossProfitBestCase` | Conversion contracts/UI |
| `projectedCommissionIncomeWorstCase` | `projectedGrossProfitWorstCase` | Conversion contracts/UI |
| `commissionIncome` | `grossProfit` | Travel consultant contracts/UI |
| `commission_income_amount` | `gross_profit_amount` | Backend schema/repository/service symbols and response fields |
| `avgCommissionIncomePerItinerary` | `avgGrossProfitPerItinerary` | Itinerary actuals contracts/UI |
| `avgCommissionIncomePerPax` | `avgGrossProfitPerPax` | Itinerary actuals contracts/UI |
| `directCommissionIncomeAmount` | `directGrossProfitAmount` | Trade vs direct contracts/UI |
| `tradeCommissionIncomeAmount` | `tradeGrossProfitAmount` | Trade vs direct contracts/UI |
