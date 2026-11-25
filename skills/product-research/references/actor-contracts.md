# Actor contracts

Prices are Apify Store pay-per-event prices on the FREE tier as of 2026-11-25; paid Apify plans are 2.5 to 20 percent lower. A row is charged when it is written to the dataset (extra fees are noted per Actor). Every run also charges one start event. Schemas were read from the Actors' live builds on 2026-11-25 and every output shape below was checked against a live run the same day.

All Actors belong to Apify user `muhammadafzal`. On the **ecommerce-research** MCP server the tool name is the Actor ID with `/` replaced by `--`, for example `muhammadafzal--amazon-product-offers-bestsellers-scraper`. Some apps add a server prefix, turn hyphens into underscores, or shorten a long name and add a suffix; match on the start of the Actor slug. Each tool returns the run (`runId`, `status`, `storages.datasets.default.id`, `storages.keyValueStores.default.id`), not the rows. If a tool response already includes dataset items, use them; otherwise fetch them with `get-dataset-items`.

Run options (`memory`, `timeout`, `maxTotalChargeUsd`) go in the tool's `callOptions`, never inside `input`. Send only the input fields listed below. Every Actor handles its own network routing internally, so nothing else is needed.

## Shared rules

- Always set the result cap (`maxResults`, `maxProducts`) to the number the user asked for (default 20 per marketplace), never leave the Actor's default, which can be 1,000.
- Prices are in the storefront's currency, as the storefront showed them to a shopper at that moment, without tax. Say "at the time of the check" and give the currency.
- Always compute a cost from the prices in this file. A run response or summary record can include usage, data-access or cost fields (`usageTotalUsd`, `estimatedUsd`, `dataAccessUsage`); those are internal figures, not what the user is billed, so never report them as the cost.
- The summary record named per Actor explains the run: rows delivered, per-input status (`ok`, `empty`, `not-found`, `rejected`, `blocked`, `failed`), why it stopped, and warnings. Read it whenever fewer rows than requested came back.

---

## Amazon: `amazon-product-offers-bestsellers-scraper`

Price: USD 0.0025 per row, start USD 0.005 (256 MB). `maxResults` 1 to 50,000. About 5 seconds for a search page or a product, 25 seconds for a Best Sellers page.

