---
name: apify-tiktok-shop-research
description: This skill should be used when the user asks to "find best-selling TikTok Shop products", "what sells on TikTok Shop", "TikTok Shop revenue / GMV / units sold", "find TikTok Shop affiliate creators", "which videos sell this product", "top TikTok Shop shops or live streams", "scrape TikTok Shop reviews", "search TikTok Shop for a product", or wants product research, winning-product discovery, creator (affiliate) sourcing or competitor sales data for TikTok Shop in the US, UK, Southeast Asia, Japan, Mexico, Europe or Brazil. It routes to the right Apify Actor (top products, videos, creators, shops and lives, keyword search, reviews), builds a valid input and keeps the run cheap. Do not use it for ordinary TikTok videos without a shop link, for Amazon or Shopee, or for placing orders.
license: MIT
metadata:
  version: "1.0"
  category: ecommerce
  keywords: "tiktok shop, tiktok shop products, winning products, product research, gmv, units sold, tiktok shop creators, affiliates, tiktok shop videos, tiktok shop reviews, live commerce, apify"
---

# TikTok Shop research with Apify: products, videos, creators, shops, reviews

Find what sells on TikTok Shop and who sells it: product rankings by revenue and units sold, the videos and live streams that drive the sales, the creators (affiliates) behind them, top shops, keyword search results and buyer reviews — in 15 markets.

Disclosure: the Actors this skill routes to are paid (pay per result) and are built and maintained by the author of this skill. Links carry no affiliate parameters.

## Example prompts

Prompts this skill handles:

- "Top 25 air fryers on TikTok Shop US by revenue in the last 30 days."
- "Which TikTok videos sold the most skincare last week in the UK?"
- "Find 25 beauty creators on TikTok Shop US with revenue and follower counts."
- "Read the 1–2 star reviews of https://shop.tiktok.com/us/pdp/1732222955322249301."

Out of scope (the boundary):

- "Download this TikTok video" or "get comments of a normal TikTok post" — use a TikTok skill, not TikTok Shop.
- "Find this product on Amazon / Shopee / Temu" — other marketplaces.

## Prerequisites

