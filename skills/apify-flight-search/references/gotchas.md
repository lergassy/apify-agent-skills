# Cost guardrails and gotchas

## What a run costs (pay per result, Free-plan list prices; higher Apify plans pay less)

| Actor | Price |
|-------|-------|
| `lergassy/google-flights-scraper` | $0.001 per flight, calendar day or destination row (no run fee) |
| `lergassy/flight-price-calendar` | $0.0002 per row until 24 Nov 2026, $0.001 from 25 Nov 2026 |
| `lergassy/kayak-flights-scraper` | $0.003 per run + $0.001 per fare; from 30 Nov 2026 $0.005 + $0.0015 |
| `lergassy/flights-mcp-server` | $0.03 per tool call |

- "Cheapest flight on a date" with `maxResults: 10` costs about 1 cent. The default `maxResults` is 100 — set it.
- A 60-day price calendar costs 6 cents; with `routes` it multiplies by the number of routes.
- `resolveBookingOptions`, `resolveReturnFlights` and multi-city legs make extra requests for the top rows; keep their limits (`maxBookingResolutions`, `maxReturnResolutions`) small.
- Notice, error, search-summary and run-summary rows are free.

## Things that surprise people

- Google `price` is the total for all passengers; KAYAK `price` is per person (`totalPrice` is the total).
- Round-trip Google prices are for both directions together.
- `market` (point of sale, e.g. `"ID"`) can change the price; `currency` only changes the display.
- Prices are a snapshot; for monitoring schedule the same input daily and compare.
