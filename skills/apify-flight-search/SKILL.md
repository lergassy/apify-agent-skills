---
name: apify-flight-search
description: This skill should be used when the user asks to "find flights from X to Y", "cheapest flight on a date", "Google Flights prices", "which day is cheapest to fly", "price calendar for a route", "where can I fly cheaply from X", "compare Google Flights and KAYAK", "monitor a fare", or wants flight itineraries, fares, airlines, stops, durations, booking options or cheapest dates and destinations as data. It routes to the right Apify Actor (Google Flights search, date calendar, explore-anywhere, KAYAK fares per booking site), builds a valid input with IATA codes and future dates, and keeps the run cheap. Do not use it for hotels, car rental, train or bus tickets, or for booking a ticket.
license: MIT
metadata:
  version: "1.0"
  category: travel
  keywords: "google flights, flight prices, cheap flights, airfare, flight search, price calendar, cheapest dates, explore destinations, kayak, momondo, travel data, apify"
---

# Flight search with Apify: Google Flights, cheapest dates and destinations, KAYAK

Get live flight data as rows: every itinerary Google Flights shows for a route and date (price, airline, flight numbers, stops, layovers, duration, emissions, bags), the cheapest day to fly over a month or a year, the cheapest destinations from an airport, and KAYAK fares per booking site.

Disclosure: the Actors this skill routes to are paid (pay per result) and are built and maintained by the author of this skill. Links carry no affiliate parameters.

## Example prompts

Prompts this skill handles:

- "Cheapest flights from JFK to London on 20 January, nonstop only."
- "Which day in December is cheapest to fly Bali (DPS) → Singapore?"
- "Where can I fly cheaply from Singapore on 15 January?"
- "Compare the Google Flights and KAYAK price for JFK → LHR on 20 January."

Out of scope (the boundary):

- "Book this ticket for me" — the skill returns prices and booking links; it does not buy tickets.
- Hotels, trains, buses, car rental — use a travel or hotel skill.

## Prerequisites

