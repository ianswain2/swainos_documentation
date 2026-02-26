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
