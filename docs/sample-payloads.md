# Sample Payloads

Purpose: canonical request/response examples for active frontend/backend contracts.

## Conventions
- Response envelope: `{ data, pagination, meta }` for **FastAPI** (`/api/v1/...`) success responses.
- Same-origin **Next.js App Router** handlers under `/api/...` (not `/api/v1`) may return a slimmer `{ data: … }` success body or `{ error: { code, message, … } }` on failure—see `POST /api/auth/login` below. These are not Python backend routes.
- Query params: `snake_case`
- JSON fields: `camelCase`
- Dates: ISO 8601
- **Dashboard snapshots** return composite `data` objects (see below); individual domain shapes match their standalone endpoints.

## Error Envelope

```json
{
  "error": {
    "code": "bad_request",
    "message": "Unsupported time window format",
    "details": {
      "requestId": "f9f2f6ca-1d6b-49a7-9848-a0c7a5c2d322"
    }
  }
}
```

## Health

### `GET /health/ready`

```json
{
  "status": "ok",
  "service": "swainos-backend",
  "checks": {
    "supabase": "ok"
  }
}
```

## Authentication and Access Control

### `POST /api/auth/login` (Next.js — same origin as the web app)

Request (password sign-in):

```json
{
  "email": "member@example.com",
  "password": "••••••••"
}
```

When **`NEXT_PUBLIC_TURNSTILE_SITE_KEY`** is configured on the Next.js app, the route **requires** a Cloudflare Turnstile token (same contract as Supabase password sign-in CAPTCHA):

```json
{
  "email": "member@example.com",
  "password": "••••••••",
  "captchaToken": "0.xxx-turnstile-response-token"
}
```

Success (`200`): session cookies are set on the response; body is intentionally minimal:

```json
{
  "data": {
    "success": true
  }
}
```

Failure examples (JSON body only; status matches `4xx`/`5xx`):

```json
{
  "error": {
    "code": "invalid_request",
    "message": "Email and password are required."
  }
}
```

```json
{
  "error": {
    "code": "invalid_credentials",
    "message": "Invalid email or password."
  }
}
```

```json
{
  "error": {
    "code": "too_many_attempts",
    "message": "Too many sign in attempts. Please try again shortly.",
    "retryAfterSeconds": 60
  }
}
```

`429` responses may also include a `Retry-After` header (seconds) aligned with `retryAfterSeconds`.

```json
{
  "error": {
    "code": "invalid_captcha",
    "message": "Complete the security verification challenge."
  }
}
```

Returned when Turnstile is required (site key set) but `captchaToken` is missing/empty, or when Supabase rejects the CAPTCHA/Turnstile verification (still surfaced as `invalid_captcha` with generic copy—no raw provider strings to callers).

```json
{
  "error": {
    "code": "service_unavailable",
    "message": "Sign in is temporarily unavailable."
  }
}
```

### `GET /api/v1/auth/me`

