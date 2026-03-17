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
| Supplier Liability | Outstanding supplier AP from payable line rollups. | Currency | `totalOutstandingAmount`, `due30dAmount` | Supplier Payables, Open Supplier Liability |
| Deposit Liability | Outstanding customer receivable/deposit posture in current window. | Currency | `outstandingDeposits`, `availableCashAfterLiability` | Open Deposit Liability |
| Net Liquidity (AR/AP) | Net liquidity after combining cash window with customer receivable and supplier AP liabilities. | Currency | Derived UI metric (`netCashTotal + outstandingDeposits - totalOutstandingAmount`) | Cash Position (ambiguous) |
| Deposit Coverage % | Deposits received divided by deposits targeted/required. | Percent | Derived from deposit timeline summary | Deposit Health (generic) |
| Cash Risk Status | Risk classification for upcoming cash posture by currency. | Enum (`healthy`, `watch`, `at_risk`) | `riskStatus` | Cash Health Flag |
| First Risk Date | Earliest date where projected cash violates risk rule by currency. | Date | `firstRiskDate` | First Breach Date |
| Cash Buffer Threshold | Operating cash reserve threshold used in risk scoring. | Currency | `cashBufferAmount` | Reserve Floor |
| Coverage Ratio | Projected inflows divided by projected outflows in selected horizon. | Ratio (`x`) | `coverageRatio` | Inflow/Outflow Ratio |

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

## Marketing Web Analytics Canonical Terms

