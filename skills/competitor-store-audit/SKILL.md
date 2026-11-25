---
name: competitor-store-audit
description: Audit a competitor's online store - a Shopify store's full catalog, prices, discounts, stock, new products, collections, installed apps and public contact details, a TikTok Shop store's best sellers and sold counts, or an Amazon seller's profile and listings - and summarize its assortment and pricing strategy. Use when the user gives a store URL or seller and asks "audit this store", "what does this Shopify store sell", "competitor's catalog", "their price range", "what's on sale at", "which apps does this store use", "is this a Shopify store", "best sellers of this TikTok Shop", "what's new in their store", "Amazon seller profile", or wants a competitor's products as a spreadsheet.
---

# Competitor store audit

You audit stores by running Imzlab Technologies' Actors on Apify through the **ecommerce-research** MCP server (some apps call it a connector). Match tools on the start of the Actor slug: `shopify-store`, `tiktok-shop`, `amazon-product`. The server also exposes `get-actor-run`, `get-dataset-items` and `get-key-value-store-record`.

The input and output of each Actor are in `../product-research/references/actor-contracts.md`; read it before your first call in a conversation.

## Steps

1. **Identify the store.** A website URL goes to the Shopify Actor (it returns no products if the site is not on Shopify or is locked to visitors; say so then). A tiktok.com/shop/store link goes to TikTok Shop `mode: "shop"`. An Amazon seller ID or seller link goes to Amazon `mode: "sellers"` for the profile; for that seller's prices on given ASINs use `mode: "offers"`. Several stores of the same kind fit in one run.

2. **Size the audit.**
   - Shopify: `maxProducts` 100 for an overview (raise to the full catalog when the user asks; large stores have thousands), `includeReviews: false` unless ratings or sales estimates are wanted, `scrapeTechStack: true` for the app list, `scrapeCollections: true` for category structure, `scrapeContactInfo: true` only when the user wants the store's public contact details.
   - TikTok Shop: `sortBy: "top"` for best sellers, `new_releases` for launches, `maxResults` 50, `maxPages` 2.
   - Amazon seller: profile only (one row per seller).

3. **Estimate the cost** with the price table (a 100-product Shopify audit with tech stack is about USD 0.52; with reviews it is the same per product but much slower). If the estimate is USD 0.50 or less, run now and report the cost afterwards; above it, run only after the user agrees. For catalogs over 500 products, also say the run may take several minutes.

4. **Run, wait and fetch**: start the runs, call `get-actor-run` with `waitSecs: 45` until each run is terminal (up to 8 times; Shopify runs with reviews may need more), then `get-dataset-items` with a `limit` that covers every row (products plus contact, store-intel and collection rows) and the summary record (`SUMMARY` for Shopify, `OUTPUT` for the others). Never invent a product or a price.

5. **Analyze** from the rows:
   - assortment: products by `category` or `tags`, variant counts, share out of stock (`fully_out_of_stock`);
   - pricing: price range and median, price bands, share on sale and the average `discount_percentage`;
   - activity: products created or published in the last 30 days (`created_at`, `published_at`), recently updated items;
   - stack (Shopify): apps by category (`apps_by_category`), theme;
   - TikTok Shop: best sellers by `soldCount`, store totals (`sellerSoldCount`, `sellerProductCount`, `sellerFollowersCount`), price points that sell.

## Answer format

1. Store snapshot: products read (and whether that is the whole catalog), price range and median in the store's currency, share on sale, share out of stock, newest launches.
2. A table of up to 25 products: Product, Category, Price, Was price, Discount, In stock, Variants, Added, Link. Sort by what the user asked (discount, newest, best sellers).
3. Apps and theme (Shopify) or store totals (TikTok Shop), when read.
4. "Strategy read": three to five observations tied to the data (hero categories, discount depth, launch rhythm, price ladder, tools they rely on).
5. One line: "Apify cost: about USD X for N rows in M runs, billed to your Apify account", from the price table. Never quote the usage, data-access or cost figures a run response or record carries.

Offer a CSV of the full catalog (columns from `product-schema.md`), a price check of their hero products elsewhere (`price-check`), or a re-run later to see what changed.

## Responsible use

Audit public storefronts for competitive research. Contact details are the store's own published business contacts; use them for business purposes only. `estimated_monthly_sales` is an estimate from review counts; always say so.

## When things go wrong

- The ecommerce-research server is not connected or a tool call returns an authorization error: tell the user to sign in to Apify for the ecommerce-research server in their app's connector or MCP settings, then retry.
- Shopify returns 0 products and `stores_failed` or `stores_blocked` is set: the site is not a Shopify store, is locked to visitors, or refused the requests. Say which the summary suggests.
- `stopped_by_timeout` or `stopped_by_charge_limit`: report the products delivered and suggest a smaller `maxProducts` or turning `includeReviews` off.
