---
name: price-check
description: Check what one product costs right now across Amazon, Walmart, Target, eBay, Google Shopping merchants, TikTok Shop and Lazada, including every Amazon seller's offer and who holds the featured offer, and flag the cheapest in-stock option. Use when the user gives a product name, model number, ASIN, UPC, TCIN or product link and asks "where is it cheapest", "price check", "compare prices for", "who sells this", "Amazon sellers for this ASIN", "Buy Box", "is this a good price", "price match", "MAP check", "track the price of", or "is it in stock at Walmart or Target".
---

# Price check

You check prices by running Imzlab Technologies' marketplace Actors on Apify through the **ecommerce-research** MCP server (some apps call it a connector). Match tools on the start of the Actor slug (`amazon-product`, `walmart`, `target`, `ebay`, `google-shopping`, `tiktok-shop`, `lazada`). The server also exposes `get-actor-run`, `get-dataset-items` and `get-key-value-store-record`.

The input and output of each Actor are in `../product-research/references/actor-contracts.md`, and the matching and comparison rules in `../product-research/references/product-schema.md`; read both before your first call in a conversation.

## Steps

1. **Identify the product exactly.** Best: identifiers or links (ASIN, Walmart item ID, TCIN, UPC, store links). Otherwise brand plus model number ("De'Longhi EC685"). A vague name ("a good espresso machine") is product research, not a price check: hand over to `product-research`. Note the country, condition (new, used, refurbished) and buyer ZIP or postal code if the user gives one.

2. **Plan the lookups** (one run per marketplace; several products fit in one run):
   - Amazon: `mode: "product"` with the ASIN for the listing, plus `mode: "offers"` with the same ASIN for every seller's price (`maxPages: 1` gives up to 10 offers). Without an ASIN, `mode: "search"` with brand and model, `maxResults` 5, then take the matching ASIN.
   - Walmart: `productIds` or `productUrls`, else `searchQueries` with brand and model, `maxResults` 5.
   - Target (US): `productIds` (TCIN) for store availability, else `searchQueries`, `maxResults` 5. Product pages take about a minute each.
   - Google Shopping: `searchQueries` with brand and model, `maxResults` 20, `sortBy: "price_low"`: one run shows many other merchants.
   - eBay for used or refurbished, with `condition`; TikTok Shop and Lazada only when the user asks or the market fits.
   - Location: `postalCode` (Amazon), `deliveryZip` (Walmart, Target), `location` (Google Shopping).

3. **Estimate the cost** with the price table. A typical check (Amazon product + offers, Walmart, Target search, Google Shopping, eBay) is about USD 0.38. If the estimate is USD 0.50 or less, run now and report the cost afterwards; above it, run only after the user agrees.

4. **Run, wait and fetch** as in `product-research`: start the runs together, call `get-actor-run` with `waitSecs: 45` until each run is terminal (up to 8 times), then `get-dataset-items` and the summary record. Never invent a price or a seller.

5. **Match** rows to the product with the rules in `product-schema.md`. Drop accessories, bundles and other models; count what you dropped. Keep possible matches separately and mark them.

## Answer format

1. Headline: "Cheapest new, in stock: USD X at <merchant> (<link>), checked <date and time>."
2. A table: Merchant, Price, Shipping, Total, Condition, In stock or delivery, Seller, Link. Sort by total. Mark sponsored rows and marketplace sellers (not the retailer itself).
3. Amazon block when offers were read: the featured offer (offer position 1) with its seller, the number of offers, the lowest new offer and the price spread.
4. One or two lines of judgement: how far the cheapest is below the typical price (median of the matched rows), whether a was-price discount is real by that measure, and any seller worth caution (no rating, ships from abroad).
5. One line: "Apify cost: about USD X for N rows in M runs, billed to your Apify account", from the price table. Never quote the usage, data-access or cost figures a run response or record carries.

For monitoring ("track this weekly"), give the same table and say the user can repeat the check; the `price-monitor` agent writes dated CSV snapshots and the changes since the last one.

## When things go wrong

- The ecommerce-research server is not connected or a tool call returns an authorization error: tell the user to sign in to Apify for the ecommerce-research server in their app's connector or MCP settings, then retry.
- Amazon offers come back empty: the listing has a single seller or offers are hidden for the storefront; use the product row's price and seller.
- A marketplace has no matching product: say "not found at <merchant>" rather than showing a different model.
- Prices differ by store or ZIP code: say which store or ZIP each price came from (Walmart `storeCity`, Target `storeName`).
