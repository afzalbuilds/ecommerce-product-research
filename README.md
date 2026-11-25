# E-commerce Product & Price Research for Claude and other AI agents

Research products, prices, competitor stores and suppliers without leaving Claude. Ask about a product, a keyword, a store or a supplier and the plugin reads **Amazon (23 storefronts, including every seller's offer and the Best Sellers lists), Walmart, Target, eBay, Google Shopping, TikTok Shop, Lazada, MercadoLibre and any Shopify store**, finds suppliers on **Alibaba and AliExpress**, and returns one comparable table with prices, ratings, sales signals, sellers and links.

It works in Claude chat, in Cowork and in Claude Code, and the same repository also installs as a native plugin in OpenAI Codex, Gemini CLI, GitHub Copilot, Grok, Cursor, Hermes Agent and OpenCode, with the skills usable in many more agents (see [Other AI agents](#other-ai-agents-codex-gemini-cli-copilot-and-more)). There are no developer accounts to set up and nothing to configure. Lookups run as pay-per-result Apify Actors on your own Apify account, and you only pay for rows that come back.

## What you can ask

- "Where is the De'Longhi EC685 cheapest right now? Check Amazon, Walmart, Target and Google Shopping."
- "Show every seller offering ASIN B00CH9QWOU and who has the featured offer."
- "Top 100 Amazon Best Sellers in Kitchen & Dining, with prices and ratings."
- "Compare espresso machines under USD 500 on Amazon, Walmart and eBay. What sells best?"
- "Audit allbirds.com: price range, what's on sale, newest products and the apps they use."
- "What are the best sellers in this TikTok Shop store?"
- "Find Alibaba suppliers for silicone kitchen utensil sets: MOQ, price tiers, lead times. What margin is left at Amazon prices?"
- "USB-C chargers on Lazada Singapore and Malaysia: price range and top brands."
- "Espresso machines on MercadoLibre Mexico: price range, condition and the top sellers."
- "Check these 50 products every week and tell me which prices dropped."
- "What will a price check of 200 ASINs cost?"

Claude picks the right marketplaces, estimates the cost, runs them, waits for the results, matches the same product across stores and shows one table. When fewer rows come back than you asked for, it tells you why.

## Sources covered

| Need | Source used | Per run | What you get |
|---|---|---|---|
| Amazon search, product pages, all seller offers, seller profiles, Best Sellers (23 storefronts) | Amazon Product, Offers & Best Sellers Scraper | up to 50,000 rows | price, list price, rating, reviews, "bought in past month", best-seller rank, Prime, sponsored flag, every offer with seller and shipping |
| Walmart (US, Canada, Mexico, Costa Rica) | Walmart Price Checker | up to 1,000 rows | price, was price, stock, pickup and delivery by store or ZIP, seller, rating, UPC and model |
| Target (US) | Target Price Checker | up to 1,000 rows | online and store price, regular price, promotion, pickup, delivery and shipping, rating, UPC |
| eBay (active listings) | eBay Scraper | up to 200 per query | price, condition, auction or Buy It Now, units sold, shipping cost, location, top-rated seller |
| Many merchants at once, 20 countries | Google Shopping Scraper | up to 100 per query | price, old price, merchant, rating, delivery |
| Any Shopify store's catalog | Shopify Store Scraper | up to 10,000 products per store | prices, discounts, variants, stock, launch dates, collections, installed apps, public contact details, review-based sales estimate |
| TikTok Shop (US) | TikTok Shop Product Scraper | up to 500 products | price, units sold, rating, store followers and totals, best sellers per store |
| Lazada (Singapore, Malaysia, Thailand, Philippines, Indonesia, Vietnam) | Lazada Products Scraper | up to 1,000 rows | price, discount, rating, units sold, seller, LazMall flag |
| MercadoLibre (Mexico, Argentina, Colombia, Chile, Peru, Uruguay and the other Spanish-language MercadoLibre sites; not Brazil) | MercadoLibre Product & Seller Scraper | up to 200 products (20 on free Apify plans) | current price, condition, stock, specifications, seller name, reputation and total sales |
| AliExpress (dropshipping prices) | AliExpress Products, Reviews & Sellers | up to 500 products | price, discount, units sold, reviews, store |
| Alibaba (wholesale suppliers) | Alibaba Products & Suppliers Scraper | up to 15,000 rows | price range and tiers, minimum order, lead times, certifications, supplier country, years, verification, response time |

Not included: checkout, buying or listing on any marketplace, seller-central or private account data, or sold-listing history.

## How to set it up

1. Install the plugin from the Claude directory (claude.ai > Customize > Plugins) or in Claude Code with `claude plugin install ecommerce-product-research@imzlab-ecommerce-research` after adding the marketplace `afzalbuilds/ecommerce-product-research`.
2. Open the plugin's **Connectors** tab and connect the **ecommerce-research** server. It is Apify's official MCP server (`mcp.apify.com`) limited to the eleven marketplace Actors, so a browser window opens for you to sign in or create a free Apify account. Free accounts include monthly usage credit, so you can try it without adding a card. Claude may note that the server is not a separately registered directory connector; that is expected, because it is Apify's hosted server rather than one Imzlab runs.
3. Ask Claude.

In Claude Code, `/mcp` shows the connection state, and `/ecommerce-product-research:price-check De'Longhi EC685` runs a lookup directly.

## Other AI agents: Codex, Gemini CLI, Copilot and more

The skills and the **ecommerce-research** server are the same in every app; only the manifest each app reads differs. Every app signs you in to Apify in the browser, and results are billed to your own Apify account exactly as in Claude.

| App | Install | What loads |
|---|---|---|
| Claude (chat, Cowork, Claude Code) | [How to set it up](#how-to-set-it-up) | skills, commands, price-monitor agent, ecommerce-research server |
| OpenAI Codex (CLI and app) | [OpenAI Codex](#openai-codex) | skills, ecommerce-research server |
| Gemini CLI (and Qwen Code) | [Gemini CLI](#gemini-cli) | skills, commands, price-monitor subagent, ecommerce-research server |
| GitHub Copilot CLI and VS Code | [GitHub Copilot](#github-copilot-cli-and-vs-code) | skills, commands, price-monitor agent, ecommerce-research server |
| Grok Build | [Grok Build](#grok-build) | skills, commands, price-monitor agent, ecommerce-research server |
| Cursor and Grok Bot | [Grok Bot and Cursor](#grok-bot-and-cursor) | skills, commands, price-monitor agent, ecommerce-research server |
| Hermes Agent | [Hermes Agent](#hermes-agent) | skills, ecommerce-research server |
| OpenCode | [OpenCode](#opencode) | skills, commands, price-monitor subagent, ecommerce-research server |
| Antigravity, Kiro, Devin, Factory Droid, Augment, Junie, Mistral Vibe | [More agents](#more-agents) | skills and ecommerce-research server, plus commands and agent where the app supports them |
| Any other agent with skills support | [More agents](#more-agents) | skills, plus the server URL added by hand |

Checked on 2026-11-25: the package passes the plugin validators of Claude Code, Gemini CLI, Grok Build and Hermes Agent, and every Actor behind the ecommerce-research server was run live the same day. Install, Apify sign-in and a live request were not repeated inside each app; the formats are the ones the sibling Imzlab plugins were install-tested with on 2026-11-16 (Codex 0.167, Gemini CLI 0.68.0, GitHub Copilot CLI 1.0.112, OpenCode 1.18.27).

### OpenAI Codex

Needs Codex CLI 0.142 or newer (`codex update`). The Codex app uses the same settings, so the plugin appears there too.

```bash
codex plugin marketplace add afzalbuilds/ecommerce-product-research
```

```bash
codex plugin add ecommerce-product-research@imzlab-ecommerce-research
```

```bash
codex mcp login ecommerce-research
```

The last command opens Apify's sign-in page in your browser. Then ask Codex in plain words. You can also call a skill by name, for example `$product-research`. Codex loads the skills and the ecommerce-research server, but not the plugin's commands or the price-monitor agent.

### Gemini CLI

```bash
gemini extensions install https://github.com/afzalbuilds/ecommerce-product-research
```

When Gemini CLI first connects to the ecommerce-research server it opens Apify's sign-in page in your browser; if it does not, run `/mcp auth ecommerce-research` inside Gemini CLI. The extension adds the skills, the `/product-research`, `/price-check`, `/store-audit` commands and the price-monitor subagent. Update it later with `gemini extensions update ecommerce-product-research`.

Qwen Code installs the same extension: `qwen extensions install afzalbuilds/ecommerce-product-research`.

### GitHub Copilot CLI and VS Code

```bash
copilot plugin marketplace add afzalbuilds/ecommerce-product-research
```

```bash
copilot plugin install ecommerce-product-research@imzlab-ecommerce-research
```

In a Copilot CLI session, run `/mcp auth ecommerce-research` to sign in to Apify. The plugin adds the skills, the `/product-research`, `/price-check`, `/store-audit` commands, the price-monitor agent and the ecommerce-research server.

In VS Code, turn on the `chat.plugins.enabled` setting, run **Chat: Install Plugin from Source** from the Command Palette and enter `afzalbuilds/ecommerce-product-research`.

### Grok Build

```bash
grok plugin install afzalbuilds/ecommerce-product-research
```

Grok asks you to confirm that you trust the plugin; its ecommerce-research server stays off until you do. Then open `/mcps`, select **ecommerce-research** and press `i` to sign in to Apify. Then ask Grok for a price check.

### Grok Bot and Cursor

The plugin is not listed in their marketplaces yet. Once it is, add it from the marketplace and sign in to Apify when the ecommerce-research server asks.

On grok.com you can use the Actors today without the skills: open Connectors, add a custom connector and paste the server URL below.

### Hermes Agent

Needs Hermes 0.21.6 or newer, which loads Agent Plugins packages (a root `plugin.json`).

```bash
hermes plugins install afzalbuilds/ecommerce-product-research --no-enable
```

```bash
hermes plugins enable ecommerce-product-research
```

The package declares the ecommerce-research server, but Hermes signs in to a remote server only when your own configuration asks for it. Register the server under the same name with browser sign-in, so Hermes uses your Apify account for it. Run this in a terminal, not from a script, because Hermes starts the sign-in only when it can talk to you:

```bash
hermes mcp add ecommerce-research --auth oauth --url "https://mcp.apify.com/?tools=muhammadafzal/amazon-product-offers-bestsellers-scraper,muhammadafzal/walmart-price-checker-scraper,muhammadafzal/target-price-checker-scraper,muhammadafzal/ebay-scraper,muhammadafzal/google-shopping-scraper,muhammadafzal/shopify-store-scraper,muhammadafzal/tiktok-shop-product-scraper,muhammadafzal/lazada-products-scraper,muhammadafzal/mercadolibre-product-seller-scraper,muhammadafzal/aliexpress-products-reviews-sellers,muhammadafzal/alibaba-products-suppliers-scraper"
```

Hermes opens Apify's consent page in your browser. After you allow access it lists the server's tools (the eleven Actors plus the run, dataset and record tools); press Enter to enable them all. `hermes mcp test ecommerce-research` confirms the connection, and `hermes mcp login ecommerce-research` signs in again later if the access is ever revoked.

Every request is a paid Actor run on your Apify account, so the package marks the server `untrusted`: Hermes asks before each call and refuses them in unattended runs. Your own server entry replaces the package's entry, including that setting; add `trust: untrusted` under `mcp_servers.ecommerce-research` in `~/.hermes/config.yaml` to keep the prompts.

To install only one skill instead of the whole package: `hermes skills install afzalbuilds/ecommerce-product-research/skills/product-research`. The ecommerce-research server still has to be added as above.

### OpenCode

OpenCode has no plugin bundle, so the skills, the server and the extras are added separately:

```bash
npx skills add afzalbuilds/ecommerce-product-research -a opencode -g
```

```bash
opencode mcp add ecommerce-research --url "https://mcp.apify.com/?tools=muhammadafzal/amazon-product-offers-bestsellers-scraper,muhammadafzal/walmart-price-checker-scraper,muhammadafzal/target-price-checker-scraper,muhammadafzal/ebay-scraper,muhammadafzal/google-shopping-scraper,muhammadafzal/shopify-store-scraper,muhammadafzal/tiktok-shop-product-scraper,muhammadafzal/lazada-products-scraper,muhammadafzal/mercadolibre-product-seller-scraper,muhammadafzal/aliexpress-products-reviews-sellers,muhammadafzal/alibaba-products-suppliers-scraper"
```

```bash
opencode mcp auth ecommerce-research
```

For the `/product-research`, `/price-check`, `/store-audit` commands and the price-monitor subagent, copy the `opencode/commands` and `opencode/agents` folders of this repository into `~/.config/opencode/`.

### More agents

| App | How to install |
|---|---|
| Google Antigravity | `agy plugin install https://github.com/afzalbuilds/ecommerce-product-research` |
| Kiro | Powers panel, **Add Custom Power**, **Import power from GitHub**, then `https://github.com/afzalbuilds/ecommerce-product-research` |
| Devin (formerly Windsurf) | `devin plugins install afzalbuilds/ecommerce-product-research` |
| Factory Droid | `droid plugin marketplace add afzalbuilds/ecommerce-product-research`, then `droid plugin install ecommerce-product-research@imzlab-ecommerce-research --scope user` |
| Augment (Auggie CLI) | `auggie plugin marketplace add afzalbuilds/ecommerce-product-research`, then enable the plugin in `/plugins` |
| JetBrains Junie CLI | `/extensions marketplace add afzalbuilds/ecommerce-product-research`, then `/extensions install ecommerce-product-research` |
| Mistral Vibe | clone this repository into `~/.vibe/plugins/ecommerce-product-research`, run `/reload-plugins`, then `/mcp login ecommerce-research` |
| Cline, Kilo Code, Amp, Goose and other agents that read Agent Skills | `npx skills add afzalbuilds/ecommerce-product-research` installs the skills; then add the server URL below as a remote MCP server with OAuth in that app |

### Server URL

Any MCP client can use the Actors directly by adding this URL as a remote (streamable HTTP) server with OAuth:

```
https://mcp.apify.com/?tools=muhammadafzal/amazon-product-offers-bestsellers-scraper,muhammadafzal/walmart-price-checker-scraper,muhammadafzal/target-price-checker-scraper,muhammadafzal/ebay-scraper,muhammadafzal/google-shopping-scraper,muhammadafzal/shopify-store-scraper,muhammadafzal/tiktok-shop-product-scraper,muhammadafzal/lazada-products-scraper,muhammadafzal/mercadolibre-product-seller-scraper,muhammadafzal/aliexpress-products-reviews-sellers,muhammadafzal/alibaba-products-suppliers-scraper
```

## Pricing

The plugin itself is free. Each row is billed by Apify on your account at the Actor's pay-per-event price. Prices below are the Apify Store prices on the FREE tier as of 2026-11-25. Paid Apify plans get a discount, and searches that find nothing charge no rows.

| Source | Per row | Per run start | Example |
|---|---|---|---|
| Amazon | USD 0.0025 | USD 0.005 | 20 search results: about USD 0.055 |
| Walmart | USD 0.0025 | USD 0.005 | 20 search results: about USD 0.055 |
| Target | USD 0.0015 (USD 0.0045 from a product page) | USD 0.005 | 20 search results: about USD 0.035 |
| eBay | USD 0.01 | USD 0.006 | 20 listings: about USD 0.21 |
| Google Shopping | USD 0.005 | USD 0.006 | 20 products: about USD 0.11 |
| Shopify store | USD 0.005 per product (USD 0.01 per contact or app-list row) | USD 0.006 | 100 products with apps: about USD 0.52 |
| TikTok Shop | USD 0.01 | USD 0.01 | 20 products: about USD 0.21 |
| Lazada | USD 0.0027 | USD 0.005 | 20 products: about USD 0.059 |
| MercadoLibre | USD 0.025 | USD 0.083 | 10 products: about USD 0.33 |
| AliExpress | USD 0.004 per product, review or seller row | USD 0.005 | 20 products: about USD 0.085 |
| Alibaba | USD 0.0025 (USD 0.0045 from a product page) | USD 0.005 | 20 products: about USD 0.055 |

A typical single-product price check across Amazon (with all offers), Walmart, Target, Google Shopping and eBay costs about USD 0.38. Before anything estimated above USD 0.50, Claude tells you the estimate and asks before running. Ask "what will this cost" to get an estimate without running anything.

## Data and privacy

- Your request (keywords, product IDs, links, store URLs, optional ZIP or postal code) is sent to Apify's hosted MCP server (`mcp.apify.com`), which runs the Actor on your Apify account. The Actors read public storefront and listing pages.
- Results are stored in your Apify account's datasets for your plan's retention period, where you can download them as CSV, Excel or JSON. The plugin stores nothing itself and sends nothing to any other service.
- Prices are what each storefront showed at the time of the check, in its currency, before tax. Sales figures are the marketplaces' own rounded signals; the Shopify sales figure is an estimate from review counts.
- You authorize Apify through OAuth in the plugin's Connectors tab. The plugin itself holds no Apify account details and never logs in to a marketplace.
- Full policy: https://imzlab.com/privacy#ecommerce-research. Terms: https://imzlab.com/terms#ecommerce-research. Support: https://imzlab.com/support#ecommerce-research

## Frequently asked questions

**How does it know two listings are the same product?** By UPC where the store shows it, otherwise by brand plus model number. Similar titles alone are not treated as a match; bundles, colours and accessories are kept apart and marked "possible match".

**Does it convert currencies?** No, unless you ask. Prices are compared within one currency; when you ask for a conversion, Claude says the rate is approximate.

**Can I see sold prices on eBay?** No. The eBay source reads active listings, with the number of units already sold from multi-quantity listings.

**Why are AliExpress results marked partial?** AliExpress sometimes answers detail requests with a verification check. Products still come back from search data (title, price, discount, units sold) but without seller details.

**Is the margin estimate reliable?** It is a gross margin before freight, duties, marketplace fees, returns and advertising. Give Claude those figures and it computes a landed cost and net margin; otherwise treat it as a first filter.

**Why is MercadoLibre slower than the others?** It opens every product page to read the current price, stock and seller, about a minute or more per product. Claude keeps MercadoLibre runs to about 10 products, gives them enough time, and runs the other marketplaces meanwhile.

**Is this an official Amazon, Walmart, Target, eBay, Google, Shopify, TikTok, Lazada, MercadoLibre, AliExpress or Alibaba tool?** No. This plugin and the Actors are independent and not affiliated with any of these companies or with Apify.

## Skills and commands in this plugin

- **product-research**: products for a keyword or category across marketplaces, and Amazon Best Sellers, in one normalized table.
- **price-check**: one product's current price at every retailer, every Amazon seller's offer, and the cheapest in-stock option.
- **competitor-store-audit**: a Shopify or TikTok Shop store's catalog, pricing, discounts, launches and apps, or an Amazon seller's profile.
- **product-sourcing**: Alibaba and AliExpress suppliers against retail prices, with a gross-margin estimate.
- **ecommerce-cost-estimate**: estimate the Apify cost before running.
- **/ecommerce-product-research:product-research**, **/ecommerce-product-research:price-check** and **/ecommerce-product-research:store-audit** commands (Claude Code, Cowork, Gemini CLI, Copilot, Grok Build, OpenCode).
- **price-monitor** agent (Claude Code, Cowork, Gemini CLI, Copilot, Grok Build, OpenCode): unattended price and stock snapshot of a product list with a spending cap, a dated CSV and a change report since the last snapshot.

Codex and Hermes Agent load the skills and the ecommerce-research server only; there, ask in plain words. In Codex the skills can also be called by name, for example `$price-check`.

## Source and support

Source: https://github.com/afzalbuilds/ecommerce-product-research. Issues and feature requests: https://github.com/afzalbuilds/ecommerce-product-research/issues. The Actors are published on Apify Store by Imzlab Technologies under the `muhammadafzal` account.

Last verified: 2026-11-25. All eleven Actors were run live with small inputs on that date, in every mode the skills use (MercadoLibre on its build 1.0.28, which reads the current price rather than the crossed-out one); the output fields in the skills' reference files come from those runs.
