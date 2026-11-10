# Menu Data Dictionary

The Menu Data product exposes four secure views in your shared database, all under the `PUBLIC` schema (unlike Retailer / Bridge / Insights which use product-suffixed schemas).

| View | Grain | Purpose |
|---|---|---|
| `stores` | One row per dispensary store | Store directory: identity, location, velocity, type |
| `menuitems` | One row per product per store | Every product currently on a dispensary menu |
| `deals` | One row per promotion | Active and archived promotions at stores |
| `events` | One row per menu change | Full history of menu changes: products, prices, stock, brands, new stores |

## View: `STORES`

| Column | Type | Description | Example |
|---|---|---|---|
| STORE_ID | TEXT | Unique store identifier | abc123 |
| NAME | TEXT | Dispensary name | Green Leaf Co |
| SLUG | TEXT | URL-safe identifier | green-leaf-co |
| ADDRESS | TEXT | Street address | 123 Main St |
| CITY | TEXT | City | San Francisco |
| STATE | TEXT | State / region | CA |
| COUNTRY | TEXT | ISO country code | US |
| ZIPCODE | TEXT | Postal code | 94110 |
| GOOGLE_PLACE_ID | TEXT | Google Maps place id (when known) | ChIJ... |
| LOCATION | GEOGRAPHY | Store coordinates (lon/lat). See note. | `POINT(-122.41 37.77)` |
| TIMEZONE | TEXT | IANA timezone | America/Los_Angeles |
| SOURCE | TEXT | Crawl source identifier | weedmaps |
| LAST_CRAWL_UTC | TIMESTAMP_NTZ | When the store record was last refreshed | 2026-05-14 03:12:00 |
| DISPENSARY_URL | TEXT | Public menu URL | https://… |
| PRODUCT_COUNT | NUMBER | Distinct products on the menu right now | 247 |
| BRAND_COUNT | NUMBER | Distinct brands on the menu right now | 38 |
| AVG_DAILY_UNITS_SOLD | NUMBER | Store-level total daily units (modeled) | 412 |
| AVG_DAILY_SALES | NUMBER | Store-level total daily sales (modeled) | 18750 |
| VELOCITY_SCORE | FLOAT | Relative velocity score (Headset-modeled) | 78.4 |
| VELOCITY_BAND | TEXT | One of `High`, `Medium`, `Low` | High |
| STOREFRONT | BOOLEAN | Has a physical storefront | true |
| DELIVERY | BOOLEAN | Offers delivery | true |
| TYPES | ARRAY | License / store types | `["recreational","medical"]` |
| RATING | FLOAT | Public rating (0–5) | 4.6 |
| WEBSITE | TEXT | Marketing website | https://… |
| PHONE_NUMBER | TEXT | Contact phone | 415-555-0123 |
| LOGO | TEXT | URL to store logo image | https://… |
| DESCRIPTION | TEXT | Free-text description | Family-owned… |

**Notes:**
- `LOCATION` is a native Snowflake `GEOGRAPHY`. You can pass it directly to `ST_*` functions — no `TO_GEOGRAPHY` conversion needed. By default Snowflake renders `GEOGRAPHY` values as GeoJSON text (`{"type":"Point","coordinates":[lon, lat]}`); to see WKT instead, run `ALTER SESSION SET GEOGRAPHY_OUTPUT_FORMAT='WKT'`.
- `AVG_DAILY_UNITS_SOLD` / `AVG_DAILY_SALES` here are **store-level modeled totals** across all products on the menu — they are not transactional sales from a POS.
- `VELOCITY_BAND` is bucketed (`High` / `Medium` / `Low`); use it for filtering and grouping. `VELOCITY_SCORE` is the underlying numeric.
- `LAST_CRAWL_UTC` is the freshness signal for the entire row. If it's old, the velocity numbers and product/brand counts are correspondingly old.

## View: `MENUITEMS`