Input (one `mode` per run; each mode reads only its own list):
```json
{
  "mode": "search",
  "searchQueries": ["espresso machine"],
  "marketplace": "com",
  "sortBy": "featured",
  "minPrice": 100,
  "maxPrice": 600,
  "maxPages": 1,
  "maxResults": 20
}
```
- `mode`: `search` (`searchQueries`, up to 50 keywords; `sortBy`: `featured` | `price_low_to_high` | `price_high_to_low` | `average_review` | `most_recent` | `bestsellers`; optional `minPrice`, `maxPrice`), `product` (`asins` or `productUrls`, up to 500), `offers` (`asins`: every seller's offer), `sellers` (`sellerIds` starting with `A` or seller page links), `bestsellers` (`categoryIds` such as `172282` or a Best Sellers link; 50 per page, `maxPages: 2` for the top 100).
- `marketplace`: `com`, `ca`, `com.mx`, `com.br`, `co.uk`, `ie`, `de`, `fr`, `it`, `es`, `nl`, `com.be`, `se`, `pl`, `com.tr`, `ae`, `sa`, `eg`, `in`, `co.jp`, `sg`, `com.au`, `co.za`. Links always use their own storefront.
- `postalCode`: optional buyer location for local price, stock and delivery (`10001` on amazon.com). `maxPages`: pages per input (search 20 to 80 products per page, offers 10 per page).

Dataset rows (camelCase) by `recordType`:
- `searchResult`: `asin`, `title`, `url`, `imageUrl`, `price`, `listPrice`, `currency`, `rating`, `reviewsCount`, `salesVolume` (for example `5K+ bought in past month`), `isPrime`, `isSponsored`, `isAmazonChoice`, `isBestSeller`, `couponDiscount`, `shippingInformation`, `offersCount`, `placement`, `positionOnPage`, `page`, `searchQuery`, `totalResults`, `marketplace`, `postalCode`.
- `product`: `asin`, `parentAsin`, `title`, `brand`, `url`, `price`, `listPrice`, `currency`, `availability`, `inStock`, `rating`, `reviewsCount`, `ratingDistribution[]`, `salesVolume`, `bestSellersRank[]` (`{category, rank}`), `categoryPath[]`, `bulletPoints[]`, `description`, `images[]`, `sellerName`, `sellerId`, `shipsFrom`, `fulfilledByAmazon`, `offersCount`, `isPrimeEligible`, `isAmazonChoice`, `coupon`, `deliveryOptions[]`, `variations[]`, `productDetails`.
- `offer`: `asin`, `productTitle`, `condition`, `price`, `shippingPrice`, `currency`, `sellerName`, `sellerId`, `sellerRating`, `sellerRatingsCount`, `shipsFrom`, `deliveryOptions[]`, `position` (position 1 is the featured offer, usually the Buy Box), `url`.
- `seller`: `sellerId`, `sellerName`, `businessName`, `businessAddress`, `rating`, `description`, `url`, `missingFields[]`.
- `bestseller`: `categoryId`, `rank`, `asin`, `title`, `url`, `price`, `priceText`, `currency`, `rating`, `ratingsCount`, `page`.

Key-value record `OUTPUT`: `status`, `mode`, `marketplace`, `delivered`, `resultEvents`, `inputCounts`, `inputs[]` (per input: `status`, `records`, `pages`, `detail`), `rejectedInputs[]`, `stopReason`, `warnings[]`.

---

## Walmart: `walmart-price-checker-scraper`

Price: USD 0.0025 per row, start USD 0.005 (256 MB). `maxResults` 1 to 1,000; `maxPagesPerQuery` 1 to 25 (40 to 60 products per page). About 5 to 8 seconds.

Input:
```json
{
  "site": "walmart.com",
  "searchQueries": ["espresso machine"],
  "productIds": ["2258193826"],
  "deliveryZip": "10001",
  "maxResults": 20,
  "maxPagesPerQuery": 1
}
```
- `site`: `walmart.com` | `walmart.ca` | `walmart.com.mx` | `walmart.co.cr` | `bodegaaurrera.com.mx`. Keyword search works on walmart.com, walmart.ca and walmart.co.cr; the Mexican sites take item IDs or links only.
- `productIds` (the number at the end of a walmart.com/ip/… link) or `productUrls` for known items; `searchQueries` up to 50.
- `storeId`, `deliveryZip` and `fulfillment` (`shipping` | `delivery` | `pickup`) localize price and stock. Left empty, Walmart picks a store per page; each row names the store used.

Dataset row (camelCase), `recordType` `searchResult` or `product`: `itemId`, `title`, `brand`, `url`, `imageUrl`, `price`, `wasPrice`, `currency`, `unitPrice`, `inStock`, `availabilityStatus`, `fulfillmentMethods[]`, `fulfillmentText`, `deliveryDate`, `sellerName` (`Walmart.com` or a marketplace seller), `sellerId`, `rating`, `reviewCount`, `sponsored`, `position`, `searchQuery`, `upc`, `model`, `category`, `offerType`, `otherSellersCount`, `storeId`, `storePostalCode`, `storeCity`, `storeState`, `site`, `input`. `upc` and `model` come from product rows (item IDs), not search rows.

Key-value record `OUTPUT`: `status`, `site`, `storeContext`, `delivered`, `inputCounts`, `inputs[]`, `rejectedInputs[]`, `stopReason`, `warnings[]`, `pageChecks`.

---

## Target: `target-price-checker-scraper`

Price: USD 0.0015 per search or category row; a row read from a product page (TCIN or link) costs USD 0.0015 plus a USD 0.003 product-page fee; start USD 0.005 (256 MB). `maxResults` 1 to 1,000; `maxPagesPerQuery` 1 to 20 (24 products per page). Search: about 5 seconds. Product pages: about 60 to 90 seconds each; set `callOptions.timeout: 900` for more than 5 product pages.

Input:
```json
{
  "searchQueries": ["espresso machine"],
  "productIds": ["1010937670"],
  "categoryUrls": ["https://www.target.com/c/drip-coffee-makers-espresso-kitchen-appliances-dining/-/N-658ml"],
  "storeId": "1375",
  "deliveryZip": "10001",
  "maxResults": 20,
  "maxPagesPerQuery": 1
}
```
- US only. `productIds` are TCINs (the digits after `A-` in a product link); `productUrls` also work. `storeId` and `deliveryZip` localize; `fulfillment` (`pickup` | `delivery` | `shipping`) needs `deliveryZip`.

Dataset row (camelCase): `tcin`, `title`, `brand`, `price`, `priceMax`, `priceText`, `regularPrice`, `onSale`, `promotion`, `currency`, `pickupAvailable`, `pickupMessage`, `deliveryAvailable`, `deliveryMessage`, `shippingAvailable`, `shippingMessage`, `rating`, `reviewCount`, `upc`, `dpci`, `category`, `categoryPath`, `badges[]`, `imageUrl`, `url`, `priceScope` (`online` | `store`), `storeId`, `storeName`, `source` (`searchPage` | `categoryPage` | `productPage`), `input`, `searchQuery`, `position`. `upc` and store availability come from product-page rows only.

Key-value record `OUTPUT`: `status`, `storeContext`, `delivered`, `productPageRows`, `listingRows`, `chargedEvents`, `inputCounts`, `inputs[]`, `stopReason`, `warnings[]`.

---

## eBay: `ebay-scraper`

Price: USD 0.01 per listing, start USD 0.006 (256 MB). Up to 20 queries per run; `maxResults` 1 to 200 per query. About 7 seconds per query.

Input:
```json
{
  "searchQueries": ["breville barista express"],
  "maxResults": 20,
  "condition": "Used",
  "buyingFormat": "BuyItNow",
  "freeShipping": false,
  "topRatedOnly": false
}
```
- `condition`: `all` | `New` | `Used` | `Refurbished`. `buyingFormat`: `all` | `Auction` | `BuyItNow`. Results are active listings on ebay.com, not sold or completed ones.

Dataset row (camelCase): `itemId`, `title`, `price`, `priceMax` (set for multi-variation listings), `currency`, `condition`, `buyingFormat`, `itemUrl`, `imageUrl`, `seller`, `sellerRating`, `sellerFeedback`, `productRating`, `productRatingCount`, `soldCount` (units already sold from a multi-quantity listing), `shippingCost`, `itemLocation`, `bids`, `timeLeft`, `isTopRatedSeller`, `searchQuery`, `scrapedAt`. Seller and condition fields are often `null` on search results.

Key-value record `OUTPUT`: `status` (`DATA` | `EMPTY` | other), `delivered`, `duplicatesSkipped`, `queries[]` (per query: `delivered`, `pages`, `status`, `stopReason`), `warnings[]`.

---

## Google Shopping: `google-shopping-scraper`

Price: USD 0.005 per product, start USD 0.006 (256 MB). Up to 50 queries; `maxResults` 1 to 100 per query (Google lists about 60 distinct products). About 6 seconds per query.

Input:
```json
{
  "searchQueries": ["delonghi dedica ec685"],
  "maxResults": 20,
  "countryCode": "us",
  "languageCode": "en",
  "location": "New York,New York,United States",
  "sortBy": "relevance"
}
```
- `countryCode`: `us`, `gb`, `de`, `fr`, `ca`, `au`, `jp`, `in`, `br`, `mx`, `es`, `it`, `nl`, `se`, `no`, `dk`, `fi`, `pl`, `sg`, `ae`. `languageCode` to match.
- `sortBy`: `relevance` | `price_low` | `price_high` | `review_score`. A sort cannot be combined with `minPrice` or `maxPrice` (Google drops the sort), so send one or the other.

Dataset row (camelCase): `title`, `price` (text), `priceValue`, `currency`, `oldPrice`, `oldPriceValue`, `source` (the merchant, for example `Target`), `productUrl` (a Google Shopping link), `productId`, `imageUrl` (often an embedded data image; do not show it), `rating`, `ratingCount`, `delivery`, `position`, `searchQuery`, `scrapedAt`.

Key-value record `OUTPUT`: `status`, `delivered`, `queries[]` (per query: `delivered`, `status`), `country`, `location`, `stopReason`, `warnings[]`.

---

## Shopify stores: `shopify-store-scraper`

Price: USD 0.005 per product, USD 0.005 per collection, USD 0.01 per contact record, USD 0.01 per store-intel record, USD 0.0019 per review; start USD 0.006 (256 MB). `maxProducts` 1 to 10,000 per store. A catalog-only run takes seconds to a minute per store; with `includeReviews` on, every product page is read too (about 15 seconds per product in testing).

Input:
```json
{
  "storeUrls": [{"url": "https://www.allbirds.com"}],
  "maxProducts": 50,
  "includeReviews": false,
  "maxReviewsPerProduct": 0,
  "scrapeCollections": false,
  "scrapeContactInfo": false,
  "scrapeTechStack": true,
  "requestDelayMs": 500
}
```
- `storeUrls` are objects with `url`; product links are reduced to the store. It does not discover stores: the user brings the list. Stores locked to visitors and non-Shopify sites return no products.
- Always set `includeReviews` explicitly: `false` for a catalog and pricing audit (fast), `true` only when ratings or sales estimates are wanted. `maxReviewsPerProduct` above 0 also returns review rows.
- `scrapeTechStack: true` adds one store-intel row (apps by category, theme); `scrapeContactInfo: true` adds one contact row (public store email, phone, social links); `scrapeCollections: true` adds collection rows.

Dataset rows (snake_case) by `recordType`:
- `product`: `store_url`, `product_id`, `title`, `handle`, `brand`, `category`, `tags[]`, `description`, `product_url`, `current_price`, `original_price`, `min_price`, `max_price`, `discount_percentage`, `available`, `fully_out_of_stock`, `variant_count`, `options[]`, `variants[]`, `image_count`, `main_image`, `rating`, `review_count`, `estimated_monthly_sales`, `sales_estimate_basis`, `currency`, `published_at`, `created_at`, `updated_at`.
- `collection`: title, handle, description, product count, image.
- `contact`: `merchant_name`, `merchant_email`, `merchant_phone`, `merchant_address`, `social_facebook`, `social_instagram`, `social_twitter`, `social_tiktok`, `social_youtube`, `social_pinterest`, `contact_page_url`. Phone numbers without a tel: link are pattern matches and can be wrong.
- `storeIntel`: `shop_name`, `theme_name`, `theme_id`, `apps_installed[]`, `apps_by_category` (for example `{"SMS Marketing": ["Attentive"]}`), `apps_count`, `total_products`, `domain`.

`currency` on product rows is often `null`; prices are in the store's own currency, so say "in the store's currency" unless the storefront shows it. `estimated_monthly_sales` is the Actor's estimate from review counts (2 percent of buyers review by default), not real sales; always label it an estimate.

Key-value record `SUMMARY`: `stores_requested`, `products_scraped`, `collections_scraped`, `reviews_scraped`, `contacts_scraped`, `store_intel_scraped`, `stopped_by_charge_limit`, `stores_failed`, `stores_blocked`, `stopped_by_timeout`.

---

## TikTok Shop: `tiktok-shop-product-scraper`

Price: USD 0.01 per product, start USD 0.01 (256 MB). `maxResults` 1 to 500. US TikTok Shop only. About 5 to 10 seconds.

Input:
```json
{
  "mode": "search",
  "queries": ["phone case", "magsafe wallet"],
  "productUrls": ["https://www.tiktok.com/shop/pdp/1729587769570529799"],
  "shopUrls": ["https://www.tiktok.com/shop/store/goli-nutrition/7495794203056835079"],
  "sortBy": "top",
  "maxResults": 20,
  "maxPages": 1,
  "enrichProducts": false
}
```
- `mode`: `search` (`queries`, up to 20), `products` (`productUrls`, up to 100), `shop` (`shopUrls`, up to 20 store catalogs; `sortBy`: `top` for best sellers, `new_releases`). Send only the list the mode uses.
- `enrichProducts: true` adds variants, stock, category and seller rating at the same price, but is slower.

Dataset row (camelCase): `productId`, `title`, `url`, `region`, `price`, `originalPrice`, `currency`, `soldCount`, `rating`, `reviewCount`, `sellerId`, `sellerName`, `sellerUrl`, `sellerRating`, `sellerSoldCount`, `sellerProductCount`, `sellerFollowersCount`, `images[]`, `category`, `variants[]`, `stock`, `mode`, `query`, `inputSource`. Shop mode fills the seller fields; search rows often leave them and the rating `null`.

Key-value record `OUTPUT` (ignore `STATE`): `status` (`DATA` | `EMPTY` | other), `delivered`, `sourceErrors`, `warnings[]`, `stopReason`.

---

## Lazada (Southeast Asia): `lazada-products-scraper`

Price: USD 0.0027 per row, start USD 0.005 (256 MB). `maxResults` 1 to 1,000; `maxPagesPerQuery` 1 to 10 (40 products per page). About 10 to 60 seconds.

Input:
```json
{
  "searchQueries": ["usb c charger"],
  "country": "SG",
  "maxResults": 20,
  "maxPagesPerQuery": 1
}
```
- `country`: `SG` | `MY` | `TH` | `PH` | `ID` | `VN` (prices in SGD, MYR, THB, PHP, IDR or VND). `productIds` (the number after `-i` in a product link) or `productUrls` for known items.

Dataset row (camelCase): `productId`, `skuId`, `country`, `storefront`, `title`, `url`, `imageUrl`, `price`, `priceText`, `originalPrice`, `currency`, `discountText`, `discountPercent`, `rating`, `reviewCount`, `soldText`, `soldCount`, `sellerName`, `sellerId`, `brandName`, `location`, `inStock`, `isSponsored`, `isLazMall` (official brand store), `highlights[]`, `searchQuery`, `position`.

Key-value record `OUTPUT`: `status`, `delivered`, `country`, `inputCounts`, `inputs[]`, `stopReason`, `warnings[]`.

---

## MercadoLibre (Latin America): `mercadolibre-product-seller-scraper`

Price: USD 0.025 per product, start USD 0.083 (512 MB, one event). `maxResults` 1 to 200 per run; **free Apify plans get at most 20 products per run**. Slow: every product page is rendered, about 60 to 100 seconds each, a few at a time. The run stops 20 seconds before its timeout (default 300 seconds), so always set `callOptions.timeout` to about 90 × products + 120 seconds (at most 3,600) and keep runs to 10 products or fewer unless the user accepts a long wait. Live on build 1.0.28 (2026-11-25), which reads the current price, not the crossed-out one.

Input:
```json
{
  "startUrls": ["https://listado.mercadolibre.com.mx/cafetera-espresso"],
  "maxResults": 10,
  "maxPages": 1,
  "includeSellerDetails": true
}
```
- `startUrls` is required: search pages or product pages. Build a search URL as `https://listado.mercadolibre.<domain>/<keywords-joined-with-hyphens>`, for example `listado.mercadolibre.com.mx/cafetera-espresso`, `listado.mercadolibre.com.ar/iphone-17`, `listado.mercadolibre.cl/zapatillas-running`. Domains: `com.mx` (Mexico), `com.ar` (Argentina), `com.co` (Colombia), `cl` (Chile), `com.pe` (Peru), `com.uy` (Uruguay), `com.ec` (Ecuador), `co.cr` (Costa Rica) and the other `mercadolibre.com.<cc>` sites. Brazil (mercadolivre.com.br) is not accepted.
- Product pages: `…/p/MLA1055308917` catalog links or `…/MLM-123456789-title` item links. Search terms are best in the country's language (Spanish).
- `maxPages` 1 to 10 search pages (about 48 products each); a further page opens only when earlier pages did not fill `maxResults`.

Dataset row (camelCase): `productId` (for example `MLM2883102053`), `title`, `priceAmount` (the current price, in `currency`), `currency` (`MXN`, `ARS`, `COP`, `CLP`, `PEN`, `UYU`, `USD` for dollar listings, and others), `condition` (`Nuevo`, `Usado`, `Reacondicionado`), `availableQuantity` (a listing that says "+50 disponibles" gives 50, meaning 50 or more), `description`, `imageUrls[]`, `attributes` (object of the listing's specification table, for example `Marca`, `Modelo`), `seller` (`{name, profileUrl, reputation, salesText}`; `reputation` such as `MercadoLíder Platinum`, `salesText` such as `+10 mil ventas` is the seller's total, not this product's), `productUrl`, `sourceSearchUrl`, `scrapedAt`, `detailsFetched`, `warnings[]`. No rating or review count.

Key-value record `RUN_SUMMARY`: `outcome` (`data` | `empty` | `rejected` | other), `recordsDelivered`, `customResultEventsCharged`, `chargeLimitReached`, `freePlanLimitApplied`, `stopReason` (for example `maxResults (10) reached`, or `run time limit reached with N page(s) not opened; raise the run timeout for more products`), `blockedUrls[]`, `warnings[]`. When the stop reason is the run time limit, report the products delivered and offer a re-run with a longer `callOptions.timeout`.

---

## AliExpress: `aliexpress-products-reviews-sellers`

Price: USD 0.004 per record (product, review or seller rows are each one record), start USD 0.005 (512 MB, one event). `maxProducts` 1 to 500 unique products. About 5 to 10 seconds for 10 products.

Input:
```json
{
  "searchQueries": ["espresso machine"],
  "productUrls": [{"url": "https://www.aliexpress.com/item/1005008082049439.html"}],
  "maxProducts": 20,
  "includeReviews": false,
  "maxReviewsPerProduct": 5,
  "includeSellers": false,
  "currency": "USD",
  "shipToCountry": "US"
}
```
- `searchQueries` up to 20; `productUrls` are objects with `url` for exact product pages.
- Reviews and sellers are extra records at the same price each: keep `includeReviews` and `includeSellers` off unless the user wants them, or the row count (and cost) multiplies.
- `currency`: `USD` | `EUR` | `GBP` | `CAD` | `AUD` | `JPY` | `PKR`. `shipToCountry`: two-letter code.

Dataset rows (camelCase) by `recordType`: `product` (`productId`, `productUrl`, `title`, `imageUrl`, `price`, `originalPrice`, `currency`, `discountPercent`, `rating`, `reviewCount`, `soldCountText`, `sellerName`, `sellerId`, `storeId`, `sellerUrl`, `properties[]`, `skuProperties[]`, `searchQuery`), `review` (`productId`, `reviewId`, `reviewerName` masked, `reviewerCountry`, `rating`, `reviewText`, `reviewDate`, `options[]`, `imageUrls[]`, `sellerReply`), `seller` (store metrics when available).

Data notes: AliExpress often answers product-detail requests with a verification challenge. The run then ends with `OUTPUT.status` `PARTIAL`: products still come from search data (title, price, discount, sold count) but without seller details or localized pricing, and seller rows may be missing. Report it as partial, not as an error.

Key-value record `OUTPUT`: `status` (`SUCCEEDED` | `PARTIAL` | other), `products`, `reviews`, `sellers`, `totalRecords`, `warnings[]`.

---

## Alibaba (suppliers): `alibaba-products-suppliers-scraper`

Price: USD 0.0025 per product row; rows read from a product page (`productUrls`) add a USD 0.002 product-page fee; start USD 0.005 (256 MB). `maxResults` 1 to 15,000. A search page (about 60 products) takes one to three minutes; product pages 20 to 90 seconds each, three at a time.

Input:
```json
{
  "searchQueries": ["espresso machine"],
  "productUrls": ["1601043752113"],
  "maxPages": 1,
  "maxResults": 20
}
```
- `searchQueries`: buyer keywords. `maxPages` 1 to 10; pages after the first are best-effort (Alibaba often refuses them), so prefer more specific keywords over more pages.
- `productUrls`: alibaba.com product links or bare product IDs, up to 100, for full details (price tiers, lead times, certifications, supplier facts).

Dataset row (camelCase): `productId`, `source` (`search` | `productPage`), `title`, `url`, `searchQuery`, `position`, `isSponsored`, `priceText`, `priceMin`, `priceMax`, `currency`, `priceTiers[]` (`{minQuantity, maxQuantity, price}`, product pages), `minOrderQuantity`, `minOrderUnit`, `soldCount`, `imageUrl`, `categoryPath[]`, `attributes[]`, `leadTimes[]` (`{minQuantity, maxQuantity, days}`), `certifications[]`, `supplierName`, `supplierUrl`, `supplierCountry`, `supplierYears`, `supplierVerification`, `supplierBadges[]`, `supplierBusinessType`, `supplierRating`, `supplierReviewCount`, `supplierResponseTime`, `supplierOnTimeDeliveryRate`. Search rows leave tiers, lead times, certifications and some supplier facts `null`.

Key-value record `OUTPUT`: `status`, `delivered`, `resultEvents`, `productFeeEvents`, `inputCounts`, `inputs[]`, `stopReason`, `warnings[]`.

---

## Cost formula

cost = runs × start fee + charged rows × per-row price (+ product-page or record-type fees where listed).

Examples on the FREE tier: 20 Amazon search results ≈ USD 0.055; offers for 5 ASINs at 10 offers each ≈ USD 0.13; 20 Walmart results ≈ USD 0.055; 20 Target search results ≈ USD 0.035, 5 Target product pages ≈ USD 0.0275; 20 eBay listings ≈ USD 0.206; 20 Google Shopping products ≈ USD 0.106; a Shopify store audit of 100 products with tech stack ≈ USD 0.516; 20 TikTok Shop products ≈ USD 0.21; 20 Lazada products ≈ USD 0.059; 10 MercadoLibre products ≈ USD 0.333; 20 AliExpress products ≈ USD 0.085; 20 Alibaba products from search ≈ USD 0.055.

Quote costs as "about": searches often return fewer rows than requested, and the user's Apify plan tier may be discounted.
