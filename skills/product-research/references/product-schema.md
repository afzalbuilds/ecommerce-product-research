# Normalized product rows, matching and CSV export

Every marketplace names its fields differently. Normalize rows to these columns before comparing, merging or exporting.

| Column | Amazon | Walmart | Target | eBay | Google Shopping | Shopify | TikTok Shop | Lazada | AliExpress | Alibaba | MercadoLibre |
|---|---|---|---|---|---|---|---|---|---|---|---|
| marketplace | `amazon.<marketplace>` | `site` | `target.com` | `ebay.com` | `google shopping: <source>` | store domain | `tiktok shop` | `storefront` | `aliexpress` | `alibaba` | `mercadolibre.<domain>` |
| id | `asin` | `itemId` | `tcin` | `itemId` | `productId` | `product_id` | `productId` | `productId` | `productId` | `productId` | `productId` |
| title | `title` | `title` | `title` | `title` | `title` | `title` | `title` | `title` | `title` | `title` | `title` |
| brand | `brand` | `brand` | `brand` | from title | from title | `brand` | from title | `brandName` | from title | from title | `attributes.Marca` or title |
| price | `price` | `price` | `price` | `price` | `priceValue` | `current_price` | `price` | `price` | `price` | `priceMin` (and `priceMax`) | `priceAmount` |
| was_price | `listPrice` | `wasPrice` | `regularPrice` | none | `oldPriceValue` | `original_price` | `originalPrice` | `originalPrice` | `originalPrice` | none | none |
| currency | `currency` | `currency` | `currency` | `currency` | `currency` | store currency | `currency` | `currency` | `currency` | `currency` | `currency` |
| rating | `rating` | `rating` | `rating` | `productRating` | `rating` | `rating` | `rating` | `rating` | `rating` | `supplierRating` (supplier) | none |
| reviews | `reviewsCount` | `reviewCount` | `reviewCount` | `productRatingCount` | `ratingCount` | `review_count` | `reviewCount` | `reviewCount` | `reviewCount` | `supplierReviewCount` | none |
| sales_signal | `salesVolume`, best-seller rank | none | `badges` | `soldCount` | none | `estimated_monthly_sales` (estimate) | `soldCount` | `soldCount` | `soldCountText` | `soldCount`, `minOrderQuantity` | `seller.salesText` (seller total) |
| seller | `sellerName` | `sellerName` | Target | `seller` | `source` | store | `sellerName` | `sellerName` | `sellerName` | `supplierName` | `seller.name` |
| availability | `inStock`, `shippingInformation` | `availabilityStatus`, `fulfillmentText` | `shippingMessage`, `pickupMessage` | `shippingCost`, `itemLocation` | `delivery` | `available` | `stock` | `inStock` | none | `leadTimes` | `availableQuantity` |
| url | `url` | `url` | `url` | `itemUrl` | `productUrl` | `product_url` | `url` | `url` | `productUrl` | `url` | `productUrl` |
| upc_or_model | product rows: `productDetails` | `upc`, `model` | `upc` | from title | none | variant SKU | none | none | none | none | `attributes.Modelo` |

Leave a cell empty when the source has no value; never fill it from another row.

## Same product across marketplaces

Treat two rows as the same product only when one of these holds, in this order:
1. Same UPC or GTIN (Walmart and Target product rows, Amazon `productDetails`).
2. Same brand and the same model number in both titles (for example `BES870XL`, `EC685`).
3. The user gave the exact links.

Similar titles alone are not a match: bundles, colours, sizes, refurbished units and "compatible with" accessories differ in price for real reasons. When you are not sure, show the row separately and mark it "possible match".

## Price comparison rules

- Compare prices only in the same currency. Do not convert currencies unless the user asks; then say the rate is approximate and the date.
- Add shipping when the row gives it (eBay `shippingCost`, Amazon offer `shippingPrice`) and show "price + shipping" as its own column.
- Sponsored placements (`isSponsored`, `sponsored`) are ads; keep them but mark them.
- A was-price is the retailer's own reference price, not proof of a discount.

## CSV export

When asked for a file, write `<keyword>-products-<YYYY-MM-DD>.csv` in the working folder (or as a downloadable file in chat) with the normalized columns above in that order, plus `sponsored`, `position`, `search_query` and `checked_at` (the row's scrape time). One row per product per marketplace.
