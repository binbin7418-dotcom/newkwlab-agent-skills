---
name: newkwlab-router
description: Reference and routing layer for the four NewKwLab APIs on RapidAPI - EmailCheck email validation, Page2MD URL to Markdown conversion, FoodFacts barcode nutrition lookup, and FlightScope live flight tracking - plus the RapidAPI hosted MCP gateway at mcp.rapidapi.com: auth headers, x-rapidapi-key and x-rapidapi-host, every host, every endpoint and request parameter, exact response field names, error codes and non-200 handling, MCP tool names and how to install the MCP server with claude mcp add, subscription plans and per-request billing. Use when the user asks about emailcheck-email-validation-api, page2md-url-to-clean-markdown-api, foodfacts-product-nutrition-barcodes-and-food-search, flightscope-live-flight-tracking-and-airport-data-api, newkwlab, a RapidAPI key, how to subscribe to an API, why a call returns "You are not subscribed to this API", a 401, 403, 404 or 429 from RapidAPI, what a call costs, curl or client code for any of the four APIs, the request body or response schema, or which NewKwLab API fits a job. Returns endpoint tables, runnable curl you can port to any language, and the exact JSON fields for every endpoint. Plumbing layer - for an actual job prefer the emailcheck, page2md, foodfacts, or flightscope skill.
---

# NewKwLab Router — API Reference and Routing

## Overview

The shared routing and reference layer for four independent APIs published by NewKwLab on RapidAPI:

| API | What it does | RapidAPI host | Listing |
|---|---|---|---|
| EmailCheck | Email validation: syntax, MX, disposable, role, typo fix, domain age, 0–100 score; batch list cleaning up to 100 addresses per call | `emailcheck-email-validation-api.p.rapidapi.com` | https://rapidapi.com/binbin7418/api/emailcheck-email-validation-api |
| Page2MD | Convert any web page to clean Markdown with title and metadata, single URL or batch | `page2md-url-to-clean-markdown-api.p.rapidapi.com` | https://rapidapi.com/binbin7418/api/page2md-url-to-clean-markdown-api |
| FoodFacts | Packaged-food lookup by barcode: Nutri-Score, NOVA, Eco-Score, ingredients, allergens, per-100g nutrients; keyword and category search | `foodfacts-product-nutrition-barcodes-and-food-search.p.rapidapi.com` | https://rapidapi.com/binbin7418/api/foodfacts-product-nutrition-barcodes-and-food-search |
| FlightScope | Live flight profile by flight number, batch lookup up to 100 flights per call, aircraft near a point, aircraft and airport profiles | `flightscope-live-flight-tracking-and-airport-data-api.p.rapidapi.com` | https://rapidapi.com/binbin7418/api/flightscope-live-flight-tracking-and-airport-data-api |

Every other skill in this repo delegates here for auth, hosts, parameter names and response field names. Prefer the MCP tools when a client has them configured; use REST otherwise.

## Authentication

All four APIs use the standard RapidAPI header pair. The key is always **the caller's own RapidAPI key** — one key works for every API the caller has subscribed to, and each call bills against that caller's subscription for that API:

```
x-rapidapi-key:  YOUR_RAPIDAPI_KEY
x-rapidapi-host: <the API's host from the table above>
```

There is no shared or demo key in this repo, by design. Never print, log, echo, or commit the key. Do not ask the user for a key unless a live call is required and none is configured.

A caller must **subscribe to an API before calling it** (a free BASIC tier exists on every listing). Calling an API you have not subscribed to returns the platform error `You are not subscribed to this API.` — that is a subscription state, not a bug: subscribe on the listing page, then retry.

## Access paths

### 1. RapidAPI hosted MCP gateway (preferred for agent clients)

RapidAPI auto-generates one MCP tool per endpoint for every listing and hosts them all at one URL. Each API is a separate server entry, selected by the `x-api-host` header:

```bash
# EmailCheck as an MCP server (Claude Code)
claude mcp add --transport http emailcheck https://mcp.rapidapi.com \
  --header "x-api-host: emailcheck-email-validation-api.p.rapidapi.com" \
  --header "x-api-key: YOUR_RAPIDAPI_KEY"

# FlightScope as an MCP server
claude mcp add --transport http flightscope https://mcp.rapidapi.com \
  --header "x-api-host: flightscope-live-flight-tracking-and-airport-data-api.p.rapidapi.com" \
  --header "x-api-key: YOUR_RAPIDAPI_KEY"
```

Verified 2026-11-17 against the live gateway: EmailCheck exposes 3 tools (`Validate_Email_via_GET`, `Validate_Email`, `Batch_validate`) and FlightScope exposes 8 tools (including `Batch_Live_Flight_Lookup`, `List_Live_Flights`, `Get_Airport_Profile`). Page2MD exposes 2 tools (`Convert_to_Markdown`, `Batch_Convert_Pages_to_Markdown`) and FoodFacts exposes 3 tools (`Get_Product`, `Search_Products`, `Browse_Category`) — all four hosts were copied from their listings' Playground MCP panels and verified the same day; never guess a host.

MCP calls count against the same subscription and quota as REST calls. A `tools/call` without an active subscription returns `You are not subscribed to this API.`

### 2. REST (any HTTP client)

Base URL is `https://<host>`; send both RapidAPI headers on every request. Runnable curl for each API lives in its use-case skill.

## Endpoint index

