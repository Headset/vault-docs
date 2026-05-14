# Menu Data Dictionary

The Menu Data product exposes three secure views in your shared database, all under the `PUBLIC` schema (unlike Retailer / Bridge / Insights which use product-suffixed schemas).

| View | Grain | Purpose |
|---|---|---|
| `stores` | One row per dispensary store | Store directory: identity, location, velocity, type |
| `menuitems` | One row per product per store | Every product currently on a dispensary menu |
| `deals` | One row per promotion | Active and archived promotions at stores |

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
| PRODUCT_CATEGORIES | ARRAY | Categories the deal applies to | `["Edibles"]` |
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
- `CLAIMED_COUNT` is most meaningful as a relative signal within a market — absolute counts depend on the source.
- A `storewide = true` deal usually has empty `BRANDS` and `PRODUCT_CATEGORIES` arrays.

## Joining the views

```
stores ─ store_id ─< menuitems
stores ─ store_id ─< deals
```

To relate `deals` to specific `menuitems`, join `deals.brands` (after `LATERAL FLATTEN`) against `menuitems.brand_name` within the same `state` — there's no per-item deal id.
