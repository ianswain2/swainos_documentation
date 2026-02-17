# Sample Payloads - Backend and Frontend Contracts

> Purpose: Canonical request/response examples for frontend consumption.

## Table of Contents
- [Conventions](#conventions)
- [Error Envelope (All endpoints)](#error-envelope-all-endpoints)
- [Core Platform Endpoints](#core-platform-endpoints)
- [AI Insights Endpoints](#ai-insights-endpoints)

## Conventions
- All responses use `{ data, pagination, meta }` envelopes.
- JSON properties are `camelCase`.
- Query params are `snake_case`.
- Dates use ISO 8601 strings.
- `commissionIncomeAmount` remains the API field name and is sourced from itinerary `gross_profit`.
- Closed-won analytics are based on the allowlist statuses:
  `Deposited/Confirming`, `Amendment in Progress`, `Pre-Departure`, `eDocs Sent`, `Traveling`, `Traveled`, `Cancel Fees`.

## Error Envelope (All endpoints)
**Response**
```json
{
  "error": {
    "code": "bad_request",
    "message": "Unsupported time window format",
    "details": null
  }
}
```

## Core Platform Endpoints

### GET /api/v1/health
**Response**
```json
{
  "data": {
    "status": "ok"
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-07",
    "source": "system",
    "timeWindow": "now",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### GET /api/v1/healthz
**Response**
```json
{
  "data": {
    "status": "ok"
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-07",
    "source": "system",
    "timeWindow": "now",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### GET /api/v1/revenue-bookings
**Status**: deprecated and removed.  
**Replacement**: `GET /api/v1/itinerary-revenue/outlook` and related owner-cockpit endpoints.

### GET /api/v1/revenue-bookings/{booking_id}
**Status**: deprecated and removed.  
**Replacement**: module-specific owner cockpit and future detail endpoints.

### GET /api/v1/cash-flow/summary
**Request (query params)**
```json
{
  "time_window": "90d",
  "currency_code": "USD",
  "page": 1,
  "page_size": 50
}
```

**Response**
```json
{
  "data": [
    {
      "currencyCode": "USD",
      "cashInTotal": 1000,
      "cashOutTotal": 600,
      "netCashTotal": 400
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 50,
    "totalItems": 1,
    "totalPages": 1
  },
  "meta": {
    "asOfDate": "2026-02-07",
    "source": "salesforce_kaptio",
    "timeWindow": "90d",
    "calculationVersion": "v1",
    "currency": "USD"
  }
}
```

### GET /api/v1/cash-flow/timeseries
**Request (query params)**
```json
{
  "time_window": "90d",
  "currency_code": "USD",
  "page": 1,
  "page_size": 50
}
```

**Response**
```json
{
  "data": [
    {
      "periodStart": "2026-02-01",
      "cashIn": 1000,
      "cashOut": 600,
      "netCash": 400
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 50,
    "totalItems": 1,
    "totalPages": 1
  },
  "meta": {
    "asOfDate": "2026-02-07",
    "source": "salesforce_kaptio",
    "timeWindow": "90d",
    "calculationVersion": "v1",
    "currency": "USD"
  }
}
```

### GET /api/v1/deposits/summary
**Request (query params)**
```json
{
  "time_window": "90d",
  "currency_code": "USD",
  "page": 1,
  "page_size": 50
}
```

**Response**
```json
{
  "data": [
    {
      "currencyCode": "USD",
      "totalDeposits": 1000,
      "receivedDeposits": 1000,
      "outstandingDeposits": 0
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 50,
    "totalItems": 1,
    "totalPages": 1
  },
  "meta": {
    "asOfDate": "2026-02-07",
    "source": "salesforce_kaptio",
    "timeWindow": "90d",
    "calculationVersion": "v1",
    "currency": "USD"
  }
}
```

### GET /api/v1/payments-out/summary
**Request (query params)**
```json
{
  "time_window": "90d",
  "currency_code": "USD",
  "page": 1,
  "page_size": 50
}
```

**Response**
```json
{
  "data": [
    {
      "currencyCode": "USD",
      "totalInvoices": 800,
      "paidAmount": 400,
      "outstandingAmount": 400
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 50,
    "totalItems": 1,
    "totalPages": 1
  },
  "meta": {
    "asOfDate": "2026-02-07",
    "source": "salesforce_kaptio",
    "timeWindow": "90d",
    "calculationVersion": "v1",
    "currency": "USD"
  }
}
```

### GET /api/v1/booking-forecasts
**Request (query params)**
```json
{
  "lookback_months": 12,
  "horizon_months": 3,
  "page": 1,
  "page_size": 50
}
```

**Response**
```json
{
  "data": [
    {
      "periodStart": "2026-03-01",
      "projectedBookings": 5,
      "confidence": 0.7
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 50,
    "totalItems": 1,
    "totalPages": 1
  },
  "meta": {
    "asOfDate": "2026-02-07",
    "source": "salesforce_kaptio",
    "timeWindow": "12m",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### GET /api/v1/itinerary-trends
**Request (query params)**
```json
{
  "time_window": "12m"
}
```

**Response**
```json
{
  "data": {
    "summary": {
      "createdItineraries": 120,
      "closedItineraries": 0,
      "travelStartItineraries": 90,
      "travelEndItineraries": 86
    },
    "timeline": [
      {
        "periodStart": "2025-09-01",
        "createdCount": 8,
        "closedCount": 0,
        "travelStartCount": 6,
        "travelEndCount": 5
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-07",
    "source": "salesforce_kaptio",
    "timeWindow": "12m",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### GET /api/v1/itinerary-pipeline
**Status**: deprecated and removed.  
**Replacement**: `GET /api/v1/itinerary-revenue/outlook` and `GET /api/v1/itinerary-revenue/conversion`.

### GET /api/v1/itinerary-revenue/outlook
**Request (query params)**
```json
{
  "time_window": "12m",
  "grain": "monthly",
  "currency_code": "USD"
}
```

**Response**
```json
{
  "data": {
    "summary": {
      "totalOnBooksGrossAmount": 982000.0,
      "totalPotentialGrossAmount": 244000.0,
      "totalExpectedGrossAmount": 1226000.0,
      "totalExpectedCommissionIncomeAmount": 986400.0,
      "totalExpectedMarginAmount": 239600.0,
      "totalOnBooksPaxCount": 1320,
      "totalPotentialPaxCount": 284.5,
      "totalExpectedPaxCount": 1604.5
    },
    "timeline": [
      {
        "periodStart": "2026-03-01",
        "periodEnd": "2026-03-31",
        "onBooksGrossAmount": 141000.0,
        "potentialGrossAmount": 39000.0,
        "expectedGrossAmount": 180000.0,
        "onBooksCommissionIncomeAmount": 113000.0,
        "potentialCommissionIncomeAmount": 31500.0,
        "expectedCommissionIncomeAmount": 144500.0,
        "onBooksPaxCount": 176,
        "potentialPaxCount": 33.1,
        "expectedPaxCount": 209.1,
        "expectedMarginAmount": 35500.0,
        "expectedMarginPct": 0.1972
      }
    ],
    "closeRatio": 0.5421
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-11",
    "source": "mv_itinerary_revenue_monthly,mv_itinerary_revenue_weekly,mv_itinerary_pipeline_stages",
    "timeWindow": "12m",
    "calculationVersion": "v2",
    "currency": "USD"
  }
}
```

### GET /api/v1/itinerary-revenue/deposits
**Response**
```json
{
  "data": {
    "timeline": [
      {
        "periodStart": "2026-03-01",
        "periodEnd": "2026-03-31",
        "closedItineraryCount": 48,
        "closedGrossAmount": 640000.0,
        "depositReceivedAmount": 154000.0,
        "targetDepositAmount": 160000.0,
        "depositGapAmount": -6000.0,
        "depositCoverageRatio": 0.9625
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-11",
    "source": "mv_itinerary_deposit_monthly",
    "timeWindow": "12m",
    "calculationVersion": "v2",
    "currency": "USD"
  }
}
```

### GET /api/v1/itinerary-revenue/conversion
**Response**
```json
{
  "data": {
    "timeline": [
      {
        "periodStart": "2026-03-01",
        "periodEnd": "2026-03-31",
        "quotedCount": 34,
        "confirmedCount": 40,
        "closeRatio": 0.5405,
        "projectedConfirmedCount": 18.4,
        "projectedCommissionIncomeExpected": 144500.0,
        "projectedCommissionIncomeBestCase": 162200.0,
        "projectedCommissionIncomeWorstCase": 128900.0
      }
    ],
    "lookbackCloseRatio": 0.5421
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-11",
    "source": "mv_itinerary_pipeline_stages",
    "timeWindow": "12m",
    "calculationVersion": "v2",
    "currency": "USD"
  }
}
```

### GET /api/v1/itinerary-revenue/channels
**Response**
```json
{
  "data": {
    "topConsortia": [
      {
        "label": "Virtuoso",
        "itineraryCount": 122,
        "paxCount": 346,
        "grossAmount": 1842000.0,
        "commissionIncomeAmount": 1487000.0,
        "marginAmount": 355000.0,
        "tradeCommissionAmount": 0
      }
    ],
    "topTradeAgencies": [
      {
        "label": "Swain Premier Agency",
        "itineraryCount": 74,
        "paxCount": 226,
        "grossAmount": 1160000.0,
        "commissionIncomeAmount": 932000.0,
        "marginAmount": 228000.0,
        "tradeCommissionAmount": 94000.0
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-11",
    "source": "mv_itinerary_consortia_monthly,mv_itinerary_trade_agency_monthly",
    "timeWindow": "12m",
    "calculationVersion": "v2",
    "currency": "USD"
  }
}
```

### GET /api/v1/itinerary-revenue/actuals-yoy
**Request (query params)**
```json
{
  "years_back": 2,
  "currency_code": "USD"
}
```

### GET /api/v1/itinerary-revenue/actuals-channels
**Request (query params)**
```json
{
  "years_back": 3,
  "actuals_year": 2026,
  "currency_code": "USD"
}
```

**Response**
```json
{
  "data": {
    "topConsortia": [
      {
        "label": "Virtuoso",
        "itineraryCount": 258,
        "paxCount": 791,
        "grossAmount": 10982975.0,
        "commissionIncomeAmount": 9827563.0,
        "marginAmount": 1155412.0,
        "tradeCommissionAmount": 0.0
      }
    ],
    "topTradeAgencies": [
      {
        "label": "Swain Premier Agency",
        "itineraryCount": 134,
        "paxCount": 418,
        "grossAmount": 5822100.0,
        "commissionIncomeAmount": 5012000.0,
        "marginAmount": 810100.0,
        "tradeCommissionAmount": 492800.0
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-11",
    "source": "mv_itinerary_consortia_actuals_monthly,mv_itinerary_trade_agency_actuals_monthly",
    "timeWindow": "2026",
    "calculationVersion": "v2",
    "currency": "USD"
  }
}
```

**Response**
```json
{
  "data": {
    "years": [2025, 2026],
    "timeline": [
      {
        "year": 2025,
        "month": 1,
        "monthLabel": "Jan",
        "itineraryCount": 41,
        "paxCount": 126,
        "grossAmount": 422000.0,
        "commissionIncomeAmount": 352000.0,
        "marginAmount": 70000.0,
        "tradeCommissionAmount": 22000.0,
        "marginPct": 0.1659,
        "avgGrossPerItinerary": 10292.68,
        "avgCommissionIncomePerItinerary": 8585.37,
        "avgGrossPerPax": 3349.21,
        "avgCommissionIncomePerPax": 2793.65,
        "avgNumberOfDays": 8.7,
        "avgNumberOfNights": 7.6,
        "grossShareOfYearPct": 0.0784,
        "itineraryShareOfYearPct": 0.0817
      }
    ],
    "yearSummaries": [
      {
        "year": 2025,
        "itineraryCount": 502,
        "paxCount": 1522,
        "grossAmount": 5382000.0,
        "commissionIncomeAmount": 4518000.0,
        "marginAmount": 864000.0,
        "tradeCommissionAmount": 292000.0,
        "marginPct": 0.1605,
        "avgGrossPerItinerary": 10720.12,
        "avgCommissionIncomePerItinerary": 9000.0,
        "avgGrossPerPax": 3536.14,
        "avgCommissionIncomePerPax": 2968.46,
        "avgNumberOfDays": 8.5,
        "avgNumberOfNights": 7.4
      }
    ]
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-11",
    "source": "mv_itinerary_revenue_monthly",
    "timeWindow": "2y",
    "calculationVersion": "v2",
    "currency": "USD"
  }
}
```

### GET /api/v1/fx/rates
**Request (query params)** `limit` (optional, default 50).  
**Scope**: returns rates only for supplier currencies `ZAR`, `USD`, `AUD`, `NZD`.

**Response**
```json
{
  "data": [
    {
      "id": "uuid",
      "currencyPair": "USD/ZAR",
      "rateTimestamp": "2026-02-10T12:00:00Z",
      "bidRate": 1.082,
      "askRate": 1.0824,
      "midRate": 1.0822,
      "source": "ecb",
      "createdAt": "2026-02-10T12:00:00Z"
    }
  ],
  "pagination": null,
  "meta": { "asOfDate": "2026-02-10", "source": "supabase", "timeWindow": "", "calculationVersion": "v1", "currency": null }
}
```

### GET /api/v1/fx/exposure
**Response**
```json
{
  "data": [
    {
      "currencyCode": "ZAR",
      "confirmed30d": 450000,
      "confirmed60d": 820000,
      "confirmed90d": 1200000,
      "estimated30d": 150000,
      "estimated60d": 280000,
      "estimated90d": 400000,
      "currentHoldings": 200000,
      "netExposure": 1400000
    }
  ],
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-10",
    "source": "mv_fx_exposure",
    "timeWindow": "",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### GET /api/v1/travel-consultants/leaderboard
**Request (query params)**
```json
{
  "period_type": "year",
  "domain": "travel",
  "year": 2026,
  "sort_by": "booked_revenue",
  "sort_order": "desc",
  "currency_code": "USD"
}
```

**Response**
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
        "commissionIncome": 102900.0,
        "marginAmount": 45300.0,
        "marginPct": 0.3057,
        "leadCount": 22,
        "closedWonCount": 11,
        "closedLostCount": 5,
        "conversionRate": 0.5,
        "closeRate": 0.6875,
        "avgSpeedToBookDays": 31.0,
        "spendToBook": null,
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
    "asOfDate": "2026-02-16",
    "source": "mv_travel_consultant_leaderboard_monthly,mv_travel_consultant_funnel_monthly",
    "timeWindow": "year",
    "calculationVersion": "v1",
    "currency": "USD"
  }
}
```

### GET /api/v1/travel-consultants/{employee_id}/profile
**Request (query params)**
```json
{
  "period_type": "year",
  "year": 2026,
  "yoy_mode": "same_period",
  "currency_code": "USD"
}
```

**Response**
```json
{
  "data": {
    "employee": {
      "employeeId": "e2f9f8d2-aaaa-bbbb-cccc-2a2a2a2a2a2a",
      "employeeExternalId": "005A0000001XyzQ",
      "firstName": "Alex",
      "lastName": "Taylor",
      "email": "alex@swain.com"
    },
    "sectionOrder": ["heroKpis", "trendStory", "funnelHealth", "operationalSnapshot", "forecastAndTarget", "compensationImpact", "signals", "insightCards"],
    "heroKpis": [
      {
        "key": "booked_revenue",
        "displayLabel": "Booked Revenue",
        "description": "Closed-won realized travel revenue for selected period.",
        "value": 148200.0,
        "trendDirection": "up",
        "trendStrength": "high",
        "isLaggingIndicator": false
      },
      {
        "key": "avg_gross_profit",
        "displayLabel": "Average Gross Profit",
        "description": "Average gross profit per closed-won itinerary in selected period.",
        "value": 10520.7,
        "trendDirection": "up",
        "trendStrength": "medium",
        "isLaggingIndicator": false
      }
    ],
    "threeYearPerformance": {
      "travelClosedFiles": {
        "key": "travel_closed_files",
        "title": "Closed Travel Revenue (Travel Date Basis)",
        "metricLabel": "revenue",
        "series": [
          { "year": 2024, "monthlyValues": [88570, 247234, 227654, 90234, 135443, 42842, 28295, 188423, 127098, 420311, 272940, 215321], "total": 2168365 },
          { "year": 2025, "monthlyValues": [452310, 307222, 407302, 368740, 139022, 245937, 13350, 167825, 389990, 387995, 258228, 78107], "total": 3212028 },
          { "year": 2026, "monthlyValues": [64946, 169592, 118005, 58075, 76135, 138738, 0, 263829, 31425, 108335, 109363, 0], "total": 1138443 }
        ],
        "variances": [
          { "label": "2025 vs 2024", "monthlyVariancePct": [4.1067, 0.2426, 0.788, 3.0862, 0.0264, 4.7403, -0.5278, -0.1083, 2.0685, -0.077, -0.0541, -0.6372], "totalVariancePct": 0.4813 },
          { "label": "2026 vs 2025", "monthlyVariancePct": [-0.8564, -0.4481, -0.7103, -0.8425, -0.4523, -0.4359, -1, 0.5721, -0.9194, -0.7208, -0.5765, -1], "totalVariancePct": -0.6456 }
        ]
      },
      "leadFunnel": {
        "key": "lead_funnel",
        "title": "Lead Funnel Revenue (Created/Booked Basis)",
        "metricLabel": "revenue",
        "series": [
          { "year": 2024, "monthlyValues": [364000, 402300, 318700, 294800, 271200, 310900, 281400, 336100, 355600, 372500, 348400, 327900], "total": 3983800 },
          { "year": 2025, "monthlyValues": [402100, 355900, 250700, 203400, 276500, 298700, 221900, 304100, 287400, 311800, 296300, 284000], "total": 3492800 },
          { "year": 2026, "monthlyValues": [214500, 188300, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0], "total": 402800 }
        ],
        "variances": [
          { "label": "2025 vs 2024", "monthlyVariancePct": [0.1047, -0.1153, -0.213, -0.3093, 0.0195, -0.0392, -0.2114, -0.0952, -0.1918, -0.163, -0.1495, -0.1339], "totalVariancePct": -0.1232 },
          { "label": "2026 vs 2025", "monthlyVariancePct": [-0.4666, -0.471, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1], "totalVariancePct": -0.8847 }
        ]
      }
    },
    "ytdVariancePct": -0.691,
    "trendStory": {
      "points": [],
      "currentTotal": 148200.0,
      "baselineTotal": 132000.0,
      "yoyDeltaPct": 0.1227
    },
    "funnelHealth": {
      "leadCount": 22,
      "closedWonCount": 11,
      "closedLostCount": 5,
      "conversionRate": 0.5,
      "closeRate": 0.6875,
      "avgSpeedToBookDays": 31.0
    },
    "forecastAndTarget": {
      "timeline": [],
      "summary": {
        "totalProjectedRevenueAmount": 1675000.0,
        "totalTargetRevenueAmount": 1712000.0,
        "totalGrowthGapPct": -0.0216
      }
    },
    "compensationImpact": {
      "salaryAnnualAmount": 95000.0,
      "salaryPeriodAmount": 7916.67,
      "commissionRate": 0.15,
      "estimatedCommissionAmount": 15435.0,
      "estimatedTotalPayAmount": 23351.67
    },
    "operationalSnapshot": {
      "currentTravelingFiles": [
        {
          "itineraryId": "b3a3f183-aaaa-bbbb-cccc-e4d16aa0160e",
          "itineraryNumber": "ITI-10293",
          "itineraryName": "South Africa Family Journey",
          "itineraryStatus": "Traveling",
          "primaryCountry": "South Africa",
          "travelStartDate": "2026-02-12",
          "travelEndDate": "2026-02-24",
          "grossAmount": 28450.0,
          "paxCount": 4
        }
      ],
      "topOpenItineraries": [
        {
          "itineraryId": "f81d4f3c-aaaa-bbbb-cccc-0f4c5a8bb62b",
          "itineraryNumber": "ITI-10388",
          "itineraryName": "Botswana Safari Draft",
          "itineraryStatus": "Proposal Sent",
          "primaryCountry": "Botswana",
          "travelStartDate": "2026-07-14",
          "travelEndDate": "2026-07-25",
          "grossAmount": 96200.0,
          "paxCount": 2
        }
      ]
    },
    "signals": [],
    "insightCards": [],
    "comparisonContext": {
      "currentPeriod": "2026-01-01..2026-12-31",
      "baselinePeriod": "2025-01-01..2025-12-31",
      "yoyMode": "same_period"
    }
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-16",
    "source": "mv_travel_consultant_profile_monthly,mv_travel_consultant_funnel_monthly,mv_travel_consultant_compensation_monthly",
    "timeWindow": "year",
    "calculationVersion": "v1",
    "currency": "USD"
  }
}
```

### GET /api/v1/travel-consultants/{employee_id}/forecast
**Request (query params)**
```json
{
  "horizon_months": 12,
  "currency_code": "USD"
}
```

## AI Insights Endpoints

### GET /api/v1/ai-insights/briefing
**Request (query params)**
```json
{
  "briefing_date": "2026-02-16"
}
```

**Response**
```json
{
  "data": {
    "id": "b8f3d2a1-1111-2222-3333-444455556666",
    "briefingDate": "2026-02-16",
    "title": "Daily operating brief",
    "summary": "Command center metrics reviewed with focus on cash, conversion, and deposits.",
    "highlights": [
      "Lead conversion 12m is 31.2%.",
      "Deposit coverage 6m average is 92.7%."
    ],
    "topActions": [
      "Prioritize advisor coaching for low-conversion segments.",
      "Escalate deposit follow-up on at-risk open itineraries."
    ],
    "confidence": 0.84,
    "evidence": {
      "summary": "Built from ai_context_command_center_v1 aggregate metrics.",
      "metrics": [
        {
          "key": "lead_conversion_rate_12m",
          "label": "Lead conversion (12m)",
          "currentValue": 0.312,
          "baselineValue": 0.35,
          "deltaPct": -0.038,
          "unit": "ratio"
        }
      ],
      "sourceViewNames": ["ai_context_command_center_v1"],
      "referencePeriod": "2026-02-16"
    },
    "generatedAt": "2026-02-16T15:00:00Z",
    "modelName": "gpt-5.2",
    "modelTier": "decision",
    "tokensUsed": 624,
    "latencyMs": 918,
    "runId": "f34f6a6c-d8f0-48d5-a512-a4f6f8abf0c2",
    "updatedAt": "2026-02-16T15:00:00Z"
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-16",
    "source": "ai_briefings_daily",
    "timeWindow": "daily",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### GET /api/v1/ai-insights/feed
**Request (query params)**
```json
{
  "domain": "travel_consultant",
  "status": "new",
  "severity": "high",
  "page": 1,
  "page_size": 25
}
```

**Response**
```json
{
  "data": {
    "items": [
      {
        "id": "c2ce9cb2-3fd2-4a81-a33a-65f6a8e4e126",
        "insightType": "coaching_signal",
        "domain": "travel_consultant",
        "severity": "high",
        "status": "new",
        "entityType": "employee",
        "entityId": "employee-123",
        "title": "Alex coaching opportunity",
        "summary": "Conversion is 27.1%, growth variance is -8.2%, and margin is 19.4%.",
        "recommendedAction": "Run pipeline review, tighten lead qualification, and set weekly close-plan actions.",
        "priority": 2,
        "confidence": 0.82,
        "evidence": {
          "summary": "Built from consultant context + benchmark rollups.",
          "metrics": [
            {
              "key": "conversion_rate",
              "label": "Conversion rate",
              "currentValue": 0.271,
              "baselineValue": 0.35,
              "deltaPct": -0.079,
              "unit": "ratio"
            }
          ],
          "sourceViewNames": ["ai_context_travel_consultant_v1", "ai_context_consultant_benchmarks_v1"],
          "referencePeriod": "2026-02-01"
        },
        "generatedAt": "2026-02-16T15:02:00Z",
        "modelName": "gpt-5.2",
        "modelTier": "decision",
        "tokensUsed": 412,
        "latencyMs": 732,
        "runId": "f34f6a6c-d8f0-48d5-a512-a4f6f8abf0c2",
        "createdAt": "2026-02-16T15:02:00Z",
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
    "asOfDate": "2026-02-16",
    "source": "ai_insight_events",
    "timeWindow": "rolling",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### GET /api/v1/ai-insights/recommendations
**Request (query params)**
```json
{
  "domain": "travel_consultant",
  "status": "new",
  "priority_min": 1,
  "priority_max": 3,
  "page": 1,
  "page_size": 25
}
```

**Response**
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
        "summary": "Conversion is 27.1%, growth variance is -8.2%, and margin is 19.4%.",
        "recommendedAction": "Run pipeline review, tighten lead qualification, and set weekly close-plan actions.",
        "priority": 2,
        "confidence": 0.82,
        "ownerUserId": null,
        "dueDate": null,
        "resolutionNote": null,
        "evidence": {
          "summary": "Built from consultant context + benchmark rollups.",
          "metrics": [],
          "sourceViewNames": ["ai_context_travel_consultant_v1", "ai_context_consultant_benchmarks_v1"],
          "referencePeriod": "2026-02-01"
        },
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
    "asOfDate": "2026-02-16",
    "source": "ai_recommendation_queue",
    "timeWindow": "rolling",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### PATCH /api/v1/ai-insights/recommendations/{id}
**Request**
```json
{
  "status": "in_progress",
  "owner_user_id": "e2f9f8d2-aaaa-bbbb-cccc-2a2a2a2a2a2a",
  "resolution_note": "Owner accepted and started consultant coaching plan."
}
```

**Response**
```json
{
  "data": {
    "id": "9c02bde2-d0b5-4025-b6b0-122f7f8ec0a9",
    "status": "in_progress",
    "ownerUserId": "e2f9f8d2-aaaa-bbbb-cccc-2a2a2a2a2a2a",
    "updatedAt": "2026-02-16T15:10:00Z"
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-16",
    "source": "ai_recommendation_queue",
    "timeWindow": "point_in_time",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

### GET /api/v1/ai-insights/history
**Request (query params)**
```json
{
  "domain": "travel_consultant",
  "status": "resolved",
  "date_from": "2026-01-01",
  "date_to": "2026-12-31",
  "page": 1,
  "page_size": 50
}
```

### GET /api/v1/ai-insights/entities/{entity_type}/{entity_id}
**Request**
```json
{
  "entity_type": "employee",
  "entity_id": "employee-123"
}
```

### POST /api/v1/ai-insights/run
**Request**
Header: `x-ai-run-token: <AI_MANUAL_RUN_TOKEN>`
```json
{}
```

**Response**
```json
{
  "data": {
    "runId": "f34f6a6c-d8f0-48d5-a512-a4f6f8abf0c2",
    "trigger": "manual_api",
    "status": "completed",
    "createdEvents": 5,
    "createdRecommendations": 4,
    "briefingGenerated": true,
    "consultantsEvaluated": 25
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-16",
    "source": "ai_context_*",
    "timeWindow": "manual",
    "calculationVersion": "v1",
    "currency": null
  }
}
```

**Response**
```json
{
  "data": {
    "employee": {
      "employeeId": "e2f9f8d2-aaaa-bbbb-cccc-2a2a2a2a2a2a",
      "employeeExternalId": "005A0000001XyzQ",
      "firstName": "Alex",
      "lastName": "Taylor",
      "email": "alex@swain.com"
    },
    "timeline": [
      {
        "periodStart": "2026-03-01",
        "periodEnd": "2026-03-31",
        "projectedRevenueAmount": 139200.0,
        "targetRevenueAmount": 143000.0,
        "growthGapPct": -0.0266
      }
    ],
    "summary": {
      "totalProjectedRevenueAmount": 1675000.0,
      "totalTargetRevenueAmount": 1712000.0,
      "totalGrowthGapPct": -0.0216
    }
  },
  "pagination": null,
  "meta": {
    "asOfDate": "2026-02-16",
    "source": "mv_travel_consultant_profile_monthly",
    "timeWindow": "12m",
    "calculationVersion": "v1",
    "currency": "USD"
  }
}
```
