---
name: product-research
description: Research products for a keyword or category across Amazon, Walmart, Target, eBay, Google Shopping, TikTok Shop and Lazada, or pull Amazon Best Sellers, and compare prices, ratings, reviews, sales signals and sellers in one table. Use when the user asks "find products for", "what sells on Amazon for", "Amazon best sellers in", "top products in this category", "product research", "winning products", "compare marketplaces for", "what does Walmart or Target carry for", "eBay listings for", "TikTok Shop products for", "Lazada products", "price range for", "market research for a product idea", or wants a product list with prices from online stores.
---

# Product research

You research products by running Imzlab Technologies' marketplace Actors on Apify through the **ecommerce-research** MCP server (some apps call it a connector). Each Actor is a tool on that server whose name contains the Actor slug, for example `muhammadafzal--amazon-product-offers-bestsellers-scraper`; depending on the app, the tool name may carry a server prefix, use underscores in place of hyphens, or be shortened with a suffix, so match on the start of the slug (`amazon-product`, `walmart`, `target`, `ebay`, `google-shopping`, `shopify-store`, `tiktok-shop`, `lazada`, `aliexpress`, `alibaba`). The server also exposes `get-actor-run`, `get-dataset-items` and `get-key-value-store-record`, which you need to collect results.

Read `references/actor-contracts.md` for the exact input and output of every Actor before your first call in a conversation, and `references/product-schema.md` for normalized columns, matching rules and CSV export.

## Steps

1. **Pin down the request.** You need a keyword or category, the marketplaces, the country, and a count. Infer what you can: default count 20 per marketplace; default marketplaces for the US are Amazon, Walmart and Google Shopping (Google Shopping shows many other merchants at once); add Target, eBay or TikTok Shop when the user names them or the product fits (eBay for used and collectibles, TikTok Shop for impulse and trending items). Outside the US: Amazon on the local storefront and Google Shopping in that country; Lazada for Singapore, Malaysia, Thailand, the Philippines, Indonesia and Vietnam. Ask only when the keyword is missing.

2. **Route** with the contracts file:
   - keyword: Amazon `mode: "search"`, Walmart and Target `searchQueries`, eBay `searchQueries`, Google Shopping `searchQueries`, TikTok Shop `mode: "search"`, Lazada `searchQueries` with `country`;
   - "best sellers" or "top 100" in a category: Amazon `mode: "bestsellers"` with the category ID or Best Sellers link (ask for the link if you cannot name the category ID with confidence);
   - price band: Amazon `minPrice`/`maxPrice`, Google Shopping `minPrice`/`maxPrice` (no sort with a price filter);
   - a buyer location: Amazon `postalCode`, Walmart and Target `deliveryZip`, Google Shopping `location`.

3. **Estimate the cost** with the price table in the contracts file. If the estimate is USD 0.50 or less, run now and report the cost afterwards. Above USD 0.50, show the estimate as one line ("about USD 0.67 for 20 products on each of Amazon, Walmart, Target, eBay, Google Shopping and TikTok Shop, billed to your Apify account") and run only after the user agrees.

4. **Run the Actors** together, up to 4 at a time, with the input shapes from the contract file and the result cap set to the count. Never add fields the contract does not list.

5. **Wait for every run to finish.** A tool response with status `RUNNING` or `READY` is not a result. Call `get-actor-run` with the `runId` and `waitSecs: 45`, repeat until the status is `SUCCEEDED`, `FAILED`, `ABORTED` or `TIMED-OUT`, up to 8 times. Never report a run as done while it is still running, and never invent a product, price or rating.

6. **Fetch the results**, whatever the run status was: `get-dataset-items` with the run's default `datasetId` and `limit` equal to the cap, and `get-key-value-store-record` with the record named in the contract (`OUTPUT` for most, `SUMMARY` for Shopify). The record's per-input status explains empty or short results.

7. **Normalize** with `references/product-schema.md`, mark sponsored rows, and analyze: price range and median per marketplace, the best-rated products with enough reviews (at least 50), demand signals (Amazon `salesVolume` and best-seller rank, eBay and TikTok Shop `soldCount`, Lazada `soldCount`), brands that appear on several marketplaces, and gaps (a product type that is common on one marketplace and missing on another).

## Answer format

1. One line per marketplace: products found, price range and median, currency.
2. One table, up to 25 rows in chat: Marketplace, Product, Brand, Price (and was-price), Rating (reviews), Sales signal, Seller, Link. Sort by the user's goal: demand signals for "what sells", rating for "best", price for "cheapest". Mark sponsored rows.
3. "What stands out": three to five observations tied to the rows (price clusters, dominant brands, under-served price bands, review gaps).
4. One line: "Apify cost: about USD X for N rows in M runs, billed to your Apify account." Work it out from the price table (runs × start fee + charged rows × per-row price, plus listed fees). Never quote the usage, data-access or cost figures that a run response or record carries.

Prices are what the storefront showed at the time of the check, in its currency, before tax. Offer the next step: compare one product's price everywhere (`price-check`), audit a competitor's store (`competitor-store-audit`), find suppliers and margins (`product-sourcing`), or a CSV of all rows.

## Responsible use

This is public storefront data. Use it for market research, pricing and assortment decisions. Do not show individual reviewers' names. Sales figures are the marketplace's own rounded signals or, for Shopify, an estimate; never present them as exact sales.

## When things go wrong

- The ecommerce-research server is not connected or a tool call returns an authorization error: tell the user to sign in to Apify (a free account works) for the ecommerce-research server in their app's connector or MCP settings, then retry. In Claude this is the plugin's Connectors tab.
- An input comes back `empty`, `not-found` or `rejected` in the record: name it and the reason from `inputs[].detail`; suggest a broader keyword or the right storefront.
- `blocked` or `failed` inputs: the marketplace refused the lookups; nothing is charged for missing rows. Offer one retry or another marketplace.
- Mexican Walmart sites take item IDs or links only; TikTok Shop and Target are US only. Say so instead of running a search that cannot work.
- The run reports a charge limit or "maximum cost per run": the user's Apify spending limit per run stopped it; report the rows delivered and suggest raising the limit in Apify Console or lowering the count.
