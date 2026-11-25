---
name: product-sourcing
description: Find suppliers and wholesale prices for a product on Alibaba (price tiers, minimum order, lead times, certifications, supplier years and verification) and AliExpress (unit prices, sold counts, reviews), compare them with current retail prices on Amazon, Walmart or Google Shopping, and estimate the gross margin. Use when the user asks "find suppliers for", "source this product", "Alibaba suppliers", "wholesale price of", "MOQ for", "AliExpress price for", "dropshipping product", "private label", "what margin can I make on", "landed cost", or "is this product worth selling".
---

# Product sourcing

You source products by running Imzlab Technologies' Actors on Apify through the **ecommerce-research** MCP server (some apps call it a connector). Match tools on the start of the Actor slug: `alibaba`, `aliexpress`, and for retail prices `amazon-product`, `walmart`, `google-shopping`. The server also exposes `get-actor-run`, `get-dataset-items` and `get-key-value-store-record`.

The input and output of each Actor are in `../product-research/references/actor-contracts.md`; read it before your first call in a conversation.

## Steps

1. **Pin down the product and the model.** Product type and key specs (material, size, capacity), target market and currency, and the business model: wholesale or private label (Alibaba, buying hundreds of units) or dropshipping (AliExpress, single units). Default: both, 20 results each, plus the retail benchmark in the user's market.

2. **Plan the runs:**
   - Alibaba: `searchQueries` with specific buyer wording (specs beat pages), `maxPages: 1`, `maxResults` 20. For the 3 to 5 most promising suppliers, a second run with their `productUrls` for price tiers, lead times and certifications (product-page fee applies; about a minute each).
   - AliExpress: `searchQueries`, `maxProducts` 20, `includeReviews: false`, `includeSellers: false`, `currency` and `shipToCountry` for the user's market.
   - Retail benchmark: Amazon `mode: "search"` and Google Shopping, 20 each, in the user's market.

3. **Estimate the cost** with the price table (default plan about USD 0.30; the Alibaba product-page run adds USD 0.005 plus USD 0.0045 per page). If the estimate is USD 0.50 or less, run now and report the cost afterwards; above it, run only after the user agrees.

4. **Run, wait and fetch**: start the runs together, call `get-actor-run` with `waitSecs: 45` until each run is terminal (up to 8 times; Alibaba search takes one to three minutes), then `get-dataset-items` and the `OUTPUT` record. Never invent a supplier, price or minimum order.

5. **Compare.** Keep supplier rows that match the specs. Retail benchmark = median price of matching retail rows with at least 50 reviews. Unit cost = Alibaba tier price at the user's quantity (or `priceMin` to `priceMax` from search, marked as a range) or the AliExpress price. Gross margin before costs = (retail benchmark − unit cost) ÷ retail benchmark. List the costs this leaves out: shipping and freight, duties and taxes, marketplace and payment fees, returns, advertising; if the user gives figures for them, compute a landed cost and a net margin, otherwise do not guess them.

## Answer format

1. Retail benchmark line: median, range and count of matching retail products, with currency and date.
2. Supplier table (up to 15): Source, Product, Unit price (or tier at the user's quantity), MOQ, Lead time, Supplier, Country, Years, Verified, Rating, Link.
3. Margin table for the top options: Unit cost, Retail benchmark, Gross margin before costs, Notes.
4. "Checks before ordering": samples, certifications the market needs (from `certifications` where shown), and supplier signals (`supplierYears`, `supplierVerification`, `supplierOnTimeDeliveryRate`, `supplierResponseTime`). Say these are listing facts, not a vetting result.
5. One line: "Apify cost: about USD X for N rows in M runs, billed to your Apify account", from the price table. Never quote the usage, data-access or cost figures a run response or record carries.

## Responsible use

Margins here are rough, before freight, duties, fees and returns; say so every time. Do not suggest copying another brand's trademarked product, design or listing photos. Certifications and verification badges are what the listing shows; the user must confirm them with the supplier.

## When things go wrong

- The ecommerce-research server is not connected or a tool call returns an authorization error: tell the user to sign in to Apify for the ecommerce-research server in their app's connector or MCP settings, then retry.
- AliExpress `OUTPUT.status` is `PARTIAL`: products came from search data without seller details or localized prices; use the prices with that caveat.
- Alibaba returns only the first page: later pages are best-effort; refine the keyword instead.
- No matching retail rows: give the supplier table and say the benchmark is missing, rather than guessing a retail price.
