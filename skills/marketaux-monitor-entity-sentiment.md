---
name: Monitor entity sentiment over time
description: >-
  Chart how entities perform in the news — document counts and average
  sentiment as a time series or single-window aggregation.
api: openapi/marketaux-openapi.yml
operations: [getEntityStatsIntraday, getEntityStatsAggregation, getAllNews]
generated: '2026-07-22'
method: generated
---

# Monitor entity sentiment over time

Requires a **Standard plan or above** — the stats endpoints return `403`
(`endpoint_access_restricted`) on Free/Basic. Authenticate with the
`api_token` query parameter.

## Steps

1. **Time series** — call `getEntityStatsIntraday`
   (`GET /v1/entity/stats/intraday`) with `symbols=TSLA,AMZN,MSFT`,
   `interval=day`, and a `published_after`/`published_before` window. Each
   bucket returns `key`, `total_documents`, and `sentiment_avg`. Interval
   windows are capped (minute=7 days, hour=1 month, day=3 years, week/month=5
   years, quarter/year=10 years) — exceeding the cap silently clamps, no error.
2. **Single-window ranking** — call `getEntityStatsAggregation`
   (`GET /v1/entity/stats/aggregation`) with `sort=sentiment_avg` (or
   `total_documents`) and `sort_order=desc|asc` to rank best/worst performers.
   Use `group_by=symbol|exchange|industry|country` to change the rollup and
   `min_doc_count` to drop thin signals.
3. **Drill into the why** — for an interesting spike, call `getAllNews` with
   the same `symbols` and date window plus `sentiment_gte`/`sentiment_lte` to
   read the driving articles.

## Rules

- Entities returned per stat request are plan-capped (20 Standard / 100 Pro).
- `sentiment_avg` may be `null` when a bucket has zero documents.
- Respect `429`/`402` limits per `rate-limits/marketaux-rate-limits.yml`.
