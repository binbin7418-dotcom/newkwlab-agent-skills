# NewKwLab Agent Skills

Agent skills for four NewKwLab APIs on RapidAPI — email validation, web-to-Markdown conversion, barcode nutrition lookup, and live flight tracking. Every skill teaches your agent the exact hosts, endpoints, parameters, response fields, and billing rules, so it can call the APIs correctly on the first try.

**You bring your own RapidAPI key.** Subscribe to an API (every one has a free tier), plug in your key, and every call bills against your own subscription. There is no shared key and no middleman server: agent → RapidAPI → the API.

## Install (60 seconds)

### Option A — MCP, one line per API

RapidAPI hosts an MCP gateway (`https://mcp.rapidapi.com`) that turns every listing's endpoints into tools. Point your client at it with two headers — the API's host and your key:

```bash
# EmailCheck — 3 tools: validate one, validate via GET, batch validate
claude mcp add --transport http emailcheck https://mcp.rapidapi.com \
  --header "x-api-host: emailcheck-email-validation-api.p.rapidapi.com" \
  --header "x-api-key: YOUR_RAPIDAPI_KEY"

# FlightScope — 8 tools: live flights, batch lookup, airport profiles, and more
claude mcp add --transport http flightscope https://mcp.rapidapi.com \
  --header "x-api-host: flightscope-live-flight-tracking-and-airport-data-api.p.rapidapi.com" \
  --header "x-api-key: YOUR_RAPIDAPI_KEY"

# Page2MD — 2 tools: convert one page, batch convert
claude mcp add --transport http page2md https://mcp.rapidapi.com \
  --header "x-api-host: page2md-url-to-clean-markdown-api.p.rapidapi.com" \
  --header "x-api-key: YOUR_RAPIDAPI_KEY"

# FoodFacts — 3 tools: product lookup, search, category browse
claude mcp add --transport http foodfacts https://mcp.rapidapi.com \
  --header "x-api-host: foodfacts-product-nutrition-barcodes-and-food-search.p.rapidapi.com" \
  --header "x-api-key: YOUR_RAPIDAPI_KEY"
```

All four hosts were copied from their listings' Playground MCP panels and verified with live `tools/list` calls on 2026-11-17 — hosts are verified, never guessed.

Works with Claude Code, Claude Desktop, Cursor, and any MCP client that accepts custom headers (via `mcp-remote` where needed).

### Option B — Skills

```bash
# Claude Code plugin
claude plugin marketplace add binbin7418-dotcom/newkwlab-agent-skills
claude plugin install newkwlab-agent-skills@newkwlab-agent-skills

# Or any agent with a skills directory
git clone https://github.com/binbin7418-dotcom/newkwlab-agent-skills.git
cp -r newkwlab-agent-skills/skills/* ~/.claude/skills/
```

### Option C — REST

Every skill contains runnable curl for its API. Auth is always the same pair:

```
x-rapidapi-key:  YOUR_RAPIDAPI_KEY
x-rapidapi-host: <that API's host>
```

## Get a key (free tiers, no card)

Subscribe on the listing, then copy your key from your RapidAPI dashboard. One key works for every API you subscribe to.

| API | Subscribe | Free tier |
|---|---|---|
| EmailCheck | https://rapidapi.com/binbin7418/api/emailcheck-email-validation-api | 100 calls/mo |
| Page2MD | https://rapidapi.com/binbin7418/api/page2md-url-to-clean-markdown-api | 100 calls/mo |
| FoodFacts | https://rapidapi.com/binbin7418/api/foodfacts-product-nutrition-barcodes-and-food-search | 500 calls/mo |
| FlightScope | https://rapidapi.com/binbin7418/api/flightscope-live-flight-tracking-and-airport-data-api | 50 calls/mo |

## The four APIs

### EmailCheck — validate one address, or clean a list of 100 in one call

Syntax, domain and MX checks, disposable / role-account / free-provider signals, typo suggestions, domain age from RDAP registry records, and a 0–100 score with a `valid` / `risky` / `invalid` verdict. The batch endpoint takes up to 100 addresses and returns per-address verdicts **plus a summary report** — and bills as a single call.

Real response, `GET /validate?email=support@allbirds.com`:

```json
{
  "email": "support@allbirds.com",
  "valid": true, "score": 90, "verdict": "valid",
  "checks": { "syntax": true, "domainExists": true, "mxFound": true,
              "disposable": false, "roleAccount": true, "freeProvider": false },
  "mxRecords": ["allbirds-com.mail.protection.outlook.com"],
  "suggestion": null, "typoLikely": false,
  "domainAgeDays": 9068, "domainAgeSource": "RDAP", "catchAllRisk": "possible"
}
```

Plans: BASIC $0 / 100 calls a month · PRO $9 / 10,000 · ULTRA $29 / 50,000.
Skill: [`skills/emailcheck`](skills/emailcheck/SKILL.md)