| Canonical Display Term | Definition | Preferred Format | Canonical API Field(s) | Deprecated/Synonym Terms |
|---|---|---|---|---|
| Web Analytics Overview | Strategic top-level web performance view for traffic, engagement, and conversion trends. | Section title | `/marketing/web-analytics/overview` payload | Website Summary |
| Source Tracking | Source/medium and referral performance surface for acquisition quality and value ranking decisions. | Section title | `/marketing/web-analytics/search` payload | Search Performance |
| Search Console Insights | Search-demand and organic-performance workspace powered by canonical Search Console snapshots, with overview, opportunities, challenges, query/page rankings, and diagnostics. | Section title | `/marketing/web-analytics/search-console` payload | SEO Dashboard (generic) |
| Search Console Page Profile | Dedicated URL drill-down profile for one page path with query, benchmark, and recommendation context. | Section title | `/marketing/web-analytics/search-console/page-profile` payload | URL Detail (generic) |
| AI Website Insights | Structured action-engine recommendations generated from web analytics signals for marketer/sales actioning. | Section title | `/marketing/web-analytics/ai-insights` payload | AI SEO Tips |
| Tracking Health | Integration and freshness health payload used for operator diagnostics; not currently surfaced as a primary Marketing tab. | Backend diagnostic concept | `/marketing/web-analytics/health` payload | Data Health (generic) |
| Page Activity | Page-level behavior surface for identifying best/worst pages, itinerary diagnostics, and lookbook/destination activity. | Section title | `/marketing/web-analytics/page-activity` payload | Content Performance (generic) |
| Geography & Events | Combined geo segmentation, audience/device breakdown, and event-definition analytics surface. | Section title | `/marketing/web-analytics/geo` + `/marketing/web-analytics/events` payloads | Geo Dashboard + Event Logs |
| Sessions | Total website sessions in selected analysis window. | Integer | `sessions` | Visits |
| Users | Distinct users in selected analysis window. | Integer | `totalUsers` | Visitors |
| Engagement Rate | Share of sessions considered engaged by GA4 rules. | Percent | `engagementRate` | Engagement |
| Key Events | Count of strategically relevant conversion/intent events. | Integer | `keyEvents` | Conversions (generic) |
| Key Event Rate | Share of sessions that generated key events; used for page quality ranking. | Percent | `keyEventRate` | Conversion Rate (ambiguous) |
| Landing Page | First page in a session used for entry-point performance analysis. | Text | `landingPage` | Entry Page |
| Page Path | Canonical URL path used for per-page behavior diagnostics. | Text | `pagePath` | URL (ambiguous) |
| Quality Score | Composite score blending key event rate, engagement, and traffic scale for ranking pages. | Percent-like score | `qualityScore` | Page Score (generic) |
| Itinerary Page | Any page path matching itinerary intent (`itinerary` in path). | Boolean marker | `isItineraryPage` | Trip Page (informal) |
| Lookbook Page | Page path classified as lookbook-related using deterministic path contains rules (`lookbook`, `/about/lookbooks`). | Boolean/path classification | `lookbookPages[]`, `pagePath` | Catalog Page (generic) |
| Destination Page | Page path classified as destination-related using deterministic path contains rules (`destination`, `destinations`). | Boolean/path classification | `destinationPages[]`, `pagePath` | Locale Page (generic) |
| Internal Site Search Term | On-site user query term captured from GA4 `view_search_results` event context. | Text + integer counts | `internalSiteSearchTerms[].searchTerm` | Query (ambiguous) |
| Source Value Score | Composite source quality score blending key-event rate, qualified-session rate, bounce profile, and scaled session volume. | Decimal score | `sourceMix[].valueScore`, `topValuableSources[].valueScore` | Source Rank |
| Qualified Session Rate | Share of sessions from a source that meet engaged-session criteria and represent higher-intent traffic. | Percent | `sourceMix[].qualifiedSessionRate` | Engaged Session Share |
| Bounce Rate | Share of sessions from a source that bounce, used as a traffic quality risk signal. | Percent | `sourceMix[].bounceRate` | Exit Rate (ambiguous) |
| Source Quality Label | Classification of source quality (`qualified`, `mixed`, `poor`) based on qualification, bounce, and conversion signals. | Enum | `sourceMix[].qualityLabel` | Source Tier |
| Device Category | GA4 device segmentation bucket (mobile/desktop/tablet) for UX prioritization. | Text | `devices[].deviceCategory` | Device Type (informal) |
| Age Bracket | GA4 audience age cohort bucket used for demographic targeting. | Text | `demographics[].ageBracket` | Age Group (generic) |
| Focus Area | Primary action posture assigned by the marketing action engine (`scale`, `fix`, `cut`, `instrument`, `localize`, `optimize`). | Enum | `focusArea` | Action Type (generic) |
| Impact Score | Relative expected business impact of an action recommendation on a 0-100 scale. | Integer-like score | `impactScore` | Priority Score |
| Confidence Score | Relative confidence in a recommendation based on signal strength and consistency on a 0-100 scale. | Integer-like score | `confidenceScore` | Confidence (generic) |
| Target Label | Main entity the action recommendation is aimed at (page, channel, market, device, or demand topic). | Text | `targetLabel` | Target (generic) |
| Channel Group | Default acquisition channel grouping from GA4 attribution. | Text | `channelName` (`sessionDefaultChannelGroup`) | Traffic Source (ambiguous) |
| Canonical Daily Fact | Marketing snapshot row whose primary key matches the surfaced business grain (for example `snapshotDate + channelName` or `snapshotDate + country`). | Data-model concept | Marketing analytics snapshot tables | Derived Rollup Row |
| Overview Period Summary | Precomputed KPI window summary used by Overview (`current_30d`, `previous_30d`, `year_ago_30d`, `today`, `yesterday`) to avoid distinct-user overcounting from day-level sums. | Data-model concept | `marketing_web_analytics_overview_period_summaries` | On-read Window Math |
| Market Scope | Explicit market filter context applied to Marketing analytics responses (`all` or country name, e.g. `United States`). | Text + label pair | `meta.marketScope`, `meta.marketLabel`, `country` query param | Region Filter (ambiguous) |
| Opportunity Type | Canonical Search Console opportunity category shown in table rows. | Enum | `opportunityType` (`low_ctr`, `near_breakout`, `page_refresh`, `destination_gap`) | Opportunity Class |
| Challenge Type | Canonical Search Console challenge category shown in table rows. | Enum | `challengeType` (`page_ctr_gap`, `ranking_drop`, `coverage_gap`, `intent_mismatch`) | Challenge Class |
| Rank Band | Position grouping used to make query visibility easier to scan in Search Console surfaces. | Enum-like label | `positionBand`, `positionBandSummary[].bandLabel` | Position Bucket |

## Data Jobs Control Plane Canonical Terms

| Canonical Display Term | Definition | Preferred Format | Canonical API Field(s) | Deprecated/Synonym Terms |
|---|---|---|---|---|
| Last Run | Most recent run timestamp shown in job administration views. | Date-time | `lastStartedAt` (health), `requestedAt` fallback in UI mutation state | Last Executed |
| Run Status | Latest execution state for a job (`success`, `failed`, `blocked`, etc.). | Enum pill | `lastRunStatus` (health), `runStatus` (run records) | Job Health (ambiguous) |
| Retry Backoff (Minutes) | Minimum cooldown interval after a failed recurring run before scheduler re-dispatch. | Integer minutes | `retryBackoffMinutes` | Retry Delay |
| Scheduler Tick | Canonical scheduler dispatch trigger that selects due recurring jobs. | Operational trigger label | `POST /api/v1/data-jobs/scheduler/tick`, run `triggerType=scheduler` | Cron Runner |
| Schedule Cadence | Human-readable representation of recurring cron schedule. | Text label | Derived from `scheduleCron` and `scheduleMode` | Raw Cron (as primary label) |
| Run Logs | Cross-job run stream surface for operators to review execution outcomes and failure reasons with deep pagination history. | Section title | `/settings/run-logs`, `/api/v1/data-jobs/run-feed` | Job Stream |
| Run Duration | Persisted elapsed runtime per job run for historical analysis. | Seconds | `durationSeconds` (`duration_seconds` in DB) | Runtime |
| Run Output Size | Persisted serialized output payload size per run for trend analysis and cost signal monitoring. | Bytes | `outputSizeBytes` (`output_size_bytes` in DB) | Payload Size |

