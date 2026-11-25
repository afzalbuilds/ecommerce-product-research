---
description: Compare one product's current price across Amazon (all sellers), Walmart, Target, eBay and Google Shopping merchants
argument-hint: <brand and model, ASIN, UPC or product link> [ZIP code] [new|used]
---

Check prices using the `price-check` skill from this plugin for: $ARGUMENTS

Identify the exact product, plan one lookup per marketplace (Amazon product and offers, Walmart, Target, Google Shopping, plus eBay for used), estimate the cost and ask first when it is above USD 0.50, run the Actors through the ecommerce-research MCP server, wait for every run, keep only rows that match the product, and present the cheapest in-stock option, the price table sorted by total, the Amazon offers block and the Apify cost line.