| Column | Type | Description | Example |
|---|---|---|---|
| MENU_ITEM_ID | TEXT | Unique menu-item identifier | mi_abc123 |
| PRODUCT_ID | NUMBER | Headset internal product id | 4598221 |
| PRODUCT_NAME | TEXT | Canonicalized product name | Blue Dream 1g Cart |
| ORIGINAL_PRODUCT_NAME | TEXT | Original scraped name (pre-canonicalization) | Blue Dream Cartridge 1G |
| BRAND_NAME | TEXT | Brand name as scraped (fuzzy — see note) | STIIIZY |
| BRAND_ID | TEXT | Brand identifier | b_4421 |
| BRAND_LOGO | TEXT | URL to brand logo | https://… |
| PHOTO_URL | TEXT | Product photo URL | https://… |
| CATEGORY | TEXT | Top-level category | Vapor Pens |
| SUBCATEGORY | TEXT | Subcategory | Cartridge |
| LABEL | TEXT | Display label (e.g., Sativa / Indica / Hybrid) | Hybrid |
| PRICE | NUMBER | Current shelf price | 45 |
| ORIGINAL_PRICE | NUMBER | Pre-discount price (if any) | 55 |
| ON_SALE | BOOLEAN | Flagged as on-sale on the menu | true |
| DISCOUNT_LABEL | TEXT | Free-text discount label | 20% off |
| QUANTITY_IN_STOCK | NUMBER | Current observed stock count | 12 |
| AVG_DAILY_UNITS_SOLD | NUMBER | Item-level modeled daily units | 3 |
| AVG_DAILY_SALES | NUMBER | Item-level modeled daily sales | 135 |
| EST_DAYS_ON_HAND | NUMBER | Projected days until stockout | 4 |
| INVENTORY_HISTORY | ARRAY | Time-series of observed stock. See note. | `[{"DATE":"2026-05-14","VALUE":8}, …]` |
| STORE_ID | TEXT | FK to `stores.store_id` | abc123 |
| STATE | TEXT | Denormalized state from the store | CA |
| LOCATION | GEOGRAPHY | Denormalized store coordinates | `POINT(-122.41 37.77)` |
| LAST_SEEN | TIMESTAMP_NTZ | When this item was last observed | 2026-05-14 03:12:00 |

**Notes:**
- `BRAND_NAME` is the value as scraped. The same brand can appear under multiple spellings ("STIIIZY", "Stiiizy", "stiizy"). **Always use `ILIKE` with patterns, never `=`**. See [Distribution examples](./examples/distribution.md) for the canonical multi-variant CTE pattern.
- `INVENTORY_HISTORY` is an `ARRAY` of `{"DATE": "YYYY-MM-DD", "VALUE": <stock>}` objects, daily granularity. **Days may be missing** — a gap means no observation that day, not necessarily zero stock. Flatten with `LATERAL FLATTEN`:
  ```sql
  SELECT
      m.menu_item_id,
      m.product_name,
      ih.value:DATE::date    AS history_date,
      ih.value:VALUE::number AS stock_level
  FROM menuitems m,
       LATERAL FLATTEN(input => m.inventory_history) ih
  WHERE m.menu_item_id = 'some-id'
  ORDER BY history_date;
  ```
- `AVG_DAILY_UNITS_SOLD` here is **per-item**, not store-level. Don't confuse with `stores.avg_daily_units_sold`.
- `EST_DAYS_ON_HAND` divides `QUANTITY_IN_STOCK` by recent sell rate; treat very small or very large values as approximate.
- `ORIGINAL_PRODUCT_NAME` is the raw scraped string; `PRODUCT_NAME` is canonicalized. They usually match.

## View: `DEALS`

| Column | Type | Description | Example |
|---|---|---|---|
| DEAL_ID | TEXT | Unique deal identifier | d_abc123 |
| STORE_ID | TEXT | FK to `stores.store_id` | abc123 |
| TITLE | TEXT | Deal headline | 20% off all edibles |
| DESCRIPTION | TEXT | Free-text deal description | Valid Mon–Fri until 6pm |
| PICTURE | TEXT | URL to deal image | https://… |
| DISCOUNT_TYPE | TEXT | One of `BOGO`, `BUNDLE`, `DOLLAR_OFF`, `FIXED_PRICE`, `FREE_GIFT`, `MULTI_BUY`, `OTHER`, `PERCENTAGE_OFF`, `STARTING_AT`, `TIERED_SPEND` | PERCENTAGE_OFF |
| DISCOUNT_AMOUNT | TEXT | Free-text amount. See note. | 20% |
| RESTRICTIONS | TEXT | Free-text restrictions | Limit 1 per customer |
| STOREWIDE | BOOLEAN | Applies to the entire store | false |
| FIRST_TIME_CUSTOMER | BOOLEAN | Only for first-time customers | false |
| BRANDS | ARRAY | Brand names the deal applies to | `["STIIIZY","Wyld"]` |
| PRODUCT_CATEGORIES | ARRAY | Categories the deal applies to | `["Edible"]` |
| CITY | TEXT | Denormalized store city | San Francisco |
| STATE | TEXT | Denormalized store state | CA |
| LOCATION | GEOGRAPHY | Denormalized store coordinates | `POINT(-122.41 37.77)` |
| IS_ARCHIVED | BOOLEAN | True once the deal is no longer active | false |
| ARCHIVED_AT | TIMESTAMP_NTZ | When the deal was archived | 2026-05-01 12:00:00 |
| CLAIMED_COUNT | NUMBER | Engagement counter | 412 |