## Supplier Invoice Canonical Terms

| Canonical Display Term | Definition | Preferred Format | Canonical API Field(s) | Deprecated/Synonym Terms |
|---|---|---|---|---|
| Supplier Invoice | Parent payable record from supplier billing feed. | Entity label | `supplierInvoices[]`, `supplierInvoiceId`, `supplierInvoiceExternalId` | Vendor Invoice |
| Supplier Invoice Booking | Booking-junction parent record used by supplier invoice lines (`a29` object). | Entity label | `supplierInvoiceBookings[]`, `supplierInvoiceBookingId`, `supplierInvoiceBookingExternalId` | Supplier Invoice Booking Junction, Invoice Booking Link |
| Supplier Invoice Line | Child line item attached to a supplier invoice parent. | Entity label | `supplierInvoiceLines[]`, `supplierInvoiceLineId`, `supplierInvoiceExternalId` | Vendor Invoice Line, AP Line |
| Supplier Invoice External ID | Source-system external parent key used for strict resolver linkage. | Text key | `supplierInvoiceExternalId` | Invoice SFID (ambiguous) |
| Supplier Invoice Header External ID | Header/payables source key for `supplier_invoices` (`a2B` object). | Text key | `supplierInvoices.externalId` | Invoice Header SFID |
| Supplier Invoice Booking External ID | Booking-junction source key used for line-parent resolution (`a29` object). | Text key | `supplierInvoiceBookingExternalId`, `supplierInvoiceLines.supplierInvoiceExternalId` | Supplier Invoice Booking SFID |
| Strict FK Resolver | Import mode that derives FK UUIDs from external IDs and ignores incoming FK UUID columns. | Operational mode | `--strict-fk-resolver` in import scripts | Auto-link mode |
| Unresolved FK Export | Reconciliation artifact listing unresolved external IDs after strict resolution pass. | CSV output | `--export-unresolved-csv` output | Error export |

## Salesforce Sync Canonical Terms

| Canonical Display Term | Definition | Preferred Format | Canonical API Field(s) | Deprecated/Synonym Terms |
|---|---|---|---|---|
| System Modstamp Cursor | Incremental extraction cursor from Salesforce record mutation timestamp. | UTC datetime | `SystemModstamp`, `lastSystemmodstamp` (stored cursor) | Last Sync Date (ambiguous) |
| Window Start | Lower bound used for current run extraction after overlap policy. | UTC datetime | `windowStart`, `salesforce_sync_runs.window_start` | Start Date (generic) |
| Upper Bound | Single run-level extraction ceiling (`now - settle lag`) shared across objects. | UTC datetime | `upperBound`, `salesforce_sync_cursors.last_completed_upper_bound` | Pull End Date |
| Overlap Window | Intentional replay window to absorb late-arriving writes between runs. | Minutes | `overlap_minutes` (runtime config), documented default `60` | Duplicate Pull Buffer |
| Settle Lag | Safety delay before extraction upper bound to avoid in-flight transaction churn. | Minutes | `--upper-bound-lag-minutes` (default `5`) | Ingestion Delay (generic) |
| API Limits Preflight | Run-start check that blocks extraction when org API usage is over threshold. | Percent threshold + usage snapshot | `/limits` -> `DailyApiRequests`, `apiLimits.dailyApiRequests.usedPercent` | Limit Check |
| Blocked Sync Run | Non-failed operational stop caused by deterministic guardrail (for example API budget threshold or active lock). | Run status | `blocked`, `blockedReason` | Soft Failure |
| Object Metrics | Per-object ingest summary captured for each sync run. | Structured JSON object | `objectMetrics.*` (`extracted`, `staged`, `loaded`, `skippedUnresolved`, `csv_bytes`) | Script Stats |
| Bulk API Counters | Salesforce Bulk API request activity counters for cost/behavior visibility. | Integer counters | `jobsCreated`, `pollsMade`, `resultPagesRead` | API Ping Count |
| Unresolved Reference Export | CSV artifact of unresolved strict foreign-key lookups for replay/backfill workflows. | CSV file path + row count | `--export-unresolved-csv`, unresolved rows in script metrics | FK Error Dump |
| Soft Delete Mapping | Policy that maps source deletions without hard-deleting warehouse records. | Boolean flag semantics | Salesforce `IsDeleted` -> destination `isDeleted` | Delete Mirror |

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