### Page2MD — any web page in, clean Markdown out

Send a URL, get the article as Markdown with `title`, `byline` and `excerpt` — built for RAG pipelines, reading lists, archiving and content migration. `/convert` for one page, `/convert/batch` for a list of URLs in a single call.

Real response (excerpt), `POST /convert` on `https://en.wikipedia.org/wiki/Markdown`:

```markdown
From Wikipedia, the free encyclopedia

| Markdown |  |
| --- | --- |
| Filename extensions | `.md`, `.markdown` |
| Internet media type | `text/markdown` |
| Developed by | John Gruber |
```

Plans: BASIC $0 — 100 calls/mo · PRO $15/mo — 5,000 · ULTRA $49/mo — 25,000. There is also a free no-key web tool for one-off pages: https://page2md-api.pages.dev/url-to-markdown
Skill: [`skills/page2md`](skills/page2md/SKILL.md)

### FoodFacts — barcode in, nutrition facts out

Look up any packaged food by EAN/UPC barcode: name, brands, Nutri-Score, NOVA group, Eco-Score, ingredients, allergens, and per-100g nutrients from Open Food Facts. Plus keyword search with a Nutri-Score grade filter, and category browsing.

Real response (trimmed), `GET /product?barcode=3017620422003`:

```json
{
  "product": {
    "barcode": "3017620422003", "name": "Nutella", "brands": "Ferrero, Nutella",
    "nutriscoreGrade": "e", "novaGroup": 3, "ecoscoreGrade": "f",
    "allergens": "milk, nuts, soybeans",
    "nutrimentsPer100g": { "energyKcal": 539, "fatG": 30.9, "sugarsG": 56.3,
                           "proteinsG": 6.3, "saltG": 0.107 }
  }
}
```

Plans: BASIC $0 / 500 calls a month · PRO $5 / 10,000 · ULTRA $14 / 50,000.
Skill: [`skills/foodfacts`](skills/foodfacts/SKILL.md)

### FlightScope — where is that flight, right now

Give it a flight number and get back a complete live profile in one call: the airline, the aircraft (registration, type), and its live position — latitude, longitude, altitude, ground speed, heading, vertical rate, squawk — from live ADS-B broadcasts. Batch up to 100 flight numbers in one call (counts as one request), list aircraft near any point, look up aircraft by ICAO24 hex, and get profiles for 3,270 airports.

Real response, `GET /flight?flight=TS655`:

```json
{
  "flight": { "iata": "TS655", "airline": { "name": "Air Transat" } },
  "aircraft": { "icao24": "c051d1", "registration": "C-GEZO", "typeCode": "A321" },
  "live": { "latitude": 40.889648, "longitude": -73.986694, "altitudeFt": 35025,
            "groundSpeedKt": 464.3, "squawk": "6166", "lastSeenSecondsAgo": 0 }
}
```

Plans: BASIC $0 / 50 calls a month · PRO $25 / 10,000 · ULTRA $50 / 50,000.
Skill: [`skills/flightscope`](skills/flightscope/SKILL.md)

## Repo layout

```
skills/
  newkwlab-router/   ← routing & reference: all hosts, auth, errors, MCP mapping, billing
  emailcheck/        ← validate addresses, clean lists
  page2md/           ← URLs to Markdown, single and batch
  foodfacts/         ← barcode nutrition, search, categories
  flightscope/       ← live flights, batch, near, aircraft, airports
```

Each skill is a `SKILL.md` (trigger-rich description, workflow, field tables, runnable curl) plus an `agents/openai.yaml` for OpenAI-style agent runtimes.

## Design rules

1. Prefer MCP tools when available; fall back to REST otherwise.
2. Say what a call costs before making it — every billed request is the user's money. Batch endpoints exist so a 100-item job costs 1 call, not 100.
3. A named error is an answer, not a failure: `FLIGHT_NOT_LIVE` means the plane is not broadcasting; `PRODUCT_NOT_FOUND` means the barcode is not in the database.
4. Never print, log, or commit an API key. Every example in this repo uses the placeholder `YOUR_RAPIDAPI_KEY`.
5. No invented numbers — the samples above are real responses, and the plan prices are the live listing prices.

## Status

All four listing hosts (EmailCheck, FlightScope, Page2MD, FoodFacts) were copied from their listings' Playground MCP panels and verified against the live gateway with `tools/list` calls on 2026-11-17; plan numbers were read off the live listings the same day.

## License

NewKwLab Skills License — see [LICENSE](LICENSE). Free to install and use (including commercially) for accessing NewKwLab APIs with your own RapidAPI key; modified or repackaged versions may not be pointed at other API providers. The APIs themselves are separate services on RapidAPI; data attributions (Open Food Facts ODbL, adsb.lol ODbL, OurAirports public domain) are carried in each API's responses.
