# Menu Data Product

Menu Data is a Vault data product built on **web-scraped dispensary menus** across North America. Where Retailer, Bridge, and Insights show you what *sold*, Menu Data shows you what's **on the shelf right now** — every store, every product, every active deal, refreshed nightly.

## Who this is for

**For brands** — understand exactly where you're carried, identify white-space accounts, see how your shelf prices compare to MSRP and to competitors, and watch which SKUs are turning fastest where.

**For retailers** — scope nearby competitors' menus: their pricing by category, the deals they're running, and which categories move at which kinds of stores in your area.

## What's included

Four secure views in your shared database under the `PUBLIC` schema:

| View | Grain | What it answers |
|---|---|---|
| `STORES` | One row per dispensary | Where are stores? How big? How fast do they move? |
| `MENUITEMS` | One row per product per store | What's on the menu, at what price, in what stock? |
| `DEALS` | One row per promotion | What promotions are stores running, on what? |
| `EVENTS` | One row per menu change | What changed, when? Launches, delistings, price moves, low stock, doors won and lost — full history since May 2026 |

See the [Schema reference](./schema.md) for column-by-column details.

## How the views relate

```
STORES ─ store_id ─< MENUITEMS
STORES ─ store_id ─< DEALS
STORES ─ store_id ─< EVENTS >─ menu_item_id ─ MENUITEMS
```

- `MENUITEMS`, `DEALS` and `EVENTS` all link to `STORES` via `STORE_ID`.
- `EVENTS` links to `MENUITEMS` via `MENU_ITEM_ID` while the listing is still on a menu.
- `DEALS` and `MENUITEMS` aren't directly joined; relate them by `BRANDS` (after `LATERAL FLATTEN`) plus `STATE`.

## Where to find the data

Menu Data lives in the **`PUBLIC`** schema of your shared database. This differs from Retailer / Bridge / Insights, which use product-specific schemas (`retailer_share`, etc.). After connecting, set context like:

```sql
USE DATABASE <name_you_gave_the_share>;   -- the alias you chose when accepting
USE SCHEMA PUBLIC;
SHOW VIEWS;
SELECT * FROM STORES LIMIT 10;
```

See [Snowflake Basics](../snowflake-basics.md) for connection examples in Python, Node.js, and other languages.

## Your first query

Brand-side example — every store in California carrying STIIIZY products, with a SKU count per store. **Swap the brand pattern and state for your own.**

```sql
WITH brand_patterns AS (
  SELECT column1 AS pattern FROM VALUES
    ('%stiiizy%'),   -- ⬅ start with your best-guess spelling
    ('%stizzy%'),    -- ⬅ add common typos
    ('%stiiiizy%')   -- ⬅ etc.
)
SELECT
    s.name,
    s.city,
    s.state,
    s.last_crawl_utc,
    COUNT(*) AS sku_count
FROM MENUITEMS m
JOIN STORES s USING (store_id)
WHERE EXISTS (
    SELECT 1 FROM brand_patterns bp WHERE m.brand_name ILIKE bp.pattern
)
  AND s.state = 'CA'    -- ⬅ swap the state
GROUP BY 1, 2, 3, 4
ORDER BY sku_count DESC;
```

**Not sure how your brand name is spelled in the data?** Run the [name-variant discovery query](./examples/distribution.md#discover-your-brands-name-variants-in-the-data) first — web-scraped names vary a lot.

## Common use cases

**Brand-side**
- Distribution mapping — every store carrying your brand, by market
- White-space — stores in a state that *don't* carry you yet
- Competitor reach — your store count vs. brand X / Y / Z
- Shelf-price benchmarking — your prices vs. peers in the same category
- New-account leads — stores you don't yet sell to in a target market
- Doors won and lost — month-by-month net distribution, plus the stores that just dropped you
- Launch tracking — how fast a new SKU spread, store by store

**Retailer-side**
- Nearby-store pricing benchmarks by category
- What promotions competitors within X miles are running
- Category mix and velocity at peer stores
- Stores in your area carrying brands you might want to negotiate for
- Price-cut radar — which nearby competitors are marking down which categories, and whether the cuts stick

See the example files below.

## A note on data quality

Menu Data is **web-scraped**. That has implications you should know about:

- **Brand names vary.** "STIIIZY", "Stiiizy", "stiizy" can all appear. Always use `ILIKE` with multi-variant patterns, never `brand_name = '...'`.
- **`LAST_CRAWL_UTC` and `LAST_SEEN` matter.** Old rows reflect old observations. Filter when freshness matters.
- **Products disappear and reappear.** A menu can lose a SKU one day and have it back the next. `LAST_SEEN` is your truth.
- **Prices reflect scrape time, not transaction time.** Use as menu intelligence, not as a record of what was actually paid.
- **Some fields are nullable.** Especially `RATING`, `GOOGLE_PLACE_ID`, `DESCRIPTION`, and `INVENTORY_HISTORY` for newly-seen items.
- **`EVENTS` records coverage changes too.** When a store is first tracked, its whole menu "arrives" as `PRODUCT_ADDED` / `BRAND_ADDED` events on one day. Older events can also point at store listings that have since been re-keyed or closed. See the [`EVENTS` notes](./schema.md#view-events) before trending distribution over time.

## Example queries

Ready-to-run queries organized by what you're trying to answer:

- [Distribution](./examples/distribution.md) — Where am I carried? Where am I not? Geographic reach.
- [Competitive landscape](./examples/competitive-landscape.md) — Who shares shelves with me? How do prices compare?
- [Promotions](./examples/promotions.md) — What deals are running on my brand, my competitors, in my area?
- [Velocity and stock](./examples/velocity-and-stock.md) — What's turning fast? What's out of stock? Inventory trends.
- [Menu changes](./examples/menu-changes.md) — Doors won and lost, launch rollouts, price moves and markdowns, SKUs that keep running low.

## Further reading

- [Schema reference](./schema.md) — Full column definitions for every view
- [Snowflake basics](../snowflake-basics.md) — Connection setup and language examples
- [Glossary](../glossary.md) — Velocity score, days-on-hand, on-shelf, fuzzy brand match