- Apify account ([sign up](https://apify.com))
- Authentication via one of:
  - `apify login` (OAuth, if using the Apify CLI)
  - `APIFY_TOKEN` environment variable
  - Token from [Apify Console → Settings → Integrations](https://console.apify.com/settings/integrations)

## Workflow

1. **Classify the request.** "What sells / best sellers / winning products" → top products. "Which videos" → videos. "Creators / affiliates / influencers" → creators. "Shops / brands / live streams" → shops-and-lives. "What does a shopper see when searching X" (price, discount, rating) → keyword search. A product link + "reviews" → reviews.
2. **Pick the market** with `country` (US default; GB, ID, TH, VN, PH, MY, SG, JP, MX, DE, IT, FR, ES, BR) and the window with `period` (`last30Day`, `last7Day`, `lastDay`). `month` (`YYYY-MM`) is for a finished calendar month; for "this month" use `period: "last30Day"`.
3. **Start with 25 rows** (`maxResultsPerSource: 25`, the minimum on the ranking Actors). Leave `addAnalytics` off unless the user needs daily trend analytics — it costs about 24× a plain row.
4. **Chain only when asked.** A product row gives `productId` / `url`; feed it into videos (`productIds`), creators (`productIds`) or reviews (`productUrls`).
5. **Report.** Revenue is in USD for every market. Give rank, title, revenue, units sold, growth %, link; say how many rows came back and what the run cost.

## Actor routing

| User need | Actor ID | Tier | Best for |
|-----------|----------|------|----------|
| Best-selling products by keyword, category, creator or shop | `lergassy/tiktok-shop-top-products-scraper` | community | Revenue (GMV), units sold, average price, growth, revenue split video / live / shop window, commission rate |
| Videos that sell | `lergassy/tiktok-shop-videos-scraper` | community | Revenue per video, views, ad ROAS, likes, shares, AI-video flag |
| Selling creators (affiliates) | `lergassy/tiktok-shop-creators-scraper` | community | Creator revenue, units, followers, video vs live revenue, growth |
| Top shops or live streams | `lergassy/tiktok-shop-shops-live-scraper` | community | `listType: "shops"` or `"lives"`; affiliate / own-account / shop-window split |
| Keyword search as a shopper sees it (US) | `apimy/tiktok-shop-search-scraper` | community | Price, original price, discount, sold count, rating, review count, shop |
| Buyer reviews of a product (US, UK) | `lergassy/tiktok-shop-reviews-scraper` | community | Stars, text, date, photos, variant, verified purchase |

`Tier` = `apify` (Apify-maintained, prefer) or `community` (third-party).

## Inputs that work

All examples below were run on 20 Nov 2026 and returned rows in 5–25 seconds.

Fetch the schema first:

    apify actors info "lergassy/tiktok-shop-top-products-scraper" --input --json \
      --user-agent lergassy-agent-skills/apify-tiktok-shop-research 2>/dev/null

**Top products** (25 rows in 5 s):

    apify actors call "lergassy/tiktok-shop-top-products-scraper" \
      -i '{"keywords":["air fryer"],"country":"US","maxResultsPerSource":25}' \
      --json --user-agent lergassy-agent-skills/apify-tiktok-shop-research 2>/dev/null

Keywords should be 1–2 words; every word must appear in the product title. `sortBy`: `revenue` (default), `unitsSold`, `revenueGrowth`, `videoRevenue`, `commissionRate`, `price`, `newest`. Leave all sources empty for the overall top list of the market.

**Videos that sell** (25 rows in 7 s):

    apify actors call "lergassy/tiktok-shop-videos-scraper" \
      -i '{"keywords":["skincare"],"country":"US","maxResultsPerSource":25}' \
      --json --user-agent lergassy-agent-skills/apify-tiktok-shop-research 2>/dev/null

**Creators in a category** (25 rows in 8 s):

    apify actors call "lergassy/tiktok-shop-creators-scraper" \
      -i '{"categories":["Beauty & Personal Care"],"country":"US","maxResultsPerSource":25}' \
      --json --user-agent lergassy-agent-skills/apify-tiktok-shop-research 2>/dev/null

**Shopper search** (10 rows in 6–8 s, US only):

    apify actors call "apimy/tiktok-shop-search-scraper" \
      -i '{"keywords":["air fryer"],"country":"US","maxProducts":10}' \
      --json --user-agent lergassy-agent-skills/apify-tiktok-shop-research 2>/dev/null

**Reviews** (20 rows in 23 s):

    apify actors call "lergassy/tiktok-shop-reviews-scraper" \
      -i '{"productUrls":["https://shop.tiktok.com/us/pdp/1732222955322249301"],"maxReviewsPerProduct":20}' \
      --json --user-agent lergassy-agent-skills/apify-tiktok-shop-research 2>/dev/null

`sortBy` on the search Actor: `relevance` (default), `priceAsc`, `priceDesc`, `soldCount`, `sold7d`, `sold30d`, `revenue`, `rating`; filters `minPrice`, `maxPrice`, `minRating`, `minSoldCount` (skipped products are free). `rating: 0` means the product has no rating yet.

`reviewsFilter`: `all`, `1`–`5`, `media`, `verified`; `reviewsNewerThan`: a date or `7 days`.

**Row fields.** Products: `rank`, `productId`, `title`, `url`, `revenue`, `unitsSold`, `avgSellingPrice`, `revenueGrowthPercent`, `videoRevenue`, `liveRevenue`, `shopWindowRevenue`, `commissionRatePercent`. Videos: `url`, `creatorUsername`, `revenue`, `views`, `adsRoas`, `likes`, `isAd`, `isAiVideo`, `publishedAt`. Creators: `username`, `profileUrl`, `followers`, `revenue`, `unitsSold`, `videoRevenue`, `liveRevenue`. Search: `title`, `price`, `originalPrice`, `discountPercent`, `soldCount`, `rating`, `reviewCount`, `shopName`, `url`. Reviews: `reviewRating`, `reviewText`, `reviewedAt`, `variant`, `verifiedPurchase`, `reviewImages`.

**Read the rows.** The call returns `defaultDatasetId`:

    apify datasets get-items DATASET_ID --format json \
      --user-agent lergassy-agent-skills/apify-tiktok-shop-research 2>/dev/null

Rows with `type` of `notice` or `error` are explanations, not data — read them to the user.

## Calling Actors — choose your interface

The commands above use the Apify CLI. The same Actors and inputs work through the Apify MCP connector at <https://mcp.apify.com> (`call-actor` with the same input JSON; `https://mcp.apify.com/?tools=lergassy/tiktok-shop-top-products-scraper` pins the Actor as a tool) or an MCP client such as [mcpc](https://github.com/apify/mcpc).

## Troubleshooting

- **Zero rows for a keyword.** Use one or two words that appear in product titles ("air fryer", not "best air fryer for family"), or switch to `categories`.
- **Free Apify plan returns few or no rows.** On the Free plan the ranking Actors return only lists already requested by someone in the last 24 hours (up to 20 rows from each of the first 2 inputs, no analytics). A paid Apify plan removes the limit; the `notice` row says so.
- **Revenue looks low for a non-US market.** Revenue is converted to USD for every market; the window is `period` (30 days by default).
- **Reviews for a non-US/UK product.** The reviews Actor covers US and UK listings only.
- For costs see [references/gotchas.md](references/gotchas.md).