- Apify account ([sign up](https://apify.com))
- Authentication via one of:
  - `apify login` (OAuth, if using the Apify CLI)
  - `APIFY_TOKEN` environment variable
  - Token from [Apify Console → Settings → Integrations](https://console.apify.com/settings/integrations)

## Workflow

1. **Normalise the trip.** Airports as 3-letter IATA codes (`JFK`, `LHR`, `DPS`; city codes like `LON` also work). Dates as `YYYY-MM-DD`, today or later and at most about 330 days ahead. No return date → one-way. Passengers: `adults`, `children`, `infantsOnLap`, `infantsInSeat`.
2. **Pick the job.**
   - Fixed route and date → Google Flights search.
   - "Which day is cheapest" → the same Actor with `calendarOnly: true` and `calendarDays` (one row per day).
   - "Where can I go" → the same Actor with `exploreAnywhere: true` (one row per destination).
   - "Which booking site is cheapest" or Google has no price → KAYAK.
3. **Keep it small.** `maxResults: 10–20` with `sortBy: "price"` answers "cheapest"; every flight row is billed. Put filters in the input (`maxStops`, `airlines`, `maxPrice`, departure time windows) so unwanted rows are never billed.
4. **Report.** Price and currency, airline, flight numbers, departure → arrival times, stops and layovers, duration, and `googleFlightsUrl` / `bookingUrl` for booking. Say prices are a snapshot and change.

## Actor routing

| User need | Actor ID | Tier | Best for |
|-----------|----------|------|----------|
| Flights for a route and date; also price calendar, explore-anywhere, multi-city, several routes | `lergassy/google-flights-scraper` | community | Full Google Flights list (100–150 itineraries), price level vs typical, booking options, return flights |
| Cheapest day only, many routes at once | `lergassy/flight-price-calendar` | community | Calendar preset of the same engine (`calendarOnly` on by default, 60 days) |
| Fares per booking site (airline vs agencies) | `lergassy/kayak-flights-scraper` | community | KAYAK or Momondo, one row per provider with `allProviders: true` |
| Flight tools inside an MCP client | `lergassy/flights-mcp-server` | community | Four tools (search, compare Google vs KAYAK, cheapest dates, cheapest destinations) at `https://lergassy--flights-mcp-server.apify.actor/mcp` |

`Tier` = `apify` (Apify-maintained, prefer) or `community` (third-party).

## Inputs that work

All examples below were run on 20 Nov 2026 and returned rows in 4–48 seconds. Replace the dates with real future dates.

Fetch the schema first:

    apify actors info "lergassy/google-flights-scraper" --input --json \
      --user-agent lergassy-agent-skills/apify-flight-search 2>/dev/null

**Route and date, cheapest first** (10 flights in 6 s):

    apify actors call "lergassy/google-flights-scraper" \
      -i '{"origin":"JFK","destination":"LHR","departureDate":"2027-01-20","maxResults":10,"sortBy":"price"}' \
      --json --user-agent lergassy-agent-skills/apify-flight-search 2>/dev/null

Round trip: add `"returnDate":"YYYY-MM-DD"` (price is the total for both directions). Filters: `maxStops` (`"0"` nonstop, `"1"`, `"2"`, `"any"` — a string), `cabinClass`, `airlines` / `excludeAirlines` (IATA codes), `maxPrice`, `maxDurationMinutes`, `departureTimeFrom` / `departureTimeTo` (`"06:00"`), `currency`.

**Cheapest day over 30 days** (30 day rows in 4 s):

    apify actors call "lergassy/google-flights-scraper" \
      -i '{"origin":"DPS","destination":"SIN","departureDate":"2026-12-10","calendarDays":30,"calendarOnly":true}' \
      --json --user-agent lergassy-agent-skills/apify-flight-search 2>/dev/null

Rows have `type: "calendar_day"`, `date`, `price`, `isCheapest`. `lergassy/flight-price-calendar` takes the same input with `calendarOnly` already on.

**Cheapest destinations from an airport** (10 rows in 19 s):

    apify actors call "lergassy/google-flights-scraper" \
      -i '{"origin":"SIN","exploreAnywhere":true,"departureDate":"2027-01-15","maxDestinations":10}' \
      --json --user-agent lergassy-agent-skills/apify-flight-search 2>/dev/null

Rows have `type: "destination"`, `destination`, `city`, `country`, `price`, `airline`, `stops`, `durationMinutes`.

**KAYAK fares** (10 rows in 48 s):

    apify actors call "lergassy/kayak-flights-scraper" \
      -i '{"origin":"JFK","destination":"LHR","departureDate":"2027-01-20","maxResults":10}' \
      --json --user-agent lergassy-agent-skills/apify-flight-search 2>/dev/null

`allProviders: true` gives one row per booking site; `site: "momondo"` reads Momondo. KAYAK `price` is per person, `totalPrice` is for all travellers; Google `price` is for all travellers.

**Row fields (Google flight).** `price`, `currency`, `airline`, `airlineCodes`, `flightNumbers`, `stops`, `layovers`, `departAirport`, `departAt`, `arriveAirport`, `arriveAt`, `totalDurationMinutes`, `legs`, `emissionsKg`, `checkedBagsIncluded`, `isCheapest`, `isFastest`, `priceLevel`, `typicalPriceLow`, `typicalPriceHigh`, `googleFlightsUrl`.

**Read the rows.** The call returns `defaultDatasetId`:

    apify datasets get-items DATASET_ID --format json \
      --user-agent lergassy-agent-skills/apify-flight-search 2>/dev/null

Rows with `type` of `notice`, `error`, `search` or `summary` are explanations and totals, not flights.

## Calling Actors — choose your interface

The commands above use the Apify CLI. The same Actors and inputs work through the Apify MCP connector at <https://mcp.apify.com> (`call-actor` with the same input JSON; `https://mcp.apify.com/?tools=lergassy/google-flights-scraper` pins the Actor as a tool). For a ready flight toolset, point an MCP client at `https://lergassy--flights-mcp-server.apify.actor/mcp` with the header `Authorization: Bearer <APIFY_TOKEN>`.

## Troubleshooting

- **Date rejected.** Dates must be today or later and within about 330 days; airlines load schedules about 11 months ahead.
- **Few itineraries or no price on some flights.** Keep the default residential proxy (`dataSource: "auto"`); some airlines keep fares off Google (`fillMissingPricesFromKayak: true` fills them from KAYAK).
- **Google returns nothing for a route.** `kayakWhenGoogleEmpty` (on by default) runs KAYAK for that route.
- **Explore returns other dates.** With `exploreSuggestedDates: true` (default) Google suggests nearby dates when your exact date has no offers; set it to `false` to forbid.
- **`maxStops` ignored.** It is a string: `"0"`, not `0`.
- For costs see [references/gotchas.md](references/gotchas.md).
