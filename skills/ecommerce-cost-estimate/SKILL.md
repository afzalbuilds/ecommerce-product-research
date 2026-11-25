---
name: ecommerce-cost-estimate
description: Estimate what an e-commerce research job will cost on Apify before running anything. Use when the user asks "how much will it cost", "price per product", "is it free", "how is this billed", "cheapest way to", "cost of checking 500 products", "what does a weekly price monitor cost", or wants to compare the prices of the Amazon, Walmart, Target, eBay, Google Shopping, Shopify, TikTok Shop, Lazada, AliExpress or Alibaba sources, or asks about Apify credits, plans or charges for this plugin.
---

# E-commerce research cost estimate

Prices are in `../product-research/references/actor-contracts.md` (FREE-tier Apify Store prices, verified 2026-11-25). Use that file as the only source; do not quote other numbers.

## How billing works (two or three sentences when asked)

The plugin is free. Each source is a pay-per-event Actor on Apify Store, billed to the user's own Apify account: one event per row written to the dataset, plus a small start event per run and, for a few sources, a fee per product page read. Empty, not-found and blocked inputs charge no rows. Free Apify accounts include monthly usage credit; paid plans get tiered discounts of up to 20 percent per row.

## Price summary (FREE tier)

| Source | Per row | Extra | Per run start |
|---|---|---|---|
| Amazon (search, product, offers, sellers, Best Sellers) | USD 0.0025 | none | USD 0.005 |
| Walmart | USD 0.0025 | none | USD 0.005 |
| Target | USD 0.0015 | USD 0.003 per product page | USD 0.005 |
| eBay | USD 0.01 | none | USD 0.006 |
| Google Shopping | USD 0.005 | none | USD 0.006 |
| Shopify store | USD 0.005 per product or collection | USD 0.01 per contact or store-intel row, USD 0.0019 per review | USD 0.006 |
| TikTok Shop | USD 0.01 | none | USD 0.01 |
| Lazada | USD 0.0027 | none | USD 0.005 |
| AliExpress | USD 0.004 per product, review or seller row | none | USD 0.005 |
| Alibaba | USD 0.0025 | USD 0.002 per product page | USD 0.005 |

## Estimate

1. Turn the request into runs with the routing in `product-research`, `price-check`, `competitor-store-audit` or `product-sourcing`: one run per marketplace (several keywords, IDs or stores fit in one run), one per Amazon mode.
2. Cost per Actor = runs × start fee + rows × per-row price + extra fees.
3. Present a small table (Source, rows, runs, per row, subtotal) and the total as "about USD X". Say that the real charge is lower when a search returns fewer rows than requested.
4. Reference figures: 20 products on each of Amazon, Walmart, Target, eBay, Google Shopping and TikTok Shop about USD 0.67; a typical single-product price check about USD 0.38; a 100-product Shopify audit with apps about USD 0.52; a default sourcing check (20 each from Alibaba, AliExpress, Amazon and Google Shopping) about USD 0.30; a weekly price monitor of 50 products on Amazon and Walmart by ID about USD 0.26 per week.
5. Add the rough time: seconds for Amazon, Walmart, eBay, Google Shopping, TikTok Shop and Lazada searches; about a minute per Target product page and per Alibaba search; Shopify with reviews about 15 seconds per product.

Do not run any Actor from this skill. If the user then says to go ahead, hand over to the matching skill.
