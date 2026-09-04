---
name: artemis-space-weather-watch
description: Check NASA DONKI for space weather events — solar flares, geomagnetic storms and radiation events — across a date window, so a lunar mission planning question has real data behind it.
api: Artemis DONKI API
operations:
  - getDonkiNotifications
generated: '2026-09-04'
method: generated
source: openapi/artemis-donki-api-openapi.yml, https://api.nasa.gov/
---

# Space weather watch (NASA DONKI)

DONKI is the Space Weather Database Of Notifications, Knowledge, Information. It is the feed that
matters for crewed lunar operations: solar particle events drive radiation exposure limits, and
geomagnetic conditions drive comms and navigation.

## Before you start

- Base URL `https://api.nasa.gov`.
- Auth is an `api_key` **query parameter**, not a header. Use `DEMO_KEY` for exploration (30
  requests/hour, 50/day, per IP) or a free registered key (1,000/hour).
- Read-only. Nothing you call here changes anything.

## Steps

1. Pick a window. DONKI is queried by date range, not by page cursor. Keep it tight — a wide window
   returns a large array with no pagination to fall back on.

2. Call `getDonkiNotifications`:

   ```
   GET https://api.nasa.gov/DONKI/notifications?startDate=2026-08-01&endDate=2026-08-31&type=all&api_key=DEMO_KEY
   ```

   `type` accepts `all`, or a specific event class. Omitting the dates defaults to a recent window.

3. Read the response as an array of notification records. Each carries a `messageID`, a
   `messageType`, an issue time and the message body. There is no cross-reference into any other
   NASA service — correlate by timestamp if you need to.

4. Check the budget before you loop. Every response carries `X-RateLimit-Limit` and
   `X-RateLimit-Remaining`. There is **no reset header and no `Retry-After`**, so if you exhaust the
   key you get HTTP 429 `OVER_RATE_LIMIT` and the only recovery is to wait out the rolling hour.

## Error handling

Two envelopes, on the same host:

- Gateway (auth, rate limit, routing): `{"error":{"code":"API_KEY_INVALID","message":"..."}}`
- Upstream service (validation): `{"code":400,"msg":"...","service_version":"v1"}`

Branch on the HTTP status and on `error.code`. NASA explicitly says message text is subject to
change, so never match on it. If `error` is absent, fall back to the top-level `code`/`msg`.

See `errors/artemis-problem-types.yml` for the full code table.

## What this skill will not do

There are no webhooks, no subscriptions and no push. DONKI is poll-only. If you need to watch for an
event, you schedule this call — and you budget for it, because polling on DEMO_KEY will exhaust 30
requests per hour quickly.
