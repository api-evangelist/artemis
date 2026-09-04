---
name: artemis-neo-close-approach-scan
description: Scan NASA's Near Earth Object Web Service for asteroids by close-approach date, look one up by SPK-ID, or page the whole dataset.
api: Artemis NeoWs API
operations:
  - getNeoFeed
  - lookupNeo
  - browseNeo
generated: '2026-09-04'
method: generated
source: openapi/artemis-neows-api-openapi.yml, https://api.nasa.gov/
---

# Near-earth object close-approach scan (NASA NeoWs)

NeoWs serves the NASA JPL Center for Near-Earth Object Studies dataset. Objects are keyed by JPL
small-body SPK-ID.

## Before you start

- Base URL `https://api.nasa.gov`. Auth is the `api_key` query parameter.
- Read-only.

## Steps

1. **Scan a date window** with `getNeoFeed`:

   ```
   GET https://api.nasa.gov/neo/rest/v1/feed?start_date=2026-09-01&end_date=2026-09-08&api_key=DEMO_KEY
   ```

   The window is capped at **7 days**. `end_date` defaults to 7 days after `start_date`. A longer
   range is not paginated — it is rejected. Loop windows if you need a month.

2. The response groups objects under `near_earth_objects`, keyed by date. Each object carries an
   `id` (the SPK-ID) and a `close_approach_data` array.

3. **Look one up** with `lookupNeo` using that id:

   ```
   GET https://api.nasa.gov/neo/rest/v1/neo/3542519?api_key=DEMO_KEY
   ```

4. **Page the whole dataset** with `browseNeo` when you want the catalog rather than a window:

   ```
   GET https://api.nasa.gov/neo/rest/v1/neo/browse?page=0&size=20&api_key=DEMO_KEY
   ```

   The response carries `page.size`, `page.total_elements`, `page.total_pages`, `page.number` and
   `links.next`/`prev`/`self`. Follow `links.next` rather than incrementing `page` yourself.

   Note: NASA does not document `page` or `size` anywhere. They were confirmed against a live
   response on 2026-09-04. Treat them as observed behaviour, not a contract.

## Pagination is not consistent across this provider

`browseNeo` uses `page`/`size`. The NASA Image and Video Library uses `page`/`page_size`. APOD,
DONKI and EPIC use date windows and no cursor at all. Do not write one pager for api.nasa.gov.

## Error handling and budget

Same two envelopes as every api.nasa.gov call, and the same `X-RateLimit-Remaining` header. Paging
the full browse dataset means 31,000+ pages at the default size — that is far beyond any published
key allowance. Filter first.
