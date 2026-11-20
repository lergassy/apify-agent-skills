# Cost guardrails and gotchas

## What a run costs (pay per result, Free-plan list prices; higher Apify plans pay less)

Ranking Actors (`tiktok-shop-top-products-scraper`, `-videos-scraper`, `-creators-scraper`, `-shops-live-scraper`):

| Event | Until 22 Nov 2026 | From 23 Nov 2026 |
|-------|-------------------|------------------|
| Run start | $0.005 | $0.025 |
| Product or video row | $0.003 | $0.003 |
| Creator, shop or live row | $0.004 | $0.004 |
| Row with `addAnalytics: true` | $0.04 | $0.072 |
| Each input that returns data is billed as at least 25 rows | — | yes |
| Creator found by @handle | — | $0.06 once (numeric creator IDs are free) |

- A 25-product list costs about $0.08 now and about $0.10 from 23 Nov 2026. With analytics: about $1.80 — ask first.
- `maxResultsPerSource` is per keyword / category / creator / shop: 4 keywords × 100 = 400 rows.

Other Actors:

| Actor | Price |
|-------|-------|
| `apimy/tiktok-shop-search-scraper` | $0.0016 per run + $0.012 per product (Free plan; $0.0072 on Gold and up) |
| `lergassy/tiktok-shop-reviews-scraper` | $0.005 per run + $0.0015 per review |

## Things that surprise people

- Rankings come from a licensed TikTok Shop analytics source, not from the shop page; numbers are estimates for the chosen window, not live counters.
- Keyword search (`apimy/tiktok-shop-search-scraper`) shows what a shopper sees on TikTok Shop US: price and discount right now. Rankings show what sold. Use both for "price vs sales".
- `month` overrides `period` and is meant for a finished calendar month; for the current month use `period: "last30Day"`.
