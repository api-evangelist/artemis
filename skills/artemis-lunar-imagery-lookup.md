---
name: artemis-lunar-imagery-lookup
description: Find Artemis and lunar imagery in the NASA Image and Video Library, pull the Astronomy Picture of the Day, and fetch lunar map tiles from Moon Trek's OGC WMTS service.
api: Artemis Images API
operations:
  - searchImages
  - getApod
generated: '2026-09-04'
method: generated
source: openapi/artemis-images-api-openapi.yml, openapi/artemis-apod-api-openapi.yml, ogc/artemis-moon-trek-lro-wac-wmts-capabilities.xml
---

# Lunar and Artemis imagery lookup

Three different surfaces serve imagery here and they do not share a host, an auth scheme or a
pagination style.

## 1. NASA Image and Video Library — `searchImages`

Different host, and **no API key at all**:

```
GET https://images-api.nasa.gov/search?q=artemis&media_type=image&page=1&page_size=25
```

- The response is a `collection` object with `metadata.total_hits`, `links` (rel `prev`/`next`) and
  `items`. Each item carries a `nasa_id`.
- Follow the `next` link rather than incrementing `page`.
- With a `nasa_id`, fetch `/asset/{nasa_id}` for the media manifest, `/metadata/{nasa_id}` for the
  metadata location, `/captions/{nasa_id}` for video captions.
- `page`/`page_size` were confirmed on a live response 2026-09-04; the api.nasa.gov catalog page does
  not document them.

## 2. Astronomy Picture of the Day — `getApod`

```
GET https://api.nasa.gov/planetary/apod?date=2026-08-14&api_key=DEMO_KEY
```

- `date`, or `start_date`+`end_date` for a range, or `count` for a random selection. Coverage begins
  1995-06-16; a date outside the covered range returns HTTP 400.
- A malformed date returns the **upstream** envelope, not the gateway one:
  `{"code":400,"msg":"time data 'notadate' does not match format '%Y-%m-%d'","service_version":"v1"}`
- `copyright` appears only when the image is NOT public domain. Its absence is your reuse signal.

## 3. Lunar map tiles — Moon Trek, OGC WMTS

This is not a REST API. It is an OGC Web Map Tile Service, and the contract is its Capabilities
document — saved verbatim in this repo at
`ogc/artemis-moon-trek-lro-wac-wmts-capabilities.xml`, fetched from:

```
https://trek.nasa.gov/tiles/Moon/EQ/LRO_WAC_Mosaic_Global_303ppd_v02/1.0.0/WMTSCapabilities.xml
```

- The document declares `ows:ServiceType` `OGC WMTS`, `ows:ServiceTypeVersion` `1.0.0`.
- Do not guess tile URLs. Parse the `ResourceURL` template out of the Capabilities document — it
  tells you the Style identifier, the TileMatrixSet, the zoom levels and, critically, whether the
  tiles are `.png` or `.jpg` (the LRO WAC mosaic is `image/jpeg`; other products differ).
- Template shape:
  `https://trek.nasa.gov/tiles/Moon/EQ/{layer}/1.0.0/{Style}/{TileMatrixSet}/{TileMatrix}/{TileRow}/{TileCol}.jpg`
- Zoom level 0 for a global product is 2 columns by 1 row. A missing tile means no data coverage, not
  an error — check the coverage bounding box first.
- No API key. Any OGC-capable GIS or map client consumes this directly.

## Budget

Only the api.nasa.gov calls consume key quota. The Image Library and the Trek WMTS tiles are
unauthenticated. Check `X-RateLimit-Remaining` on the APOD calls; there is no `Retry-After` if you
run out.
