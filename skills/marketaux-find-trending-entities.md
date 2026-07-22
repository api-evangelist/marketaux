---
name: Find trending entities in the news
description: >-
  Identify which stocks, cryptocurrencies, or industries are trending in
  financial news over any time frame, then pull the coverage behind them.
api: openapi/marketaux-openapi.yml
operations: [getTrendingEntities, getAllNews, getNewsSources]
generated: '2026-07-22'
method: generated
---

# Find trending entities in the news

Requires a **Standard plan or above** (`403 endpoint_access_restricted`
otherwise). Authenticate with the `api_token` query parameter.

## Steps

1. **Rank what's trending** — call `getTrendingEntities`
   (`GET /v1/entity/trending/aggregation`) with a time frame
   (`published_after=` 24h/7d ago, or `published_on=` for a specific day).
   Filter with `countries=us,ca`, `entity_types=equity,cryptocurrency`, or
   `industries=Technology`. Use `min_doc_count=10` to require a minimum
   article volume. Each row returns `key`, `total_documents`,
   `sentiment_avg`, and a trending `score`.
2. **Pull the coverage** — for top keys, call `getAllNews` with
   `symbols=<key>` and `filter_entities=true` over the same window to get the
   articles (with per-entity sentiment highlights) driving the trend.
3. **Curate sources** — optionally call `getNewsSources`
   (`GET /v1/news/sources`) and pass `domains=`/`exclude_domains=` to keep or
   drop specific outlets.

## Rules

- `group_by` defaults to `symbol`; switch to `industry` or `country` for
  sector/geo trends.
- Sort with `sort=total_documents|sentiment_avg` + `sort_order`.
- Respect `429`/`402` limits per `rate-limits/marketaux-rate-limits.yml`;
  errors follow `errors/marketaux-problem-types.yml`.
