# Cost guardrails and gotchas

## What a run costs (pay per result, Free-plan list prices; higher Apify plans pay less)

| Actor | Event | Price |
|-------|-------|-------|
| `lergassy/lazada-scraper` | Search row | $0.002 |
| | Search row with `enrichProducts` (full product record) | $0.00319 |
| | Specifications | $0.0077 |
| | Review | $0.001 |
| `blueskyscraper/lazada-product-scraper` | Run start | $0.005 |
| | Product page or product card | $0.0032 |
| | Review | $0.0012 |

- 20 search rows cost about 4 cents; 20 product pages about 7 cents; 100 reviews about 10 cents.
- `maxItems` is per keyword / URL on the search Actor; three keywords × 40 = 120 billed rows.
- Notice and error rows are free.

## Things that surprise people

- Prices are in the storefront's own currency; there is no USD conversion.
- `soldQuantity` / `soldTotal` is Lazada's own rounded figure ("1.2k sold").
- `isSponsored` rows are ads in the search; drop them for an organic ranking.
- For daily price tracking use `watchlistName` on `blueskyscraper/lazada-product-scraper`: each run marks rows with `changeType` and the previous price.
