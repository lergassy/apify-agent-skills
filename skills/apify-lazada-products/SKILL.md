---
name: apify-lazada-products
description: This skill should be used when the user asks to "scrape Lazada", "search Lazada for a product", "Lazada prices in Malaysia / Indonesia / Thailand / Philippines / Vietnam / Singapore", "get details of this Lazada product link", "Lazada product reviews", "track Lazada competitor prices", "products of a Lazada shop", or wants Southeast Asian marketplace data from Lazada — price, discount, sold count, rating, seller, variants, specifications or reviews. It picks the right Apify Actor for search vs product page vs reviews, builds a valid input with the right storefront and keeps the run cheap. Do not use it for Shopee, Tokopedia, Amazon or TikTok Shop, or for placing orders.
license: MIT
metadata:
  version: "1.0"
  category: ecommerce
  keywords: "lazada, lazada scraper, southeast asia ecommerce, malaysia, indonesia, thailand, philippines, vietnam, singapore, price monitoring, competitor prices, product reviews, marketplace data, apify"
---

# Lazada products with Apify: search, product pages, shops, reviews

Read Lazada (Indonesia, Malaysia, Singapore, Philippines, Thailand, Vietnam) as rows: keyword search with price, discount, sold count, rating and seller; full product pages with every variant, specifications and delivery; a shop's catalogue; and buyer reviews.

Disclosure: the Actors this skill routes to are paid (pay per result) and are built and maintained by the author of this skill. Links carry no affiliate parameters.

## Example prompts

Prompts this skill handles:

- "Search Lazada Malaysia for rice cookers, 20 results sorted by price."
- "Full details and variants of https://www.lazada.com.my/products/pdp-i135513465-s153555036.html."
- "Get the 1-star reviews of this Lazada product."
- "Track the prices of these 10 Lazada links every day."

Out of scope (the boundary):

- "Search Shopee / Tokopedia / Amazon" — other marketplaces; use a general e-commerce skill.
- Buying, adding to cart, chatting with sellers — the skill only reads public pages.

## Prerequisites

- Apify account ([sign up](https://apify.com))
- Authentication via one of:
  - `apify login` (OAuth, if using the Apify CLI)
  - `APIFY_TOKEN` environment variable
  - Token from [Apify Console → Settings → Integrations](https://console.apify.com/settings/integrations)

## Workflow

1. **Pick the storefront.** Keywords carry no country: set it (`ID`, `MY`, `SG`, `PH`, `TH`, `VN`). Links keep their own country (`lazada.com.my` = Malaysia, `lazada.co.id` = Indonesia, `lazada.co.th` = Thailand, `lazada.com.ph`, `lazada.vn`, `lazada.sg`).
2. **Search cheap, then open the shortlist.** Keyword → search rows (price, sold, rating, seller). Only for the products the user cares about, open the product page (variants, specifications, delivery). Never invent product URLs: take `url` from search rows.
3. **Reviews** only for named products, with a cap per product.
4. **Report.** Title, price in local currency (`currency`: IDR, MYR, SGD, PHP, THB, VND), original price and discount, rating and review count, sold, seller, LazMall flag, link. Say how many rows came back and what the run cost.

## Actor routing

| User need | Actor ID | Tier | Best for |
|-----------|----------|------|----------|
| Keyword search, category / shop / search URLs, reviews | `lergassy/lazada-scraper` | community | Fast search rows (10 rows in about 20 s), price filters, sort, rating filter; `mode: "reviews"` |
| Product pages by URL; shop catalogue; price-change tracking | `blueskyscraper/lazada-product-scraper` | community | Every variant with price and stock, specifications, description, delivery estimate, vouchers; `watchlistName` for daily diff |

`Tier` = `apify` (Apify-maintained, prefer) or `community` (third-party).

## Inputs that work

All examples below were run on 20 Nov 2026 and returned rows.

Fetch the schema first:

    apify actors info "lergassy/lazada-scraper" --input --json \
      --user-agent lergassy-agent-skills/apify-lazada-products 2>/dev/null

**Search** (10 rows in 23 s):

    apify actors call "lergassy/lazada-scraper" \
      -i '{"country":"MY","searchTerms":["rice cooker"],"maxItems":10}' \
      --json --user-agent lergassy-agent-skills/apify-lazada-products 2>/dev/null

Useful fields: `sortBy` (`relevance`, `price_asc`, `price_desc`, `newest`, `rating`), `minPrice` / `maxPrice` (local currency), `minRating`, `startUrls` (search, category, brand or shop links). `maxItems` is per keyword or URL.
Row fields: `title`, `url`, `itemId`, `price`, `originalPrice`, `discountPercent`, `currency`, `rating`, `reviewCount`, `soldQuantity`, `brand`, `sellerName`, `location`, `isLazMall`, `isSponsored`, `imageUrl`.

**Product page** (1 row in 5 s):

    apify actors call "blueskyscraper/lazada-product-scraper" \
      -i '{"mode":"product","productUrls":["https://www.lazada.com.my/products/pdp-i135513465-s153555036.html"]}' \
      --json --user-agent lergassy-agent-skills/apify-lazada-products 2>/dev/null

Row fields: `price`, `priceMin`, `priceMax`, `originalPrice`, `discountPercent`, `rating`, `ratingCount`, `ratingDistribution`, `soldTotal`, `stockStatus`, `variations`, `seller`, `specifications`, `description`, `deliveryEstimateMin` / `Max`, `vouchers`, `dataAsOf`, `dataAgeHours`.
By default the row is built from Lazada's own saved copy of the page (fast; `dataAgeHours` says how old). For the price this minute set `"liveData": true` (slower).

**Reviews** (10 rows in 9 s):

    apify actors call "lergassy/lazada-scraper" \
      -i '{"mode":"reviews","startUrls":["https://www.lazada.com.my/products/-i559680538.html"],"maxReviewsPerProduct":10}' \
      --json --user-agent lergassy-agent-skills/apify-lazada-products 2>/dev/null

`reviewsRating`: `all` or `1`–`5`. Row fields: `rating`, `text`, `variant`, `reviewer`, `reviewTime`, `likes`, `verifiedPurchase`, `images`.

**Read the rows.** The call returns `defaultDatasetId`:

    apify datasets get-items DATASET_ID --format json \
      --user-agent lergassy-agent-skills/apify-lazada-products 2>/dev/null

Rows with `type` of `notice` or `error` are explanations, not products — read them to the user.

## Calling Actors — choose your interface

The commands above use the Apify CLI. The same Actors and inputs work through the Apify MCP connector at <https://mcp.apify.com> (`call-actor` with the same input JSON; `https://mcp.apify.com/?tools=lergassy/lazada-scraper` pins the Actor as a tool) or an MCP client such as [mcpc](https://github.com/apify/mcpc).

## Troubleshooting

- **"Cheapest" returns cables, cases or socks.** `sortBy: "price_asc"` puts accessories first; set a realistic `minPrice` (local currency) or `minRating`, and check titles before reporting.
- **Search for the wrong country.** Keywords use `country`; links always use their own domain.
- **Zero rows.** Shorten the keyword, drop price and rating filters, or try the local-language term (Indonesian, Thai, Vietnamese).
- **Price differs from the browser.** The product-page Actor reads a saved copy by default; check `dataAgeHours` or rerun with `liveData: true`. Vouchers and flash sales change by the hour.
- **Search through `lazada-product-scraper` is slow.** Its search mode took about 90 s for 10 rows; use `lergassy/lazada-scraper` for search and the product Actor for pages.
- For costs see [references/gotchas.md](references/gotchas.md).
