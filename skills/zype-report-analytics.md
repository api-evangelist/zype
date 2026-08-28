---
name: zype-report-analytics
description: Pull engagement, revenue and platform-dynamics analytics from Zype, using the V3 surface rather than the superseded V2.
api: Zype Analytics API
base_url: https://analytics.zype.com
spec: openapi/zype-analytics-v3.json
operations:
  - plays-v3
  - viewers-v3
  - hours-watched-v3
  - view-time-v3
  - analytics-consumers-v3
  - metrics-v3
  - player-requests-v3
  - streamHoursV3
  - playoutHours
  - new-subscriptions-v3
  - subscriptionevents-v3
  - subscriptionrevenue-v3
  - new-transactions-v3
generated: '2026-08-28'
method: generated
source: openapi/zype-analytics-v3.json, openapi/zype-analytics.json
---

# Read Zype analytics

Analytics lives on its **own host** — `https://analytics.zype.com`, not `api.zype.com`.

## Use V3, not V2

Two analytics specs exist. `openapi/zype-analytics-v3.json` (13 operations, `/v3/*`) is
current. `openapi/zype-analytics.json` (9 operations, `/v2/*`) is the older surface — its
own docs title reads "List Stream Hours (V2)". **Zype has published no deprecation notice,
no `deprecated: true` flag and no sunset date for V2**, so nothing will warn you; prefer
V3 on new work and record the V2 dependency if you find one.

## The three families

- **Engagement** — `plays-v3`, `viewers-v3`, `hours-watched-v3`, `view-time-v3`.
- **Revenue** — `new-subscriptions-v3`, `subscriptionevents-v3`,
  `subscriptionrevenue-v3`, `new-transactions-v3`.
- **Platform dynamics** — `analytics-consumers-v3`, `metrics-v3`,
  `player-requests-v3`, `streamHoursV3`, `playoutHours`.

`playoutHours` has no V2 equivalent — it exists only because Playout was added after V2.

## Calling notes

- Same `api_key` query-parameter credential as the rest of the platform.
- Pagination is `page` / `per_page`, maximum 100. There is no cursor, so a long date range
  is a linear walk.
- **No rate limits are published** for analytics or anything else on Zype. That is not the
  same as no limits existing — back off on 5xx and watch https://status.zype.com.
- Errors are `{"message": "..."}` with no error code. A 404 here usually means the metric
  or date range is wrong, not that analytics is down.
