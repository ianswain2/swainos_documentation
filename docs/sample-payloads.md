# Sample Payloads

Purpose: canonical request/response examples for active frontend/backend contracts.

## Conventions
- Response envelope: `{ data, pagination, meta }`
- Query params: `snake_case`
- JSON fields: `camelCase`
- Dates: ISO 8601

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

## Travel Consultant

### `GET /api/v1/travel-consultants/leaderboard`

```json
{
  "data": {
    "periodStart": "2026-01-01",
    "periodEnd": "2026-12-31",
    "periodType": "year",
    "domain": "travel",
    "sortBy": "booked_revenue",
    "sortOrder": "desc",
    "rankings": [
      {
        "rank": 1,
        "employeeId": "e2f9f8d2-aaaa-bbbb-cccc-2a2a2a2a2a2a",
        "employeeExternalId": "005A0000001XyzQ",
        "firstName": "Alex",
        "lastName": "Taylor",
        "email": "alex@swain.com",
        "itineraryCount": 14,
        "paxCount": 38,
        "bookedRevenue": 148200.0,
        "grossProfit": 102900.0,
        "marginAmount": 45300.0,
        "marginPct": 0.3057,
        "leadCount": 22,
        "closedWonCount": 11,
        "closedLostCount": 5,
        "conversionRate": 0.5,
        "closeRate": 0.6875,
        "avgSpeedToBookDays": 31.0,
        "growthTargetVariancePct": 0.0812,
        "yoyToDateVariancePct": 0.117
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
    "asOfDate": "2026-02-26",
    "source": "mv_travel_consultant_leaderboard_monthly,mv_travel_consultant_funnel_monthly",
    "timeWindow": "year",
    "calculationVersion": "v1",
    "currency": "USD"
  }
}
```

## Travel Trade

### `GET /api/v1/travel-agents/leaderboard`

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
    "asOfDate": "2026-02-26",
    "source": "travel_trade_lead_monthly_rollup,travel_trade_booked_itinerary_monthly_rollup,travel_agent_monthly_rollup",
    "timeWindow": "year",
    "calculationVersion": "v1",
    "currency": null
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
    "currency": null,
    "dataStatus": "live",
    "isStale": false,
    "degraded": false
  }
}
```

### `GET /api/v1/marketing/web-analytics/search`

```json
{
  "data": {
    "searchConsoleConnected": false,
    "connectionMessage": "Search Console is not connected yet. Connect it to unlock query-level SEO insights.",
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
    "topQueries": []
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-03",
    "source": "ga4 + gsc",
    "timeWindow": "30d",
    "calculationVersion": "v1",
    "currency": null,
    "dataStatus": "partial",
    "isStale": false,
    "degraded": true
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
      "title": "Improve a high-traffic landing page with low engagement",
      "summary": "/destinations is attracting traffic but engagement is trailing. This is likely a messaging or page intent mismatch.",
      "evidencePoints": [
        "Sessions: 910",
        "Engagement rate: 58.1%"
      ],
      "recommendedActions": [
        "Refresh above-the-fold value proposition and CTA hierarchy.",
        "Align headline and hero copy to the intent of top entry channels.",
        "Run two headline+CTA variants for one business week and compare key events."
      ]
    }
  ],
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-03",
    "source": "ga4",
    "timeWindow": "30d",
    "calculationVersion": "v1",
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
    "allPages": []
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-03",
    "source": "ga4",
    "timeWindow": "30d",
    "calculationVersion": "v1",
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
    "topCountries": []
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-03-03",
    "source": "ga4",
    "timeWindow": "30d",
    "calculationVersion": "v1",
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