**Notes:**
- `DISCOUNT_AMOUNT` is **TEXT, not numeric**. The scraper preserves whatever format the menu uses — values like `"20%"`, `"$5"`, `"BOGO"`, `"Buy 2 get 1"` all appear. Don't blindly cast to number; filter to a single `discount_type` first.
- Filter `WHERE is_archived = false` to limit to currently-active deals.
- `BRANDS` and `PRODUCT_CATEGORIES` are arrays; flatten with `LATERAL FLATTEN` to filter or join.
- `PRODUCT_CATEGORIES` uses the same canonical category vocabulary as `menuitems.category` (e.g. `Edible`, `Vapor Pens`, `Pre-Roll`, `Flower`, `Concentrates`). You can filter or join the two views on category names without normalization.
- `CLAIMED_COUNT` is most meaningful as a relative signal within a market — absolute counts depend on the source.
- A `storewide = true` deal usually has empty `BRANDS` and `PRODUCT_CATEGORIES` arrays.

## View: `EVENTS`

A change log of every menu: one row each time a product is added or removed, a price moves, stock runs low, a store picks up or drops a brand, or a new store starts being tracked. History starts **2026-05-16** and is never trimmed; new days are added nightly.

| Column | Type | Description | Example |
|---|---|---|---|
| EVENT_ID | NUMBER | Unique event identifier | 7710552381904417332 |
| EVENT_DATE | DATE | Crawl day the change was observed | 2026-11-08 |
| EVENT_TYPE | TEXT | One of `PRODUCT_ADDED`, `PRODUCT_REMOVED`, `PRICE_INCREASE`, `PRICE_DECREASE`, `LOW_STOCK`, `BRAND_ADDED`, `BRAND_REMOVED`, `STORE_ADDED` | PRICE_DECREASE |
| STORE_ID | TEXT | Store the change happened at. Joins to `stores.store_id` (see note) | abc123 |
| STORE_NAME | TEXT | Store name | Green Leaf Co |
| MENU_ITEM_ID | TEXT | Listing the change is about. Joins to `menuitems.menu_item_id` while the listing is live | mi_abc123 |
| PRODUCT_NAME | TEXT | Product name. For `BRAND_*` events a summary such as `5 products added` | Blue Dream 1g Cart |
| BRAND_NAME | TEXT | Brand name (fuzzy — use `ILIKE`) | STIIIZY |
| BRAND_ID | TEXT | Brand identifier, same as `menuitems.brand_id` | stiiizy |
| CATEGORY | TEXT | Top-level category | Vapor Pens |
| SOURCE | TEXT | Crawl source of the menu | weedmaps |
| STATE | TEXT | State / province of the store | CA |
| CITY | TEXT | City of the store | San Francisco |
| COUNTY | TEXT | County of the store | San Francisco County |
| LOCATION | GEOGRAPHY | Store coordinates | `POINT(-122.41 37.77)` |
| OLD_VALUE | FLOAT | Price before (`PRICE_*`) or stock before (`LOW_STOCK`) | 45 |
| NEW_VALUE | FLOAT | Price after (`PRICE_*`) or stock after (`LOW_STOCK`) | 39.99 |
| DATA | OBJECT | Type-specific details (see below) | `{"oldPrice":45,"newPrice":39.99,…}` |

**What triggers each event type**

