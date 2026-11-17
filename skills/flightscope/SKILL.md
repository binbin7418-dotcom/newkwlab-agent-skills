---
name: flightscope
description: Track live flights and look up aircraft and airports with the FlightScope API on RapidAPI - give it a flight number (IATA or ICAO, like TS655 or TSC655) and get back the airline, the aircraft registration and type, and exactly where the plane is right now: latitude, longitude, altitude in feet, ground speed in knots, heading, vertical rate, squawk, and how many seconds old the position is, straight from live ADS-B broadcasts. Batch up to 100 flight numbers in one call that counts as a single request, list every live aircraft near a point with lat, lon and radiusKm, look up an aircraft by its ICAO24 transponder hex, and get profiles for 3,270 airports by IATA or ICAO code plus airport search by name, city or code. Use when the user asks where a flight is, to track a flight, build a flight tracker or planes-near-me map, check flights near a city or airport, identify an aircraft overhead, look up an airport's coordinates, elevation or municipality, monitor a fleet, or asks what altitudeFt, groundSpeedKt, trackDeg, verticalRateFpm, ageSeconds, icao24, or FLIGHT_NOT_LIVE mean. A flight that is not broadcasting returns 404 FLIGHT_NOT_LIVE, which is an answer, not an error. For auth, hosts, and billing details use the newkwlab-router skill.
---

# FlightScope — Live Flight Tracking and Airport Data

## Overview

FlightScope gives an agent live aviation data without a schedule database or an ADS-B receiver. One call with a flight number returns a complete live profile: who operates the flight, which aircraft is flying it (registration and type), and where that aircraft is right now — latitude, longitude, altitude, ground speed, heading, vertical rate and squawk, from the live ADS-B broadcast, with `ageSeconds` stating how fresh the position is.

Four more shapes: batch lookup of up to 100 flight numbers in **one call that counts as one request**, a radius search for every aircraft near any point on Earth, aircraft lookup by ICAO24 hex, and profiles for 3,270 commercial airports. Built for flight tracker pages and widgets, "planes near me" maps, fleet and ops dashboards, spotter apps, and travel tools.

Full plumbing reference: the `newkwlab-router` skill.

## Access

- REST host: `flightscope-live-flight-tracking-and-airport-data-api.p.rapidapi.com`
- MCP (preferred in agent clients): server at `https://mcp.rapidapi.com` with headers `x-api-host: flightscope-live-flight-tracking-and-airport-data-api.p.rapidapi.com` and `x-api-key: YOUR_RAPIDAPI_KEY`. The gateway exposes 8 tools for this API, including `Batch_Live_Flight_Lookup`, `List_Live_Flights` and `Get_Airport_Profile` (verified 2026-11-17).
- Subscribe first (free BASIC tier: 50 calls/month): https://rapidapi.com/binbin7418/api/flightscope-live-flight-tracking-and-airport-data-api
- Auth on REST: headers `x-rapidapi-key: YOUR_RAPIDAPI_KEY` and `x-rapidapi-host: flightscope-live-flight-tracking-and-airport-data-api.p.rapidapi.com`. Never print or log the key.

## Endpoints

| Job | Endpoint | Input |
|---|---|---|
| List live flights | `GET /live` | `?limit=` |
| One flight's live profile | `GET /flight` | `?flight=TS655` (IATA or ICAO flight number) |
| Up to 100 flights at once | `POST /flights/batch` | JSON array of flight numbers — counts as ONE call |
| Aircraft near a point | `GET /flights/near` | `?lat=40.64&lon=-73.78&radiusKm=80` |
| Aircraft by transponder hex | `GET /aircraft/{icao24}` | hex in the path, e.g. `/aircraft/c051d1` |
| Airport profile | `GET /airport/{code}` | IATA or ICAO code, e.g. `/airport/CAN` |
| Find airports | `GET /airports/search` | `?q=` name, city or code |

`/flight` returns `{flight: {iata, callsign, airline: {name, iata, icao}}, aircraft: {icao24, registration, typeCode}, live: {latitude, longitude, altitudeFt, groundSpeedKt, trackDeg, verticalRateFpm, onGround, phase, squawk, lastSeenSecondsAgo}, ageSeconds}`.

## Workflow

1. **"Where is flight X?"** — one `/flight` call. Lead with the position in words ("35,025 ft over New Jersey, 464 kt, climbing"), then the aircraft and airline. Always carry the freshness: quote `ageSeconds` or `lastSeenSecondsAgo` when you present a position as current.
2. **Several flights?** Use `/flights/batch` — up to 100 flight numbers in one call that bills as a single request. State that saving before you make the call; never loop `/flight` 100 times.
3. **"What's flying near me / over this city?"** — `/flights/near` with the point's coordinates and a radius in km. If the user named a city or airport but not coordinates, get them from `/airport/{code}` or `/airports/search` first.
4. **Aircraft and airport facts** are static lookups: `/aircraft/{icao24}` for registration and type, `/airport/{code}` for coordinates, elevation and municipality.
5. **`404 FLIGHT_NOT_LIVE` is an answer, not a failure:** the flight is not broadcasting right now (on the ground with transponder off, outside receiver coverage, or not yet departed). Say exactly that; do not retry in a loop and do not conclude the flight does not exist.
6. **Phase honesty:** the `phase` field is derived from the ADS-B broadcast itself (altitude data), not the airline's official flight status — present it as such.

## Real response (verified live)

`GET /flight?flight=TS655`:

```json
{
  "flight": { "iata": "TS655", "callsign": "TSC655",
    "airline": { "name": "Air Transat", "iata": "TS", "icao": "TSC" } },
  "aircraft": { "icao24": "c051d1", "registration": "C-GEZO", "typeCode": "A321" },
  "live": { "latitude": 40.889648, "longitude": -73.986694,
    "altitudeFt": 35025, "groundSpeedKt": 464.3, "trackDeg": 19.76,
    "verticalRateFpm": 128, "onGround": false, "phase": "airborne",
    "squawk": "6166", "lastSeenSecondsAgo": 0 },
  "ageSeconds": 0
}
```

## Runnable curl

```bash
: "${RAPIDAPI_KEY:?set RAPIDAPI_KEY to your own RapidAPI key}"

# One flight, live
curl -s "https://flightscope-live-flight-tracking-and-airport-data-api.p.rapidapi.com/flight?flight=TS655" \
  -H "x-rapidapi-key: $RAPIDAPI_KEY" \
  -H "x-rapidapi-host: flightscope-live-flight-tracking-and-airport-data-api.p.rapidapi.com"

# Everything airborne within 80 km of JFK
curl -s "https://flightscope-live-flight-tracking-and-airport-data-api.p.rapidapi.com/flights/near?lat=40.64&lon=-73.78&radiusKm=80" \
  -H "x-rapidapi-key: $RAPIDAPI_KEY" \
  -H "x-rapidapi-host: flightscope-live-flight-tracking-and-airport-data-api.p.rapidapi.com"
```

## Billing and data

BASIC $0 — 50 calls/month · PRO $25/month — 10,000 calls · ULTRA $50/month — 50,000 calls. The batch endpoint makes the free tier go far: 100 flights = 1 call.

Data sources: live positions from the adsb.lol community receiver network (ODbL 1.0), airport data from OurAirports (public domain), airline codes from OpenFlights contributors (ODbL 1.0). Every response carries its sources and attribution.