### EmailCheck — host `emailcheck-email-validation-api.p.rapidapi.com`

| Endpoint | Body / params | Returns |
|---|---|---|
| `GET /validate` | `?email=user@example.com` | one validation object |
| `POST /validate` | `{"email": "user@example.com"}` | one validation object |
| `POST /validate/batch` | `{"emails": [...]}` — up to 100 addresses | `{results: [...], summary: {...}}`, counts as ONE call |

Validation object fields: `email`, `valid`, `score` (0–100), `verdict` (`valid` / `risky` / `invalid`), `checks` (`syntax`, `domainExists`, `mxFound`, `disposable`, `roleAccount`, `freeProvider`), `mxRecords`, `suggestion`, `typoLikely`, `domainAgeDays`, `domainAgeSource`, `catchAllRisk`. Batch `summary` carries `total` and `{count, percent}` for `valid`, `risky`, `invalid`, `disposable`, `typoLikely`. Batch errors: `400 MISSING_EMAILS`, `400 EMPTY_BATCH`, `400 BATCH_TOO_LARGE` (>100 — split and retry).

### Page2MD — host `page2md-url-to-clean-markdown-api.p.rapidapi.com`

| Endpoint | Body | Returns |
|---|---|---|
| `POST /convert` | `{"url": "https://…"}` | `{markdown, title, url, byline, excerpt}` |
| `POST /convert/batch` | `{"urls": ["https://…", …]}` | array in request order; each item a convert result or an error object |

Errors carry a machine-readable code: `{error: {code, message}}` with HTTP 400 (bad request), 413 (too large), 415 (not HTML), 422 (unprocessable), 502/504 (fetch failed / timed out).

### FoodFacts — host `foodfacts-product-nutrition-barcodes-and-food-search.p.rapidapi.com`

| Endpoint | Params | Returns |
|---|---|---|
| `GET /product` | `?barcode=` (8–14 digit EAN/UPC) | `{product: {...}}` — name, brands, quantity, `nutriscoreGrade`, `novaGroup`, `ecoscoreGrade`, `ingredientsText`, `allergens`, categories, image, `nutrimentsPer100g` |
| `GET /search` | `?q=` plus optional `grade=` (a–e), `page_size=` (1–25), `page=` | `{count, page, pageSize, products: [...]}` |
| `GET /category` | `?name=` (e.g. `beverages`) plus optional `grade=`, paging | same shape as search |

Unknown barcode → `404 PRODUCT_NOT_FOUND` (an answer, not a failure). Bad barcode format → `400 INVALID_BARCODE`. When a product's upstream ingredient list has two or fewer entries, the product object carries a `note` warning that the source data may be incomplete — surface that note instead of presenting a short list as exhaustive. Data: Open Food Facts (ODbL); every response carries `_attribution`.

### FlightScope — host `flightscope-live-flight-tracking-and-airport-data-api.p.rapidapi.com`

| Endpoint | Params | Returns |
|---|---|---|
| `GET /live` | `?limit=` | currently airborne flights (feeds `List_Live_Flights`) |
| `GET /flight` | `?flight=TS655` (IATA or ICAO flight number) | `{flight, aircraft, live}` — airline, registration, type, latitude, longitude, `altitudeFt`, `groundSpeedKt`, `trackDeg`, `verticalRateFpm`, `squawk`, `ageSeconds` |
| `POST /flights/batch` | JSON array of up to 100 flight numbers | one result per flight plus a summary; counts as ONE call |
| `GET /flights/near` | `?lat=&lon=&radiusKm=` | every live aircraft within the radius |
| `GET /aircraft/{icao24}` | transponder hex in the path | aircraft profile (registration, type) |
| `GET /airport/{code}` | IATA or ICAO code in the path | airport profile (3,270 airports: name, codes, coordinates, elevation, municipality) |
| `GET /airports/search` | `?q=` name, city or code | matching airports |

A flight that is not broadcasting returns `404` with code `FLIGHT_NOT_LIVE` — the flight exists, it simply has no live signal right now. Live responses include `ageSeconds`; never present a position as live without it. Data: live positions from the adsb.lol community receiver network (ODbL 1.0), airports from OurAirports (public domain).

## Billing — say the cost before you spend it

Every HTTP request (REST or MCP) is one billed call against the caller's plan for that API, with two exceptions that exist to save money: EmailCheck `/validate/batch` (up to 100 addresses = 1 call) and FlightScope `/flights/batch` (up to 100 flights = 1 call). Prefer the batch endpoints whenever a job has more than a handful of items.

| API | BASIC (free) | PRO | ULTRA |
|---|---|---|---|
| EmailCheck | $0 — 100 calls/mo | $9/mo — 10,000 | $29/mo — 50,000 |
| Page2MD | $0 — 100 calls/mo | $15/mo — 5,000 | $49/mo — 25,000 |
| FoodFacts | $0 — 500 calls/mo | $5/mo — 10,000 | $14/mo — 50,000 |
| FlightScope | $0 — 50 calls/mo | $25/mo — 10,000 | $50/mo — 50,000 |

## Which skill for which job

- Validate an address, clean a mailing list, check signups → **emailcheck**
- Turn a web page or a reading list into Markdown → **page2md**
- Barcode nutrition, ingredients, allergen checks → **foodfacts**
- Where is flight X right now, planes near me, airport facts → **flightscope**
- Auth, hosts, schemas, errors, MCP setup → **this skill**
