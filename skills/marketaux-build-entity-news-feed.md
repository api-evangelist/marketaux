---
name: Build an entity-filtered market news feed
description: >-
  Resolve ticker symbols with entity search, then build a concise financial
  news feed filtered to those entities with per-entity sentiment.
api: openapi/marketaux-openapi.yml
operations: [searchEntities, getAllNews, getSimilarNews, getNewsByUuid]
generated: '2026-07-22'
method: generated
---

# Build an entity-filtered market news feed

Authentication: every request takes your `api_token` as a GET query parameter
(free token at https://www.marketaux.com/register). URL-encode all parameters.
All dates are UTC; all text is UTF-8.

## Steps

1. **Resolve symbols** — call `searchEntities` (`GET /v1/entity/search`) with
   `search=<company name>` and optionally `countries=us`. Use the returned
   `symbol` values. To discover valid filter values, use `listEntityTypes` and
   `listEntityIndustries`.
2. **Fetch the feed** — call `getAllNews` (`GET /v1/news/all`) with
   `symbols=TSLA,AMZN` and `filter_entities=true` so only your queried
   entities are returned per article. Add `must_have_entities=true` to drop
   articles with no identified entities, `language=en` to constrain language,
   and `published_after=` for recency.
3. **Paginate** — use `page` + `limit`. `limit` maxes at your plan (3 free /
   20 basic / 50 standard / 100 pro). Stop when `meta.returned < meta.limit`;
   the result window is capped at 20,000.
4. **Expand a story** — for any article, store `uuid` and call `getNewsByUuid`
   later, or call `getSimilarNews` (`GET /v1/news/similar/{uuid}`) to cluster
   related coverage (`group_similar=true` is the default on feeds).

## Rules

- On `429` (`rate_limit_reached`) back off 60 seconds; watch
  `X-RateLimit-Limit`. On `402` (`usage_limit_reached`) the daily plan quota
  is exhausted — do not retry until the next day (`X-UsageLimit-Limit`).
- Errors arrive as `{"error": {"code", "message"}}` — see
  `errors/marketaux-problem-types.yml`.
- Sentiment fields range -1..+1; 0 is neutral.
