# E-commerce Product & Price Research

This extension researches products, prices, competitor stores and suppliers on Amazon, Walmart, Target, eBay, Google Shopping, TikTok Shop, Lazada, MercadoLibre, Shopify stores, AliExpress and Alibaba, with Imzlab Technologies' Actors on Apify, through the `ecommerce-research` MCP server. The full instructions live in the extension's skills. Activate the matching skill before you call any of the server's tools:

- `product-research`: products for a keyword or category across marketplaces, and Amazon Best Sellers.
- `price-check`: one product's current price everywhere, with Amazon offers.
- `competitor-store-audit`: a Shopify or TikTok Shop store's catalog, pricing, launches and apps.
- `product-sourcing`: Alibaba and AliExpress suppliers against retail prices, with a margin estimate.
- `ecommerce-cost-estimate`: the Apify cost of a job before running it.

Every tool call on `ecommerce-research` starts a paid Actor run on the user's own Apify account. Follow the skill's cost rules: estimate from the price table in its references, ask before large batches, and never quote a cost from a run's usage figures. Wait for each run to finish, account for every requested item, and never invent a result.

If the marketplace tools are missing, the server is not signed in yet: ask the user to run `/mcp auth ecommerce-research` and approve Apify's sign-in page in the browser.

Commands: `/price-check`, `/product-research`, `/store-audit`. Subagent for unattended runs that leave a report on disk: `price-monitor`.