```json
{
  "data": {
    "userId": "4d87e6c9-8cae-4f35-b7b4-9d94a2c0b210",
    "email": "ianswain2@gmail.com",
    "role": "admin",
    "isAdmin": true,
    "isActive": true,
    "permissionKeys": [
      "ai_insights",
      "cash_flow",
      "command_center",
      "debt_service",
      "destination",
      "fx_command",
      "itinerary_actuals",
      "itinerary_forecast",
      "marketing_web_analytics",
      "operations",
      "search_console_insights",
      "settings_job_controls",
      "settings_run_logs",
      "settings_user_access",
      "travel_agencies",
      "travel_consultant"
    ],
    "canManageAccess": true
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-10",
    "source": "user_access_summary_v1",
    "timeWindow": "",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### `GET /api/v1/settings/user-access`

```json
{
  "data": [
    {
      "userId": "4d87e6c9-8cae-4f35-b7b4-9d94a2c0b210",
      "email": "ianswain2@gmail.com",
      "role": "admin",
      "isActive": true,
      "permissionKeys": [
        "command_center",
        "settings_user_access"
      ],
      "createdAt": "2026-03-10T20:11:05.218Z",
      "updatedAt": "2026-03-10T21:44:16.604Z"
    },
    {
      "userId": "a9f87f95-4609-4d2f-8c68-f9979fb3fe6f",
      "email": "member@example.com",
      "role": "member",
      "isActive": true,
      "permissionKeys": [],
      "createdAt": "2026-03-10T21:41:04.993Z",
      "updatedAt": "2026-03-10T21:41:04.993Z"
    }
  ],
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-10",
    "source": "user_access_summary_v1",
    "timeWindow": "",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### `PUT /api/v1/settings/user-access/{user_id}`

Request:

```json
{
  "role": "member",
  "isActive": true,
  "permissionKeys": ["command_center", "marketing_web_analytics"],
  "reason": "access_update_by_admin"
}
```

Response:

```json
{
  "data": {
    "userId": "a9f87f95-4609-4d2f-8c68-f9979fb3fe6f",
    "email": "member@example.com",
    "role": "member",
    "isActive": true,
    "permissionKeys": ["command_center", "marketing_web_analytics"],
    "createdAt": "2026-03-10T21:41:04.993Z",
    "updatedAt": "2026-03-10T21:47:41.428Z"
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-10",
    "source": "user_access_summary_v1",
    "timeWindow": "",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

## Dashboard snapshots

### `GET /api/v1/dashboard-snapshots/command-center`

Abbreviated: nested values mirror standalone service shapes (camelCase in JSON). Partial failures populate `sectionErrors` and may set `warning` / `error` inside `data`.

```json
{
  "data": {
    "cashFlowSummaries": [
      {
        "currencyCode": "USD",
        "cashInTotal": 1200000.0,
        "cashOutTotal": 980000.0,
        "netCashTotal": 220000.0
      }
    ],
    "depositsSummaries": [],
    "paymentsSummaries": [],
    "forecast": [],
    "itineraryLeadFlow": { "timeline": [] },
    "itineraryOutlook": { "summary": {}, "timeline": [], "closeRatio": 0.0 },
    "itineraryActuals": { "years": [], "timeline": [], "yearSummaries": [] },
    "debtOverview": { "asOfDate": "2026-03-14", "facilityCount": 0 },
    "briefing": null,
    "warning": null,
    "error": null,
    "sectionErrors": {}
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-14",
    "source": "command_center_snapshot_v1",
    "timeWindow": "30d+12m",
    "calculationVersion": "v1",
    "currency": null,
    "dataStatus": "live",
    "isStale": false,
    "degraded": false
  }
}
```

### `GET /api/v1/dashboard-snapshots/cash-flow`

```json
{
  "data": {
    "riskOverview": [],
    "forecast3m": [],
    "forecast12m": [],
    "scenarios": []
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-14",
    "source": "cash_flow_snapshot_v1",
    "timeWindow": "12m",
    "calculationVersion": "v1",
    "currency": null,
    "dataStatus": "live",
    "isStale": false,
    "degraded": false
  }
}
```

## Liquidity (AR/AP)

### `GET /api/v1/payments-out/summary?time_window=90d`

```json
{
  "data": [
    {
      "currencyCode": "AUD",
      "openLineCount": 321,
      "totalOutstandingAmount": 418922.44,
      "due30dAmount": 210305.0,
      "nextDueDate": "2026-03-04"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 50,
    "totalItems": 1,
    "totalPages": 1
  },
  "meta": {
    "asOfDate": "2026-03-02",
    "source": "ap_open_liability_v1",
    "timeWindow": "90d",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### `GET /api/v1/ap/aging?time_window=90d`

```json
{
  "data": [
    {
      "currencyCode": "AUD",
      "openLineCount": 321,
      "totalOutstandingAmount": 418922.44,
      "currentNotDueAmount": 180010.0,
      "overdue130Amount": 145512.44,
      "overdue3160Amount": 55100.0,
      "overdue6190Amount": 20200.0,
      "overdue90PlusAmount": 18100.0
    }
  ],
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-02",
    "source": "ap_aging_v1",
    "timeWindow": "90d",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### `GET /api/v1/ap/payment-calendar?time_window=90d`

```json
{
  "data": [
    {
      "paymentDate": "2026-03-04",
      "currencyCode": "AUD",
      "lineCount": 42,
      "supplierCount": 8,
      "amountDue": 56200.0
    }
  ],
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-02",
    "source": "ap_payment_calendar_v1",
    "timeWindow": "90d",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### `GET /api/v1/deposits/summary?time_window=90d`

```json
{
  "data": [
    {
      "currencyCode": "USD",
      "totalDeposits": 1200000.0,
      "receivedDeposits": 945000.0,
      "outstandingDeposits": 255000.0,
      "availableCashAfterLiability": 712000.0
    }
  ],
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-02",
    "source": "salesforce_kaptio",
    "timeWindow": "90d",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### `GET /api/v1/cash-flow/risk-overview?time_window=12m`

```json
{
  "data": [
    {
      "currencyCode": "USD",
      "riskStatus": "watch",
      "firstRiskDate": "2026-07-01",
      "timeToRiskDays": 121,
      "projectedEndingCash": 742500.0,
      "projectedMinCash": 184000.0,
      "cashBufferAmount": 225000.0,
      "coverageRatio": 1.12,
      "riskDrivers": [
        {
          "code": "buffer_breach",
          "message": "Projected cash drops below the operating buffer threshold."
        }
      ]
    }
  ],
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-02",
    "source": "customer_payments + ap_payment_calendar_v1",
    "timeWindow": "12m",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### `GET /api/v1/cash-flow/forecast?time_window=12m`

```json
{
  "data": [
    {
      "currencyCode": "USD",
      "timeWindow": "12m",
      "points": [
        {
          "periodStart": "2026-03-01",
          "periodEnd": "2026-03-31",
          "cashIn": 812000.0,
          "cashOut": 641500.0,
          "netCash": 170500.0,
          "projectedEndingCash": 170500.0,
          "coverageRatio": 1.27,
          "atRisk": false
        }
      ]
    }
  ],
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-02",
    "source": "customer_payments + ap_payment_calendar_v1",
    "timeWindow": "12m",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### `GET /api/v1/cash-flow/ap-schedule?time_window=12m`

```json
{
  "data": [
    {
      "paymentDate": "2026-03-04",
      "currencyCode": "AUD",
      "amountDue": 56200.0,
      "lineCount": 42,
      "supplierCount": 8
    }
  ],
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-02",
    "source": "ap_payment_calendar_v1",
    "timeWindow": "12m",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### `GET /api/v1/cash-flow/scenarios?time_window=12m`

```json
{
  "data": [
    {
      "scenarioName": "Delay 10% of near-term AP by 30 days",
      "currencyCode": "USD",
      "description": "Read-only simulation showing a timing relief case for a portion of upcoming AP obligations.",
      "projectedEndingCash": 906250.0,
      "firstRiskDate": "2026-07-01",
      "riskStatus": "healthy"
    }
  ],
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-02",
    "source": "synthetic_scenarios_v1",
    "timeWindow": "12m",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

## Debt Service

### `GET /api/v1/debt-service/overview`

```json
{
  "data": {
    "asOfDate": "2026-09-01",
    "facilityCount": 3,
    "outstandingBalanceAmount": 6780029,
    "nextPaymentDate": "2026-09-01",
    "nextPaymentAmount": 49927,
    "principalPaidYtdAmount": 19971,
    "interestPaidYtdAmount": 79883,
    "scheduledDebtService30dAmount": 49927,
    "scheduledDebtService60dAmount": 99854,
    "scheduledDebtService90dAmount": 149781,
    "dscrValue": null,
    "covenantStatus": "in_compliance",
    "facilities": [
      {
        "facilityId": "9ac0b77e-bda3-4784-b4de-7a76a53f2110",
        "facilityName": "SBA 7A Guaranteed Loan",
        "currencyCode": "USD",
        "asOfDate": "2026-09-01",
        "outstandingBalanceAmount": 4280029,
        "principalPaidToDateAmount": 19971,
        "interestPaidToDateAmount": 79883,
        "extraPrincipalToDateAmount": 0,
        "nextDueDate": "2026-09-01",
        "nextDueAmount": 49927,
        "scheduledDebtService30dAmount": 49927,
        "scheduledDebtService60dAmount": 99854,
        "scheduledDebtService90dAmount": 149781,
        "covenantInCompliance": true
      },
      {
        "facilityId": "fd95dfdd-7b13-41ea-b6a8-460938ac395d",
        "facilityName": "Seller Note 1",
        "currencyCode": "USD",
        "asOfDate": "2026-09-01",
        "outstandingBalanceAmount": 2142105,
        "principalPaidToDateAmount": 0,
        "interestPaidToDateAmount": 0,
        "extraPrincipalToDateAmount": 0,
        "nextDueDate": "2028-06-01",
        "nextDueAmount": null,
        "scheduledDebtService30dAmount": 0,
        "scheduledDebtService60dAmount": 0,
        "scheduledDebtService90dAmount": 0,
        "covenantInCompliance": null
      },
      {
        "facilityId": "b6fd95aa-7e8e-40ff-b969-6ab9301b74e1",
        "facilityName": "Seller Note 2 (Equity Injection)",
        "currencyCode": "USD",
        "asOfDate": "2026-09-01",
        "outstandingBalanceAmount": 357895,
        "principalPaidToDateAmount": 0,
        "interestPaidToDateAmount": 0,
        "extraPrincipalToDateAmount": 0,
        "nextDueDate": null,
        "nextDueAmount": null,
        "scheduledDebtService30dAmount": 0,
        "scheduledDebtService60dAmount": 0,
        "scheduledDebtService90dAmount": 0,
        "covenantInCompliance": null
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-09-01",
    "source": "debt_facilities,debt_payment_schedule,debt_payments_actual,debt_covenant_snapshots",
    "timeWindow": "90d",
    "calculationVersion": "v1",
    "currency": "USD"
  }
}
```

### `GET /api/v1/debt-service/facilities`

```json
{
  "data": [
    {
      "id": "9ac0b77e-bda3-4784-b4de-7a76a53f2110",
      "externalId": "citizens_sba_7a_2026",
      "lenderName": "Citizens Bank",
      "facilityName": "SBA 7A Guaranteed Loan",
      "facilityType": "term_loan",
      "originalPrincipalAmount": 4300000,
      "currencyCode": "USD",
      "originationDate": "2026-05-01",
      "firstPaymentDate": "2026-06-01",
      "maturityDate": "2036-05-01",
      "paymentDayOfMonth": 1,
      "prepaymentPenaltyMode": "none",
      "status": "active",
      "notes": "Bank term loan from Citizens. Fixed 7.00 percent, amortizing.",
      "createdAt": "2026-02-27T20:10:00Z",
      "updatedAt": "2026-02-27T20:10:00Z"
    },
    {
      "id": "fd95dfdd-7b13-41ea-b6a8-460938ac395d",
      "externalId": "seller_note_1_ian_swain_sr_2026",
      "lenderName": "Ian Swain Sr.",
      "facilityName": "Seller Note 1",
      "facilityType": "seller_note",
      "originalPrincipalAmount": 2142105,
      "currencyCode": "USD",
      "originationDate": "2026-05-01",
      "firstPaymentDate": "2028-06-01",
      "maturityDate": "2036-05-01",
      "paymentDayOfMonth": 1,
      "prepaymentPenaltyMode": "unknown",
      "status": "active",
      "notes": "Confirmed seller note: 10-year term, 4.65 percent fixed, 2-year standby.",
      "createdAt": "2026-02-27T20:10:00Z",
      "updatedAt": "2026-02-27T20:10:00Z"
    }
  ],
  "pagination": null,
  "meta": {
    "asOfDate": "2026-09-01",
    "source": "debt_facilities,debt_facility_terms",
    "timeWindow": "na",
    "calculationVersion": "v1",
    "currency": "USD"
  }
}
```

### `POST /api/v1/debt-service/payments`

Request:

```json
{
  "facilityId": "9ac0b77e-bda3-4784-b4de-7a76a53f2110",
  "paymentDate": "2026-09-01",
  "principalPaidAmount": 24843,
  "interestPaidAmount": 25083,
  "extraPrincipalAmount": 0,
  "feeAmount": 0,
  "sourceAccount": "citizens-operating",
  "reference": "SIM-2026-09-01"
}
```

Response:

```json
{
  "data": {
    "paymentId": "4d23313a-f3fa-40c9-b6f4-10db9ed953e4",
    "facilityId": "9ac0b77e-bda3-4784-b4de-7a76a53f2110",
    "paymentDate": "2026-09-01",
    "principalPaidAmount": 24843,
    "interestPaidAmount": 25083,
    "extraPrincipalAmount": 0,
    "remainingBalanceAmount": 4280029
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-09-01",
    "source": "debt_payments_actual,debt_balance_snapshots",
    "timeWindow": "",
    "calculationVersion": "v1",
    "currency": "USD"
  }
}
```

### `GET /api/v1/debt-service/covenants`

```json
{
  "data": [
    {
      "covenantId": "2fd79aca-abd2-4ccc-bf8f-6892f2f54c56",
      "facilityId": "9ac0b77e-bda3-4784-b4de-7a76a53f2110",
      "covenantCode": "dscr_min_1_25",
      "covenantName": "Debt Service Coverage Ratio Minimum",
      "metricName": "dscr",
      "thresholdValue": 1.25,
      "comparisonOperator": "gte",
      "asOfDate": "2026-09-01",
      "measuredValue": 1.41,
      "isInCompliance": true,
      "note": null
    }
  ],
  "pagination": null,
  "meta": {
    "asOfDate": "2026-09-01",
    "source": "debt_covenants,debt_covenant_snapshots,v_debt_service_overview",
    "timeWindow": "na",
    "calculationVersion": "v1",
    "currency": "USD"
  }
}
```

## Itinerary Revenue

### `GET /api/v1/itinerary-revenue/outlook`

```json
{
  "data": {
    "summary": {
      "totalOnBooksGrossAmount": 15038653.17,
      "totalPotentialGrossAmount": 4604839.14,
      "totalExpectedGrossAmount": 19183008.396,
      "totalExpectedGrossProfitAmount": 3935936.625,
      "totalExpectedMarginAmount": 15247071.771,
      "totalOnBooksPaxCount": 1070,
      "totalPotentialPaxCount": 359,
      "totalExpectedPaxCount": 1393.1,
      "totalForecastGrossAmount": 19831162.5754,
      "totalTargetGrossAmount": 25427521.8736,
      "totalForecastGrossProfitAmount": 4459182.6945,
      "totalTargetGrossProfitAmount": 5741261.3216,
      "totalForecastPaxCount": 1687.829,
      "totalTargetPaxCount": 2114.56
    },
    "timeline": [
      {
        "periodStart": "2026-02-01",
        "periodEnd": "2026-02-28",
        "onBooksGrossAmount": 2157995.0,
        "potentialGrossAmount": 0.0,
        "expectedGrossAmount": 2157995.0,
        "onBooksGrossProfitAmount": 505810.71,
        "potentialGrossProfitAmount": 0.0,
        "expectedGrossProfitAmount": 505810.71,
        "onBooksPaxCount": 147,
        "potentialPaxCount": 0,
        "expectedPaxCount": 147,
        "expectedMarginAmount": 1652184.29,
        "expectedMarginPct": 0.7656,
        "forecastGrossAmount": 1604119.98,
        "targetGrossAmount": 3035386.62,
        "forecastGrossProfitAmount": 359933.89,
        "targetGrossProfitAmount": 685196.16,
        "forecastPaxCount": 136.66,
        "targetPaxCount": 184.8
      }
    ],
    "closeRatio": 0.9
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-26",
    "source": "mv_itinerary_revenue_monthly,mv_itinerary_revenue_weekly,mv_itinerary_pipeline_stages",
    "timeWindow": "12m",
    "calculationVersion": "v2",
    "currency": "USD"
  }
}
```

### `GET /api/v1/itinerary-revenue/actuals-yoy`

```json
{
  "data": {
    "years": [2024, 2025, 2026],
    "timeline": [
      {
        "year": 2026,
        "month": 1,
        "monthLabel": "Jan",
        "itineraryCount": 39,
        "paxCount": 112,
        "grossAmount": 2799200.23,
        "grossProfitAmount": 603727.35,
        "marginAmount": 2195472.88,
        "tradeCommissionAmount": 2195472.88,
        "marginPct": 0.7844,
        "avgGrossPerItinerary": 71774.36,
        "avgGrossProfitPerItinerary": 15480.19,
        "avgGrossPerPax": 24992.86,
        "avgGrossProfitPerPax": 5381.49,
        "avgNumberOfDays": 19.97,
        "avgNumberOfNights": 18.97,
        "grossShareOfYearPct": 0.1581,
        "itineraryShareOfYearPct": 0.1167
      }
    ],
    "yearSummaries": [
      {
        "year": 2026,
        "itineraryCount": 417,
        "paxCount": 1208,
        "grossAmount": 16191087.17,
        "grossProfitAmount": 3370667.56,
        "marginAmount": 12820419.61,
        "tradeCommissionAmount": 12820419.61,
        "marginPct": 0.7918,
        "avgGrossPerItinerary": 38827.55,
        "avgGrossProfitPerItinerary": 8083.14,
        "avgGrossPerPax": 13403.22,
        "avgGrossProfitPerPax": 2790.29,
        "avgNumberOfDays": 17.83,
        "avgNumberOfNights": 16.83
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-26",
    "source": "mv_itinerary_revenue_monthly,mv_itinerary_consortia_actuals_monthly",
    "timeWindow": "3y",
    "calculationVersion": "v2",
    "currency": "USD"
  }
}
```

### `GET /api/v1/itinerary-revenue/booked-revenue-yoy`

Usage note: booked-revenue YoY is **close-date keyed** and uses the closed lifecycle (`closed_won` + `closed_active`) from `itinerary_status_reference`.

```json
{
  "data": {
    "years": [2024, 2025, 2026],
    "timeline": [
      {
        "year": 2026,
        "month": 1,
        "monthLabel": "Jan",
        "itineraryCount": 31,
        "paxCount": 97,
        "grossAmount": 2148920.57,
        "grossProfitAmount": 489201.14,
        "marginAmount": 1659719.43,
        "tradeCommissionAmount": 0.0,
        "marginPct": 0.7724
      }
    ],
    "yearSummaries": [
      {
        "year": 2026,
        "itineraryCount": 352,
        "paxCount": 1084,
        "grossAmount": 13755204.89,
        "grossProfitAmount": 2918401.66,
        "marginAmount": 10836803.23,
        "tradeCommissionAmount": 0.0,
        "marginPct": 0.7878
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-28",
    "source": "vw_semantic_booked_revenue_company_monthly_v2",
    "timeWindow": "5y",
    "calculationVersion": "v2",
    "currency": "USD"
  }
}
```

### `GET /api/v1/itinerary-revenue/actuals-channels-comparison?period_type=year&year=2026`

Travel Agencies channel tables use this contract. Each row may include `priorYear` with booking-pace comparison values: same travel cohort window, but only itineraries booked (`close_date`) on or before the matching as-of cutoff (closed lifecycle: `closed_won` + `closed_active`).

```json
{
  "data": {
    "topConsortia": [
      {
        "label": "Virtuoso",
        "itineraryCount": 128,
        "paxCount": 402,
        "grossAmount": 12000000.0,
        "grossProfitAmount": 2100000.0,
        "marginAmount": 9900000.0,
        "tradeCommissionAmount": 180000.0,
        "priorYear": {
          "itineraryCount": 110,
          "paxCount": 355,
          "grossAmount": 10100000.0,
          "grossProfitAmount": 1750000.0,
          "tradeCommissionAmount": 150000.0
        }
      }
    ],
    "topTradeAgencies": [
      {
        "label": "Northstar Travel",
        "itineraryCount": 44,
        "paxCount": 120,
        "grossAmount": 920000.0,
        "grossProfitAmount": 168000.0,
        "marginAmount": 752000.0,
        "tradeCommissionAmount": 42000.0,
        "priorYear": {
          "itineraryCount": 38,
          "paxCount": 101,
          "grossAmount": 801000.0,
          "grossProfitAmount": 142000.0,
          "tradeCommissionAmount": 36000.0
        }
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-14",
    "source": "vw_channel_consortia_booking_pace_monthly_v2,vw_channel_trade_agency_booking_pace_monthly_v2",
    "timeWindow": "year",
    "calculationVersion": "v3",
    "currency": null
  }
}
```

## Suppliers

### `GET /api/v1/suppliers/leaderboard?year=2026&top_n=25&location_query=Australia`

```json
{
  "data": {
    "periodStart": "2026-01-01",
    "periodEnd": "2026-03-29",
    "year": 2026,
    "sortBy": "gross_profit",
    "sortOrder": "desc",
    "topN": 25,
    "filters": {
      "country": "",
      "region": "",
      "city": "",
      "locationQuery": "Australia",
      "supplierQuery": ""
    },
    "rankings": [
      {
        "rank": 1,
        "supplierId": "7fd7cf0a-dca6-4dcb-8da7-364109bf5d57",
        "supplierExternalId": "a15A000001XyzAb",
        "supplierName": "Harbour Luxe Collection",
        "itineraryCount": 38,
        "itineraryItemCount": 112,
        "quantity": 172.0,
        "grossAmount": 2849021.77,
        "grossProfitAmount": 812944.11,
        "marginAmount": 2036077.66,
        "marginPct": 0.7146,
        "priorItineraryCount": 31,
        "priorItineraryItemCount": 96,
        "priorGrossAmount": 2418810.03,
        "priorGrossProfitAmount": 694100.52,
        "itineraryCountVariancePct": 0.2258,
        "grossVariancePct": 0.1779,
        "grossProfitVariancePct": 0.1712
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-28",
    "source": "mv_supplier_travel_revenue_monthly_v1",
    "timeWindow": "2026",
    "calculationVersion": "v1",
    "currency": "USD"
  }
}
```

### `GET /api/v1/suppliers/{supplier_id}/profile?year=2026`

```json
{
  "data": {
    "supplier": {
      "supplierId": "7fd7cf0a-dca6-4dcb-8da7-364109bf5d57",
      "supplierExternalId": "a15A000001XyzAb",
      "supplierName": "Harbour Luxe Collection",
      "supplierCode": "HLC",
      "supplierType": "Accommodation",
      "defaultCurrency": "USD",
      "contactEmail": "ops@harbourluxe.example",
      "contactPhone": "+1-555-0112",
      "addressCountry": "Australia"
    },
    "periodStart": "2026-01-01",
    "periodEnd": "2026-03-29",
    "year": 2026,
    "kpis": {
      "itineraryCount": 38,
      "itineraryItemCount": 112,
      "quantity": 172.0,
      "grossAmount": 2849021.77,
      "grossProfitAmount": 812944.11,
      "marginAmount": 2036077.66,
      "marginPct": 0.7146
    },
    "yoySeries": [
      {
        "metric": "grossRevenue",
        "currentYear": 2026,
        "priorYear": 2025,
        "points": [
          {
            "month": 1,
            "monthLabel": "Jan",
            "currentYearValue": 430221.5,
            "priorYearValue": 377002.3
          }
        ],
        "totalCurrentYearValue": 2849021.77,
        "totalPriorYearValue": 2418810.03,
        "yoyDeltaPct": 0.1779
      }
    ],
    "topLocations": [
      {
        "country": "Australia",
        "region": "New South Wales",
        "city": "Sydney",
        "itineraryCount": 21,
        "itineraryItemCount": 64,
        "grossAmount": 1662930.4,
        "grossProfitAmount": 485322.97
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-28",
    "source": "mv_supplier_travel_revenue_monthly_v1,suppliers",
    "timeWindow": "2026",
    "calculationVersion": "v1",
    "currency": "USD"
  }
}
```

## Travel Consultant

### `GET /api/v1/travel-consultants/leaderboard`

Usage note: for `period_type=year` when `year` is the **current** calendar year, services set `periodEnd` to **today** (YTD), not Dec 31. Primary ranking metrics are **travel-start-date** (`travelRevenue`, `travelGrossProfit`) from `vw_travel_consultant_travel_monthly_v2` (closed lifecycle: `closed_won` + `closed_active`). Default `sort_by` is `travel_revenue` (also `margin_pct`). Funnel fields (`itineraryCreatedCount`, `itineraryWonCount`, …) come from lead views; `travelYtd*` and `travelGrossProfitVariance*` fields reflect travel-basis PYTD comparisons computed in the service. The leaderboard top-summary callouts use the full returned `travelRankings` set for team totals and render API-provided `highlights` directly.

```json
{
  "data": {
    "periodStart": "2026-01-01",
    "periodEnd": "2026-03-14",
    "periodType": "year",
    "domain": "travel",
    "sortBy": "travel_revenue",
    "sortOrder": "desc",
    "travelRankings": [
      {
        "rank": 1,
        "employeeId": "e2f9f8d2-aaaa-bbbb-cccc-2a2a2a2a2a2a",
        "employeeExternalId": "005A0000001XyzQ",
        "firstName": "Alex",
        "lastName": "Taylor",
        "email": "alex@swain.com",
        "itineraryCount": 14,
        "paxCount": 38,
        "travelRevenue": 148200.0,
        "travelGrossProfit": 102900.0,
        "marginAmount": 45300.0,
        "marginPct": 0.3057,
        "itineraryCreatedCount": 22,
        "itineraryWonCount": 11,
        "itineraryLostCount": 5,
        "priorItineraryCreatedCount": 20,
        "priorItineraryWonCount": 9,
        "priorItineraryLostCount": 6,
        "itineraryCreatedVariancePct": 0.1,
        "itineraryWonVariancePct": 0.2222,
        "itineraryLostVariancePct": -0.1667,
        "travelYtdVariancePct": 0.117,
        "travelYtdCurrentRevenue": 148200.0,
        "travelYtdPriorRevenue": 132800.0,
        "priorTravelGrossProfit": 92000.0,
        "travelGrossProfitVariancePct": 0.1185
      }
    ],
    "highlights": [
      {
        "key": "top_mover",
        "title": "Top Mover",
        "description": "Alex Taylor leads target pace (8.1%).",
        "trendDirection": "up",
        "trendStrength": "high"
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-14",
    "source": "vw_travel_consultant_travel_monthly_v2,vw_travel_consultant_lead_monthly_v2",
    "timeWindow": "year",
    "calculationVersion": "v2",
    "currency": "USD"
  }
}
```

## Travel Trade

### `GET /api/v1/travel-agents/leaderboard`

Usage note: Travel Agencies UI uses this payload for top-performance bars and rankings tables (fixed current-year query from the route loader). Standalone top summary KPI cards were removed earlier. For `period_type=year`, service windows are full calendar year (`Jan 1` through `Dec 31`) for the selected year.

```json
{
  "data": {
    "periodStart": "2026-01-01",
    "periodEnd": "2026-12-31",
    "periodType": "year",
    "sortBy": "gross_profit",
    "sortOrder": "desc",
    "topN": 10,
    "rankings": [
      {
        "rank": 1,
        "agentId": "62f8c8de-10ac-4a32-a8f1-f05f32b4f4f6",
        "agentExternalId": "003A000001ABC123",
        "agentName": "Taylor Morgan",
        "agentEmail": "taylor.morgan@agency.com",
        "agencyId": "98f0ea5f-7f79-4f06-9f0d-14dbf63a7736",
        "agencyExternalId": "001A000001ZZZ123",
        "agencyName": "Northstar Travel",
        "leadsCount": 54,
        "convertedLeadsCount": 21,
        "bookedItinerariesCount": 19,
        "grossAmount": 912000.0,
        "grossProfitAmount": 162000.0,
        "conversionRate": 0.3889
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-14",
    "source": "vw_travel_agent_lead_monthly_v2,vw_travel_agent_booked_monthly_v2",
    "timeWindow": "year",
    "calculationVersion": "v2",
    "currency": null
  }
}
```

### `GET /api/v1/travel-agents/{agent_id}/profile` (abbreviated)

Agent profiles pair **lead-created** KPIs (`kpis.*`) with **travel-period** operational lists (`bookedItineraries`, `currentTravelingFiles`, `topOpenItineraries`). Arrays omitted here are still returned (often empty).

```json
{
  "data": {
    "agent": {
      "agentId": "62f8c8de-10ac-4a32-a8f1-f05f32b4f4f6",
      "agentExternalId": "003A000001ABC123",
      "agentName": "Taylor Morgan",
      "agentEmail": "taylor.morgan@agency.com",
      "agencyId": "98f0ea5f-7f79-4f06-9f0d-14dbf63a7736",
      "agencyExternalId": "001A000001ZZZ123",
      "agencyName": "Northstar Travel"
    },
    "periodStart": "2026-01-01",
    "periodEnd": "2026-12-31",
    "periodType": "year",
    "kpis": {
      "leadsCount": 1,
      "convertedLeadsCount": 0,
      "bookedItinerariesCount": 1,
      "grossAmount": 42000.0,
      "grossProfitAmount": 9100.0,
      "conversionRate": 0.0
    },
    "bookedItineraries": [
      {
        "itineraryId": "6f2c2b2a-1111-2222-3333-444455556666",
        "itineraryNumber": "IT-204481",
        "itineraryName": "Botswana Luxury Safari",
        "itineraryStatus": "Closed Won",
        "travelConsultantName": "Alex Taylor",
        "travelStartDate": "2026-04-12",
        "travelEndDate": "2026-04-22",
        "closeDate": "2026-02-02",
        "createdAt": "2026-01-05",
        "grossAmount": 42000.0,
        "grossProfitAmount": 9100.0
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-14",
    "source": "vw_travel_agent_lead_monthly_v2,vw_travel_agent_booked_monthly_v2,vw_travel_agent_consultant_affinity_monthly_v2,vw_travel_agent_booking_pace_monthly_v2,itineraries,contacts,itinerary_status_reference",
    "timeWindow": "year",
    "calculationVersion": "v2",
    "currency": "USD"
  }
}
```

## FX

### `GET /api/v1/fx/invoice-pressure`

```json
{
  "data": [
    {
      "currencyCode": "AUD",
      "due7dAmount": 40200.0,
      "due30dAmount": 210305.0,
      "due60dAmount": 315920.0,
      "due90dAmount": 418922.44,
      "invoicesDue30dCount": 77,
      "nextDueDate": "2026-03-04"
    }
  ],
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-02",
    "source": "ap_pressure_30_60_90_v1",
    "timeWindow": "",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### `POST /api/v1/fx/transactions`

Request:

```json
{
  "currencyCode": "AUD",
  "transactionType": "BUY",
  "transactionDate": "2026-02-18",
  "amount": 50000,
  "exchangeRate": 1.53,
  "referenceNumber": "WIRE-22818",
  "notes": "Top-up for near-term supplier invoices"
}
```

Response:

```json
{
  "data": {
    "id": "tx-uuid",
    "currencyCode": "AUD",
    "transactionType": "BUY",
    "transactionDate": "2026-02-18",
    "amount": 50000,
    "exchangeRate": 1.53,
    "usdEquivalent": 32679.74,
    "balanceAfter": 142500,
    "supplierInvoiceId": null,
    "signalId": null,
    "referenceNumber": "WIRE-22818",
    "notes": "Top-up for near-term supplier invoices",
    "enteredBy": null,
    "createdAt": "2026-02-18T08:33:00Z",
    "updatedAt": "2026-02-18T08:33:00Z"
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-26",
    "source": "fx_transactions",
    "timeWindow": "",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

## Marketing Web Analytics

### `GET /api/v1/marketing/web-analytics/overview`

KPI window values in `kpis[]` are sourced from synced canonical period summaries (not client-side day-level user summations).
Optional query: `country=United States|all`.

```json
{
  "data": {
    "kpis": [
      {
        "metricKey": "sessions",
        "label": "Sessions",
        "format": "integer",
        "currentValue": 14230,
        "previousValue": 13102,
        "yearAgoValue": 10498,
        "dayOverDayDeltaPct": 0.073,
        "monthOverMonthDeltaPct": 0.0861,
        "yearOverYearDeltaPct": 0.3555
      }
    ],
    "trend": [
      {
        "snapshotDate": "2026-03-01",
        "sessions": 523,
        "totalUsers": 402,
        "engagedSessions": 389,
        "keyEvents": 37,
        "engagementRate": 0.744
      }
    ],
    "topLandingPages": [
      {
        "snapshotDate": "2026-03-03",
        "landingPage": "/",
        "sessions": 4220,
        "totalUsers": 3052,
        "engagementRate": 0.693,
        "keyEvents": 211,
        "avgSessionDurationSeconds": null
      }
    ],
    "channels": [
      {
        "channelName": "Organic Search",
        "sessions": 5340,
        "totalUsers": 4022,
        "engagementRate": 0.721,
        "keyEvents": 286
      }
    ],
    "events": [
      {
        "snapshotDate": "2026-03-03",
        "eventName": "page_view",
        "eventCount": 33120,
        "totalUsers": 4089,
        "eventValueAmount": null
      }
    ],
    "searchConsoleConnected": false,
    "currency": "USD",
    "timezone": "America/New_York"
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-03",
    "source": "ga4",
    "timeWindow": "30d",
    "calculationVersion": "v1",
    "marketScope": "all",
    "marketLabel": "All markets",
    "currency": null,
    "dataStatus": "live",
    "isStale": false,
    "degraded": false
  }
}
```

### `GET /api/v1/marketing/web-analytics/search`

Optional query: `days_back=<7|30|90...>&country=United States|all`.

```json
{
  "data": {
    "topLandingPages": [
      {
        "snapshotDate": "2026-03-03",
        "landingPage": "/destinations",
        "sessions": 910,
        "totalUsers": 706,
        "engagementRate": 0.581,
        "keyEvents": 22,
        "avgSessionDurationSeconds": null
      }
    ],
    "channels": [
      {
        "channelName": "Direct",
        "sessions": 1940,
        "totalUsers": 1532,
        "engagementRate": 0.664,
        "keyEvents": 88
      }
    ],
    "sourceMix": [
      {
        "sourceLabel": "google / organic",
        "source": "google",
        "medium": "organic",
        "channelName": "Organic Search",
        "sessions": 2140,
        "totalUsers": 1720,
        "engagedSessions": 1495,
        "keyEvents": 141,
        "engagementRate": 0.699,
        "keyEventRate": 0.066,
        "bounceRate": 0.41,
        "qualifiedSessionRate": 0.699,
        "qualityLabel": "qualified",
        "valueScore": 42.6
      }
    ],
    "referralSources": [
      {
        "sourceLabel": "tripadvisor.com / referral",
        "source": "tripadvisor.com",
        "medium": "referral",
        "channelName": "Referral",
        "sessions": 440,
        "totalUsers": 371,
        "engagedSessions": 298,
        "keyEvents": 49,
        "engagementRate": 0.677,
        "keyEventRate": 0.111,
        "bounceRate": 0.34,
        "qualifiedSessionRate": 0.677,
        "qualityLabel": "qualified",
        "valueScore": 35.0
      }
    ],
    "topValuableSources": [
      {
        "sourceLabel": "tripadvisor.com / referral",
        "source": "tripadvisor.com",
        "medium": "referral",
        "channelName": "Referral",
        "sessions": 440,
        "totalUsers": 371,
        "engagedSessions": 298,
        "keyEvents": 49,
        "engagementRate": 0.677,
        "keyEventRate": 0.111,
        "bounceRate": 0.34,
        "qualifiedSessionRate": 0.677,
        "qualityLabel": "qualified",
        "valueScore": 35.0
      }
    ],
    "internalSiteSearchTerms": [
      {
        "searchTerm": "african safari itinerary",
        "eventCount": 62,
        "totalUsers": 51
      },
      {
        "searchTerm": "botswana",
        "eventCount": 31,
        "totalUsers": 28
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-03",
    "source": "ga4",
    "timeWindow": "30d",
    "calculationVersion": "v1",
    "marketScope": "United States",
    "marketLabel": "United States",
    "currency": null,
    "dataStatus": "live",
    "isStale": false,
    "degraded": false
  }
}
```

### `GET /api/v1/marketing/web-analytics/search-console`

Optional query: `days_back=<7|30|90...>`.

```json
{
  "data": {
    "searchConsoleConnected": true,
    "connectionMessage": "Search Console is connected and Supabase snapshots are serving US-first search insights with benchmark market comparisons.",
    "dataMode": "snapshot",
    "asOfDate": "2026-03-09",
    "overview": {
      "totalClicks": 1482,
      "totalImpressions": 45211,
      "averageCtr": 0.0328,
      "averagePosition": 8.74,
      "clicksDeltaPct": 0.11,
      "impressionsDeltaPct": 0.07,
      "ctrDeltaPct": 0.04,
      "positionDelta": -0.62,
      "freshnessDays": 1
    },
    "topQueries": [
      {
        "query": "botswana safari",
        "clicks": 214,
        "impressions": 4180,
        "ctr": 0.0512,
        "averagePosition": 6.92,
        "isBranded": false,
        "intentBucket": "destination_intent",
        "termType": "short_tail",
        "positionBand": "4-10"
      }
    ],
    "topPages": [
      {
        "pagePath": "/destinations/botswana",
        "clicks": 308,
        "impressions": 5330,
        "ctr": 0.0578,
        "averagePosition": 6.24
      }
    ],
    "countryBreakdown": [
      {
        "label": "United States",
        "clicks": 840,
        "impressions": 26010,
        "ctr": 0.0323,
        "averagePosition": 8.42
      }
    ],
    "deviceBreakdown": [
      {
        "label": "mobile",
        "clicks": 932,
        "impressions": 29110,
        "ctr": 0.0320,
        "averagePosition": 9.18
      }
    ],
    "opportunities": [
      {
        "opportunityId": "low-ctr-query-1",
        "title": "Improve CTR on high-impression query",
        "summary": "'african safari tours' has strong demand but weak click-through.",
        "pagePath": null,
        "query": "african safari tours",
        "clicks": 57,
        "impressions": 2980,
        "ctr": 0.0191,
        "averagePosition": 7.31,
        "priorityScore": 100,
        "recommendedAction": "Refresh title/meta and align snippet value proposition with search intent.",
        "opportunityType": "low_ctr"
      }
    ],
    "challenges": [
      {
        "challengeId": "page-query-ctr-gap-1",
        "title": "Page underperforms for high-demand intent",
        "summary": "/destinations/kenya is receiving impressions but not converting demand into clicks.",
        "pagePath": "/destinations/kenya",
        "query": "kenya safari luxury",
        "clicks": 36,
        "impressions": 2140,
        "ctr": 0.0168,
        "averagePosition": 8.91,
        "severityScore": 100,
        "recommendedAction": "Rework on-page headings and SERP snippets, then validate search-intent alignment against top competing pages.",
        "challengeType": "page_ctr_gap"
      }
    ],
    "marketBenchmarks": [
      {
        "marketLabel": "United States",
        "clicks": 840,
        "impressions": 26010,
        "ctr": 0.0323,
        "averagePosition": 8.42
      },
      {
        "marketLabel": "Australia",
        "clicks": 171,
        "impressions": 6930,
        "ctr": 0.0247,
        "averagePosition": 9.71
      }
    ],
    "queryIntentBuckets": [
      {
        "bucketLabel": "destination_intent",
        "queryCount": 1291,
        "clicks": 506,
        "impressions": 90521,
        "averageCtr": 0.0056
      }
    ],
    "positionBandSummary": [
      {
        "bandLabel": "4-10",
        "queryCount": 605,
        "clicks": 278,
        "impressions": 42210,
        "averageCtr": 0.0066
      }
    ],
    "issues": [
      {
        "issueKey": "search_console_healthy",
        "label": "Search Console data is healthy",
        "status": "healthy",
        "detail": "Search Console snapshots are current and query/page datasets are populated."
      }
    ],
    "organicLandingPages": [
      {
        "snapshotDate": "2026-03-09",
        "landingPage": "/destinations/botswana",
        "sessions": 308,
        "totalUsers": 308,
        "engagementRate": 0.0578,
        "keyEvents": 0,
        "avgSessionDurationSeconds": null
      }
    ],
    "internalSiteSearchTerms": [
      {
        "searchTerm": "botswana safari cost",
        "eventCount": 33,
        "totalUsers": 26
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-09",
    "source": "gsc + supabase",
    "timeWindow": "30d",
    "calculationVersion": "v1",
    "marketScope": "United States",
    "marketLabel": "United States",
    "currency": null,
    "dataStatus": "live",
    "isStale": false,
    "degraded": false
  }
}
```

### `GET /api/v1/marketing/web-analytics/search-console/page-profile?page_path=<encoded>&days_back=30`

```json
{
  "data": {
    "pagePath": "https://www.swaindestinations.com/destinations/botswana",
    "asOfDate": "2026-03-09",
    "overview": {
      "totalClicks": 308,
      "totalImpressions": 5330,
      "averageCtr": 0.0578,
      "averagePosition": 6.24,
      "clicksDeltaPct": null,
      "impressionsDeltaPct": null,
      "ctrDeltaPct": null,
      "positionDelta": null,
      "freshnessDays": 1
    },
    "dailyTrend": [
      {
        "snapshotDate": "2026-03-08",
        "clicks": 34,
        "impressions": 612,
        "ctr": 0.0556,
        "averagePosition": 6.19
      }
    ],
    "topQueries": [
      {
        "query": "botswana safari",
        "clicks": 214,
        "impressions": 4180,
        "ctr": 0.0512,
        "averagePosition": 6.92,
        "isBranded": false,
        "intentBucket": null,
        "termType": null,
        "positionBand": null
      }
    ],
    "marketBenchmarks": [
      {
        "marketLabel": "United States",
        "clicks": 308,
        "impressions": 5330,
        "ctr": 0.0578,
        "averagePosition": 6.24
      },
      {
        "marketLabel": "Australia",
        "clicks": 57,
        "impressions": 1110,
        "ctr": 0.0514,
        "averagePosition": 7.83
      }
    ],
    "issues": [
      {
        "issueKey": "search_console_healthy",
        "label": "Search Console data is healthy",
        "status": "healthy",
        "detail": "Search Console snapshots are current and query/page datasets are populated."
      }
    ],
    "recommendedActions": [
      "Refresh title/meta to reinforce intent and improve click-through from rank 4-10 positions.",
      "Align hero copy with high-demand queries and validate against competing SERP snippets."
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-09",
    "source": "gsc + supabase",
    "timeWindow": "30d",
    "calculationVersion": "v1",
    "marketScope": "United States",
    "marketLabel": "United States",
    "currency": null,
    "dataStatus": "live",
    "isStale": false,
    "degraded": false
  }
}
```

### `GET /api/v1/marketing/web-analytics/ai-insights`

```json
{
  "data": [
    {
      "insightId": "improve-low-engagement-page",
      "priority": "high",
      "category": "content",
      "focusArea": "fix",
      "title": "Fix a high-traffic landing page before buying more traffic",
      "summary": "/welcome/australia/campaign/gday is bringing in volume, but user intent is not being converted into engagement.",
      "targetLabel": "/welcome/australia/campaign/gday",
      "targetPath": "/welcome/australia/campaign/gday",
      "ownerHint": "marketing",
      "primaryMetricLabel": "Engagement Rate",
      "impactScore": 94,
      "confidenceScore": 89,
      "evidencePoints": [
        "Sessions: 29867",
        "Engagement rate: 3.9%",
        "Key events: 0"
      ],
      "recommendedActions": [
        "Rewrite the headline, hero value proposition, and primary CTA to match the acquisition intent landing here.",
        "Reduce above-the-fold distraction and give one obvious next step.",
        "Treat this page as a live experiment candidate for one-week copy and CTA testing."
      ]
    }
  ],
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-03",
    "source": "ga4",
    "timeWindow": "30d",
    "calculationVersion": "v1",
    "marketScope": "all",
    "marketLabel": "All markets",
    "currency": null,
    "dataStatus": "live",
    "isStale": false,
    "degraded": false
  }
}
```

### `GET /api/v1/marketing/web-analytics/health`

```json
{
  "data": {
    "statuses": [
      {
        "key": "ga4Configuration",
        "label": "GA4 Configuration",
        "status": "connected",
        "detail": "GA4 credentials and property ID are configured."
      },
      {
        "key": "searchConsoleConnection",
        "label": "Search Console Connection",
        "status": "pending",
        "detail": "Search Console is deferred; connect later for keyword query analytics."
      },
      {
        "key": "latestSyncRun",
        "label": "Latest Sync Run",
        "status": "healthy",
        "detail": "Latest run status: success."
      }
    ],
    "lastSyncedAt": "2026-03-03T11:31:00Z",
    "latestRunStatus": "success"
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-03",
    "source": "ga4",
    "timeWindow": "30d",
    "calculationVersion": "v1",
    "marketScope": "all",
    "marketLabel": "All markets",
    "currency": null,
    "dataStatus": "live",
    "isStale": false,
    "degraded": false
  }
}
```

### `GET /api/v1/marketing/web-analytics/page-activity`

```json
{
  "data": {
    "snapshotDate": "2026-03-03",
    "metricGuide": "Quality score blends key event rate, engagement rate, and traffic scale to rank pages.",
    "bestPages": [
      {
        "snapshotDate": "2026-03-03",
        "pagePath": "/itinerary/luxury-safari-botswana",
        "pageTitle": "Luxury Safari Itinerary",
        "screenPageViews": 2140,
        "sessions": 1310,
        "totalUsers": 990,
        "engagedSessions": 1022,
        "keyEvents": 181,
        "engagementRate": 0.78,
        "keyEventRate": 0.138,
        "avgSessionDurationSeconds": 172.4,
        "qualityScore": 0.422,
        "isItineraryPage": true
      }
    ],
    "worstPages": [
      {
        "snapshotDate": "2026-03-03",
        "pagePath": "/blog/older-post",
        "pageTitle": "Travel Blog 2022",
        "screenPageViews": 802,
        "sessions": 650,
        "totalUsers": 600,
        "engagedSessions": 181,
        "keyEvents": 9,
        "engagementRate": 0.278,
        "keyEventRate": 0.014,
        "avgSessionDurationSeconds": 41.8,
        "qualityScore": 0.119,
        "isItineraryPage": false
      }
    ],
    "itineraryPages": [],
    "lookbookPages": [
      {
        "snapshotDate": "2026-03-03",
        "pagePath": "/about/lookbooks",
        "pageTitle": "Lookbooks",
        "screenPageViews": 980,
        "sessions": 721,
        "totalUsers": 612,
        "engagedSessions": 514,
        "keyEvents": 46,
        "engagementRate": 0.713,
        "keyEventRate": 0.064,
        "avgSessionDurationSeconds": 118.7,
        "qualityScore": 0.329,
        "isItineraryPage": false
      }
    ],
    "destinationPages": [
      {
        "snapshotDate": "2026-03-03",
        "pagePath": "/destinations/botswana",
        "pageTitle": "Botswana",
        "screenPageViews": 1430,
        "sessions": 1094,
        "totalUsers": 884,
        "engagedSessions": 826,
        "keyEvents": 91,
        "engagementRate": 0.755,
        "keyEventRate": 0.083,
        "avgSessionDurationSeconds": 164.2,
        "qualityScore": 0.397,
        "isItineraryPage": false
      }
    ],
    "allPages": []
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-03",
    "source": "ga4",
    "timeWindow": "30d",
    "calculationVersion": "v1",
    "marketScope": "all",
    "marketLabel": "All markets",
    "currency": null,
    "dataStatus": "live",
    "isStale": false,
    "degraded": false
  }
}
```

### `GET /api/v1/marketing/web-analytics/geo`

```json
{
  "data": {
    "snapshotDate": "2026-03-03",
    "rows": [
      {
        "snapshotDate": "2026-03-03",
        "country": "United States",
        "region": "California",
        "city": "San Francisco",
        "sessions": 622,
        "totalUsers": 520,
        "engagedSessions": 484,
        "keyEvents": 71,
        "engagementRate": 0.778,
        "keyEventRate": 0.114
      }
    ],
    "topCountries": [],
    "demographics": [
      {
        "snapshotDate": "2026-03-03",
        "ageBracket": "25-34",
        "gender": "female",
        "sessions": 488,
        "totalUsers": 401,
        "engagedSessions": 371,
        "keyEvents": 57,
        "engagementRate": 0.76
      }
    ],
    "devices": [
      {
        "snapshotDate": "2026-03-03",
        "deviceCategory": "mobile",
        "sessions": 1640,
        "totalUsers": 1388,
        "engagedSessions": 1186,
        "keyEvents": 171,
        "engagementRate": 0.723
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-03",
    "source": "ga4",
    "timeWindow": "30d",
    "calculationVersion": "v1",
    "marketScope": "all",
    "marketLabel": "All markets",
    "currency": null,
    "dataStatus": "live",
    "isStale": false,
    "degraded": false
  }
}
```

### `GET /api/v1/marketing/web-analytics/events`

```json
{
  "data": {
    "snapshotDate": "2026-03-03",
    "events": [
      {
        "eventName": "generate_lead",
        "eventCount": 152,
        "totalUsers": 140,
        "eventValueAmount": null,
        "category": "conversion",
        "description": "User completed a lead action.",
        "isConversionEvent": true
      },
      {
        "eventName": "page_view",
        "eventCount": 33120,
        "totalUsers": 4089,
        "eventValueAmount": null,
        "category": "navigation",
        "description": "A page load or page route view.",
        "isConversionEvent": false
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-03",
    "source": "ga4",
    "timeWindow": "30d",
    "calculationVersion": "v1",
    "marketScope": "all",
    "marketLabel": "All markets",
    "currency": null,
    "dataStatus": "live",
    "isStale": false,
    "degraded": false
  }
}
```

## AI Insights

### `GET /api/v1/ai-insights/recommendations`

```json
{
  "data": {
    "items": [
      {
        "id": "9c02bde2-d0b5-4025-b6b0-122f7f8ec0a9",
        "insightEventId": "c2ce9cb2-3fd2-4a81-a33a-65f6a8e4e126",
        "domain": "travel_consultant",
        "status": "new",
        "entityType": "employee",
        "entityId": "employee-123",
        "title": "Alex coaching opportunity",
        "summary": "Conversion is below benchmark for selected period.",
        "recommendedAction": "Run pipeline review and establish weekly close plan.",
        "priority": 2,
        "confidence": 0.82,
        "ownerUserId": null,
        "dueDate": null,
        "resolutionNote": null,
        "generatedAt": "2026-02-16T15:02:00Z",
        "completedAt": null,
        "updatedAt": "2026-02-16T15:02:00Z"
      }
    ]
  },
  "pagination": {
    "page": 1,
    "pageSize": 25,
    "totalItems": 1,
    "totalPages": 1
  },
  "meta": {
    "asOfDate": "2026-02-26",
    "source": "ai_recommendation_queue",
    "timeWindow": "rolling",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

## Data Jobs Control Plane

### `GET /api/v1/data-jobs`

```json
{
  "data": [
    {
      "id": "5309debe-a33f-4574-8ce4-b6d98884b5a6",
      "jobKey": "fx-rates-pull",
      "runnerKey": "fx.rates.pull",
      "displayName": "FX Rates Pull",
      "jobKind": "source_ingestion",
      "scheduleMode": "recurring",
      "enabled": true,
      "scheduleCron": "*/15 * * * *",
      "scheduleTimezone": "UTC",
      "nextRunAt": "2026-03-10T15:00:00Z",
      "maxRuntimeSeconds": 3600,
      "freshnessSlaMinutes": 30,
      "staleAfterMinutes": 60,
      "timeoutAfterMinutes": null,
      "retryBackoffMinutes": 30,
      "owner": "finance",
      "tags": ["fx", "rates"],
      "config": {},
      "createdAt": "2026-03-10T00:00:00Z",
      "updatedAt": "2026-03-10T00:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 50,
    "totalItems": 1,
    "totalPages": 1
  },
  "meta": {
    "asOfDate": "2026-03-10",
    "source": "data_jobs",
    "timeWindow": "",
    "calculationVersion": "v1",
    "currency": null,
    "dataStatus": "live",
    "isStale": false,
    "degraded": false
  }
}
```

### `GET /api/v1/data-jobs/health`

```json
{
  "data": [
    {
      "jobId": "5309debe-a33f-4574-8ce4-b6d98884b5a6",
      "jobKey": "fx-rates-pull",
      "displayName": "FX Rates Pull",
      "jobKind": "source_ingestion",
      "scheduleMode": "recurring",
      "enabled": true,
      "scheduleCron": "*/15 * * * *",
      "scheduleTimezone": "UTC",
      "nextRunAt": "2026-03-10T15:15:00Z",
      "lastRunId": "5f178c1f-bf8d-4da6-a6ae-0b580f2b74f0",
      "lastRunStatus": "success",
      "lastStartedAt": "2026-03-10T15:03:08.412Z",
      "lastFinishedAt": "2026-03-10T15:03:12.094Z",
      "lastDurationSeconds": 4,
      "dueNow": false
    }
  ],
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-10",
    "source": "data_job_health_v1",
    "timeWindow": "",
    "calculationVersion": "v1",
    "currency": null,
    "dataStatus": "live",
    "isStale": false,
    "degraded": false
  }
}
```

### `GET /api/v1/data-jobs/run-feed?page=1&page_size=50&job_key=fx-rates-pull&run_status=failed`

```json
{
  "data": [
    {
      "id": "b5a96ff4-95c6-4c7f-9ca6-eec19e61763b",
      "jobId": "5309debe-a33f-4574-8ce4-b6d98884b5a6",
      "jobKey": "fx-rates-pull",
      "displayName": "FX Rates Pull",
      "runKey": "fx-rates-pull:f5188e48-9c2f-4f55-8fbe-67a5e06ec9ff",
      "runStatus": "failed",
      "triggerType": "scheduler",
      "triggerSource": "scheduler_tick",
      "requestedBy": "system:scheduler",
      "requestedAt": "2026-03-10T15:15:00.000Z",
      "startedAt": "2026-03-10T15:15:00.112Z",
      "finishedAt": "2026-03-10T15:15:03.902Z",
      "blockedReason": null,
      "errorCode": "runner_failed",
      "errorMessage": "fx.rates.pull failed with exit code 1",
      "durationSeconds": 4,
      "outputSizeBytes": 128,
      "output": {
        "returnCode": 1
      },
      "metadata": {},
      "createdAt": "2026-03-10T15:15:00.112Z",
      "updatedAt": "2026-03-10T15:15:03.902Z"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 50,
    "totalItems": 1,
    "totalPages": 1
  },
  "meta": {
    "asOfDate": "2026-03-10",
    "source": "data_job_runs",
    "timeWindow": "",
    "calculationVersion": "v1",
    "currency": null,
    "dataStatus": "live",
    "isStale": false,
    "degraded": false
  }
}
```

### `POST /api/v1/data-jobs/fx-rates-pull/runs`

Request:

```json
{
  "triggerType": "manual",
  "triggerSource": "fx_command",
  "requestedBy": "frontend:fx_command",
  "metadata": {}
}
```

Response (shape matches other `POST .../runs` enqueue responses):

```json
{
  "data": {
    "id": "5f178c1f-bf8d-4da6-a6ae-0b580f2b74f0",
    "jobId": "5309debe-a33f-4574-8ce4-b6d98884b5a6",
    "runKey": "fx-rates-pull:f5188e48-9c2f-4f55-8fbe-67a5e06ec9ff",
    "runStatus": "success",
    "triggerType": "manual",
    "triggerSource": "fx_command",
    "requestedBy": "frontend:fx_command",
    "requestedAt": "2026-03-10T15:03:08.411Z",
    "startedAt": "2026-03-10T15:03:08.412Z",
    "finishedAt": "2026-03-10T15:03:12.094Z",
    "blockedReason": null,
    "errorCode": null,
    "errorMessage": null,
    "durationSeconds": 4,
    "outputSizeBytes": 112,
    "output": {
      "returnCode": 0
    },
    "metadata": {},
    "createdAt": "2026-03-10T15:03:08.412Z",
    "updatedAt": "2026-03-10T15:03:12.094Z"
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-10",
    "source": "data_jobs",
    "timeWindow": "",
    "calculationVersion": "v1",
    "currency": null,
    "dataStatus": "live",
    "isStale": false,
    "degraded": false
  }
}
```

### `GET /api/v1/data-job-runs/{run_id}` (Salesforce sync detail example)

```json
{
  "data": {
    "run": {
      "id": "2fb8bd0e-0bc5-4c8a-96de-f2f2b09b6f32",
      "jobId": "dd7d6423-f0b0-4dc2-8f8d-e6e06e1ca221",
      "runKey": "salesforce-readonly-sync:becf257f-bfac-46dd-88f4-62d12f6ec1aa",
      "runStatus": "success",
      "triggerType": "manual",
      "triggerSource": "settings_job_controls",
      "requestedBy": "frontend:settings",
      "requestedAt": "2026-03-17T13:00:02.113Z",
      "startedAt": "2026-03-17T13:00:02.292Z",
      "finishedAt": "2026-03-17T13:03:44.008Z",
      "blockedReason": null,
      "errorCode": null,
      "errorMessage": null,
      "durationSeconds": 222,
      "outputSizeBytes": 3982,
      "output": {
        "returnCode": 0,
        "stdout": "{\"status\":\"success\",\"runId\":\"2fb8bd0e-0bc5-4c8a-96de-f2f2b09b6f32\",\"windowStart\":\"2026-03-16T12:00:00+00:00\",\"upperBound\":\"2026-03-17T12:55:00+00:00\"}",
        "stderr": "",
        "command": ["python", "scripts/sync_salesforce_readonly.py"],
        "timedOut": false,
        "maxRuntimeSeconds": 3600,
        "parsed": {
          "status": "success",
          "runId": "2fb8bd0e-0bc5-4c8a-96de-f2f2b09b6f32",
          "windowStart": "2026-03-16T12:00:00+00:00",
          "upperBound": "2026-03-17T12:55:00+00:00",
          "apiLimits": {
            "dailyApiRequests": {
              "max": 100000,
              "remaining": 78342,
              "used": 21658,
              "usedPercent": 21.66
            }
          },
          "objectMetrics": {
            "employees": {
              "extracted": 214,
              "staged": 214,
              "loaded": 214,
              "skipped_duplicate_email": 0,
              "csv_bytes": 60212,
              "window_start": "2026-03-16T12:00:00+00:00"
            },
            "itinerary_items": {
              "extracted": 1189,
              "staged": 1173,
              "loaded": 1173,
              "skipped_unresolved": 16,
              "csv_bytes": 402118,
              "deleted_flagged": 2,
              "unresolved": 16,
              "window_start": "2026-03-16T12:00:00+00:00"
            }
          },
          "counters": {
            "jobsCreated": 8,
            "pollsMade": 41,
            "resultPagesRead": 16
          }
        }
      },
      "metadata": {},
      "createdAt": "2026-03-17T13:00:02.292Z",
      "updatedAt": "2026-03-17T13:03:44.008Z"
    },
    "steps": [],
    "checkpoints": [
      {
        "id": "c6384f40-6816-4af8-b014-7f8a4f0ae2dd",
        "runId": "2fb8bd0e-0bc5-4c8a-96de-f2f2b09b6f32",
        "stepId": "73fca43e-5198-429c-a0f1-f74bdf8df7d6",
        "sequence": 1,
        "checkpointKey": "run_start",
        "label": "Run started",
        "status": "started",
        "recordedAt": "2026-03-17T13:00:02.307Z",
        "payload": {
          "jobKey": "salesforce-readonly-sync",
          "triggerType": "manual",
          "triggerSource": "settings_job_controls"
        },
        "createdAt": "2026-03-17T13:00:02.307Z",
        "updatedAt": "2026-03-17T13:00:02.307Z"
      },
      {
        "id": "88f39ccd-5a2d-4c72-b0d5-61c54566f00b",
        "runId": "2fb8bd0e-0bc5-4c8a-96de-f2f2b09b6f32",
        "stepId": "73fca43e-5198-429c-a0f1-f74bdf8df7d6",
        "sequence": 2,
        "checkpointKey": "agencies",
        "label": "agencies started",
        "status": "started",
        "recordedAt": "2026-03-17T13:00:05.111Z",
        "payload": {
          "kind": "progress",
          "stage": "agencies",
          "status": "started"
        },
        "createdAt": "2026-03-17T13:00:05.111Z",
        "updatedAt": "2026-03-17T13:00:05.111Z"
      },
      {
        "id": "f13dce3f-cd11-4b7e-9fe2-3a6dc6fe4f1d",
        "runId": "2fb8bd0e-0bc5-4c8a-96de-f2f2b09b6f32",
        "stepId": "73fca43e-5198-429c-a0f1-f74bdf8df7d6",
        "sequence": 3,
        "checkpointKey": "run_terminal",
        "label": "Run completed",
        "status": "completed",
        "recordedAt": "2026-03-17T13:03:44.004Z",
        "payload": {
          "runStatus": "success"
        },
        "createdAt": "2026-03-17T13:03:44.004Z",
        "updatedAt": "2026-03-17T13:03:44.004Z"
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-17",
    "source": "data_job_runs",
    "timeWindow": "",
    "calculationVersion": "v1",
    "currency": null,
    "dataStatus": "live",
    "isStale": false,
    "degraded": false
  }
}
```

### `GET /api/v1/data-jobs/run-feed?job_key=salesforce-readonly-sync&run_status=blocked`

```json
{
  "data": [
    {
      "id": "f3e41f4f-c76f-4a4f-bdf8-c5b7045ba0ad",
      "jobId": "dd7d6423-f0b0-4dc2-8f8d-e6e06e1ca221",
      "jobKey": "salesforce-readonly-sync",
      "displayName": "Salesforce Readonly Sync",
      "runKey": "salesforce-readonly-sync:8e6f85f2-2155-45e7-9eb7-cf8a0ccb0377",
      "runStatus": "blocked",
      "triggerType": "scheduler",
      "triggerSource": "scheduler_tick",
      "requestedBy": "system:scheduler",
      "requestedAt": "2026-03-17T14:00:00.000Z",
      "startedAt": "2026-03-17T14:00:00.115Z",
      "finishedAt": "2026-03-17T14:00:00.901Z",
      "blockedReason": "salesforce.readonly.sync failed with exit code 1",
      "errorCode": "runner_failed",
      "errorMessage": "salesforce.readonly.sync failed with exit code 1",
      "durationSeconds": 1,
      "outputSizeBytes": 1187,
      "output": {
        "returnCode": 1,
        "timedOut": false,
        "parsed": {
          "status": "blocked",
          "runId": "f3e41f4f-c76f-4a4f-bdf8-c5b7045ba0ad",
          "reason": "Salesforce DailyApiRequests usage is at 86.20% which exceeds the configured 85.00% safety threshold",
          "windowStart": "2026-03-17T13:00:00+00:00",
          "upperBound": "2026-03-17T13:55:00+00:00"
        }
      },
      "metadata": {},
      "createdAt": "2026-03-17T14:00:00.115Z",
      "updatedAt": "2026-03-17T14:00:00.901Z"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 50,
    "totalItems": 1,
    "totalPages": 1
  },
  "meta": {
    "asOfDate": "2026-03-17",
    "source": "data_job_runs",
    "timeWindow": "",
    "calculationVersion": "v1",
    "currency": null,
    "dataStatus": "live",
    "isStale": false,
    "degraded": false
  }
}
```

## Related documentation

- [Frontend data queries](frontend-data-queries.md) — who calls which route
- [Backend code documentation](swainos-code-documentation-backend.md) — services, rollups, endpoint families
- [Frontend code documentation](swainos-code-documentation-frontend.md) — loaders and route structure
- [Terminology glossary](swainos-terminology-glossary.md) — naming and display terms