| Event type | Fires when | `OLD_VALUE` → `NEW_VALUE` |
|---|---|---|
| `PRODUCT_ADDED` | A listing appears on a store's menu, including when it comes back after selling out | — |
| `PRODUCT_REMOVED` | A listing leaves a store's menu and doesn't come straight back | — |
| `PRICE_INCREASE` / `PRICE_DECREASE` | The shelf price of a listing changes (sale markdowns included) | old price → new price |
| `LOW_STOCK` | Stock drops from above 5 units to 5 or fewer | old stock → new stock |
| `BRAND_ADDED` | A store goes from zero of a brand's listings to at least one, including a return after the brand sold out | — |
| `BRAND_REMOVED` | A store goes from at least one of a brand's listings to zero | — |
| `STORE_ADDED` | A store's menu appears in the data for the first time | — |

**`DATA` by event type** (camelCase keys; read with `data:key::type`)

| Event type | Keys |
|---|---|
| `PRODUCT_ADDED`, `PRODUCT_REMOVED` | `menuItemId`, `productName`, `brandId`, `brandName`, `category` |
| `PRICE_INCREASE`, `PRICE_DECREASE` | the above plus `oldPrice`, `newPrice` |
| `LOW_STOCK` | the above plus `oldQuantity`, `newQuantity`, `threshold` (5) |
| `BRAND_ADDED`, `BRAND_REMOVED` | `brandId`, `brandName`, `productCount`, `items` — an array of `{menuItemId, name, category, price}`, one per listing that arrived or left |
| `STORE_ADDED` | `storeId`, `storeName`, `city`, `state`, `dispensaryUrl`, `website`, `phoneNumber`, `logo` |

**Notes:**
- **Filter on `EVENT_DATE` first.** The view is laid out by date, so a date range keeps queries fast on a table that grows by roughly 27 million rows a month.
- `EVENT_DATE` is the crawl day the change was **observed**, not a transaction time. When a store's whole menu goes dark, its removals are confirmed after four days and dated back to the day the menu disappeared.
- `STORE_NAME`, `CITY` and `COUNTY` reflect the store as it is listed today. About a quarter of events before September 2026 (under one in ten since) belong to listings that were since re-keyed or closed: their `STORE_ID` no longer appears in `STORES`, and `STORE_NAME` / `CITY` / `COUNTY` are NULL. `STATE`, `SOURCE` and `LOCATION` are always the values at event time — prefer them for market-level history, and use `LEFT JOIN stores` when you need store attributes.
- **Coverage changes look like activity.** When a store or menu platform is first tracked, every product and brand on it "arrives" at once, producing spikes of `PRODUCT_ADDED` / `BRAND_ADDED` (and the reverse when coverage drops). For distribution trends, skip a store's first week after its `STORE_ADDED` event — see [Doors gained and lost](./examples/menu-changes.md#doors-gained-and-lost-by-month).
- Products already on a menu at 2026-05-16 have no `PRODUCT_ADDED` row at that store.
- **Adds include comebacks.** A listing that sells out and returns gets a new `PRODUCT_ADDED`, and a brand that returns to a store gets a new `BRAND_ADDED`: over the 30 days to 2026-11-09, at least 40% of added listings and 53% of brand pickups were at stores that had carried them earlier. For genuinely new placements, keep only a store's first event for a product — see [Launch rollout curve](./examples/menu-changes.md#launch-rollout-curve).
- A handful of dates have no events at all (2026-07-13, 07-22, 08-28, 09-16, 10-09): menus weren't captured those nights, so the changes are recorded on the following day. Expect a gap followed by a roughly doubled day when charting daily counts.
- `DATA` is NULL on most events before 2026-05-28; use `OLD_VALUE` / `NEW_VALUE` for price and stock history across the full range.
- `PRICE_*` events track the shelf price, so a temporary sale shows as a decrease followed by an increase back. See [Temporary markdowns vs. price cuts that stuck](./examples/menu-changes.md#temporary-markdowns-vs-price-cuts-that-stuck).
- `MENU_ITEM_ID` on a `STORE_ADDED` event is just an example listing from that store; `BRAND_*` events use one representative listing — the full list is in `data:items`.

## Joining the views

```
stores ─ store_id ─< menuitems
stores ─ store_id ─< deals
stores ─ store_id ─< events >─ menu_item_id ─ menuitems
```

To relate `deals` to specific `menuitems`, join `deals.brands` (after `LATERAL FLATTEN`) against `menuitems.brand_name` within the same `state` — there's no per-item deal id.

`events.menu_item_id` joins to `menuitems` only while the listing is still on a menu; removed listings keep their history in `events` alone.
