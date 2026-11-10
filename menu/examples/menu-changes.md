# Menu Changes Examples

`EVENTS` is the change log behind the other three views: one row every time a menu gains or drops a product, moves a price, runs low on stock, picks up or loses a brand, or a new store starts being tracked. Use it for anything that needs **history** — trends, rollouts, door wins and losses, price wars — rather than a snapshot of today.

History starts **2026-05-16** and is kept indefinitely.

> If you haven't yet, run the [name-variant discovery query](./distribution.md#discover-your-brands-name-variants-in-the-data) first to confirm you're matching the right brand spellings.

**Before you start**

- **Always filter on `event_date`.** It's how the view is laid out on disk, so a date filter makes every query below fast and cheap.
- **Use the event's own `state`, `source` and `location`** for market analysis. About a quarter of events from before September 2026 (under one in ten since) point at a `store_id` that is no longer in `STORES` (the listing was re-keyed or went dark). Those rows have no `store_name` or `city`, but they still carry their state and coordinates; an inner join to `STORES` drops them.
- **Coverage changes look like activity.** When a store is first tracked, every product and brand on its menu "arrives" at once. The door-tracking and rollout queries below skip each store's first week for that reason.
- **Adds include comebacks.** `PRODUCT_ADDED` and `BRAND_ADDED` fire every time something returns to a menu, including after it sold out. Over the 30 days to 2026-11-09, at least 40% of added listings and 53% of brand pickups were at a store that had carried them earlier. To count genuinely new placements, keep only a store's first event for a product, as the rollout and spread queries below do.

---

## Daily change pulse for a market

How much is moving on menus in a state, day by day. A good first look at what the data contains.

```sql
SELECT
    event_date,
    COUNT_IF(event_type = 'PRODUCT_ADDED')   AS products_added,
    COUNT_IF(event_type = 'PRODUCT_REMOVED') AS products_removed,
    COUNT_IF(event_type = 'PRICE_INCREASE')  AS price_increases,
    COUNT_IF(event_type = 'PRICE_DECREASE')  AS price_decreases,
    COUNT_IF(event_type = 'LOW_STOCK')       AS low_stock,
    COUNT(DISTINCT store_id)                 AS stores_with_changes
FROM events
WHERE state = 'MI'                                       -- ⬅ swap the state
  AND event_date >= DATEADD(day, -30, CURRENT_DATE())
GROUP BY event_date
ORDER BY event_date;
```

---

## Doors gained and lost, by month

Net distribution change for your brand. A door is gained when a store goes from zero of your SKUs to at least one (`BRAND_ADDED`) and lost when it goes back to zero (`BRAND_REMOVED`).

```sql
WITH brand_patterns AS (
  SELECT column1 AS pattern FROM VALUES
    ('%stiiizy%'),    -- ⬅ swap your variants
    ('%stizzy%')
),
first_tracked AS (
  SELECT store_id, event_date AS tracked_on
  FROM events
  WHERE event_type = 'STORE_ADDED'
)
SELECT
    DATE_TRUNC(month, e.event_date)                                        AS month,
    COUNT(DISTINCT IFF(e.event_type = 'BRAND_ADDED',   e.store_id, NULL)) AS doors_gained,
    COUNT(DISTINCT IFF(e.event_type = 'BRAND_REMOVED', e.store_id, NULL)) AS doors_lost,
    doors_gained - doors_lost                                              AS net_change
FROM events e
JOIN stores s USING (store_id)          -- stores still tracked today: skips re-keyed / closed listings
LEFT JOIN first_tracked f
       ON f.store_id = e.store_id
      AND e.event_date BETWEEN f.tracked_on AND DATEADD(day, 7, f.tracked_on)
WHERE e.event_type IN ('BRAND_ADDED', 'BRAND_REMOVED')
  AND f.store_id IS NULL                -- skip a store's onboarding week
  AND EXISTS (SELECT 1 FROM brand_patterns bp WHERE e.brand_name ILIKE bp.pattern)
GROUP BY 1
ORDER BY 1;
```

---

## Stores that dropped you in the last 30 days

The follow-up list for your sales team: where your brand went to zero SKUs, how many SKUs left in that drop, and whether you're back on the menu today.

```sql
WITH brand_patterns AS (
  SELECT column1 AS pattern FROM VALUES
    ('%stiiizy%'),
    ('%stizzy%')
),
on_menu_now AS (
  SELECT store_id, COUNT(*) AS skus_now
  FROM menuitems m
  WHERE EXISTS (SELECT 1 FROM brand_patterns bp WHERE m.brand_name ILIKE bp.pattern)
  GROUP BY 1
)
SELECT
    e.event_date                  AS dropped_on,
    s.name                        AS store_name,
    s.city,
    s.state,
    e.data:productCount::int      AS skus_dropped,
    COALESCE(n.skus_now, 0)       AS skus_on_menu_now,
    s.dispensary_url
FROM events e
JOIN stores s USING (store_id)
LEFT JOIN on_menu_now n USING (store_id)
WHERE e.event_type = 'BRAND_REMOVED'
  AND e.event_date >= DATEADD(day, -30, CURRENT_DATE())
  AND EXISTS (SELECT 1 FROM brand_patterns bp WHERE e.brand_name ILIKE bp.pattern)
ORDER BY skus_on_menu_now, dropped_on DESC;
```

`skus_on_menu_now = 0` is a door you still need to win back.

---

## What came in with a new placement

When a store picks up your brand, `data:items` lists every SKU that arrived that day, with the shelf price it went up at.

```sql
WITH brand_patterns AS (
  SELECT column1 AS pattern FROM VALUES
    ('%stiiizy%'),
    ('%stizzy%')
)
SELECT
    e.event_date,
    e.store_name,
    e.city,
    e.state,
    i.value:name::text     AS product,
    i.value:category::text AS category,
    i.value:price::number  AS launch_price
FROM events e,
     LATERAL FLATTEN(input => e.data:items) i
WHERE e.event_type = 'BRAND_ADDED'
  AND e.event_date >= DATEADD(day, -30, CURRENT_DATE())
  AND EXISTS (SELECT 1 FROM brand_patterns bp WHERE e.brand_name ILIKE bp.pattern)
ORDER BY e.event_date DESC, e.store_name, product;
```

`BRAND_REMOVED` carries the same `data:items` list for the SKUs that left. A store coming back after your brand sold out there shows up here too; the doors query above counts that as a door lost and regained.

---

## Launch rollout curve

How fast a product spread: the day each store first listed it, and the cumulative store count per state.

```sql
WITH first_tracked AS (
  SELECT store_id, event_date AS tracked_on
  FROM events
  WHERE event_type = 'STORE_ADDED'
),
first_seen AS (
  -- each store's first event for the product; ties put the add last, so a
  -- same-day removal of another size still reads as "already carried it"
  SELECT store_id, state, event_date, event_type
  FROM events
  WHERE brand_name   ILIKE '%stiiizy%'        -- ⬅ your brand
    AND product_name ILIKE '%blue dream%'     -- ⬅ your product
    AND event_type IN ('PRODUCT_ADDED', 'PRODUCT_REMOVED', 'PRICE_INCREASE', 'PRICE_DECREASE', 'LOW_STOCK')
  QUALIFY ROW_NUMBER() OVER (PARTITION BY store_id
                             ORDER BY event_date, IFF(event_type = 'PRODUCT_ADDED', 1, 0)) = 1
)
SELECT
    fs.state,
    fs.event_date                                                         AS first_listed_on,
    COUNT(*)                                                              AS new_stores,
    SUM(COUNT(*)) OVER (PARTITION BY fs.state ORDER BY fs.event_date)     AS stores_to_date
FROM first_seen fs
LEFT JOIN first_tracked f
       ON f.store_id = fs.store_id
      AND fs.event_date BETWEEN f.tracked_on AND DATEADD(day, 7, f.tracked_on)
WHERE fs.event_type = 'PRODUCT_ADDED'   -- first seen being added, not already on the menu
  AND f.store_id IS NULL                -- skip a store's onboarding week
GROUP BY 1, 2
ORDER BY 1, 2;
```

A store counts on the day it first lists the product. Stores whose first event for it is a removal or a price move already carried it and are left out, as are stores in their first week of tracking. History starts 2026-05-16, so the curve is clearest for launches after that date.

---

## Fastest-spreading products in your category

Which products reached the most stores that had never carried them, in a market over the last two weeks: new launches and distribution pushes alike, with restocks left out. Excludes your own brand so it reads as a competitor watch list.

```sql
WITH brand_patterns AS (
  SELECT column1 AS pattern FROM VALUES
    ('%stiiizy%'),
    ('%stizzy%')
),
first_tracked AS (
  SELECT store_id, event_date AS tracked_on
  FROM events
  WHERE event_type = 'STORE_ADDED'
),
first_seen AS (
  -- each store's first event for each product (see the rollout query above)
  SELECT store_id, brand_name, product_name, menu_item_id, event_date, event_type
  FROM events
  WHERE state = 'CA'                                     -- ⬅ swap the state
    AND category = 'Edible'                              -- ⬅ swap the category
    AND event_type IN ('PRODUCT_ADDED', 'PRODUCT_REMOVED', 'PRICE_INCREASE', 'PRICE_DECREASE', 'LOW_STOCK')
  QUALIFY ROW_NUMBER() OVER (PARTITION BY store_id, brand_name, product_name
                             ORDER BY event_date, IFF(event_type = 'PRODUCT_ADDED', 1, 0)) = 1
)
SELECT
    fs.brand_name,
    fs.product_name,
    COUNT(DISTINCT fs.store_id) AS new_stores,
    MIN(fs.event_date)          AS first_added,
    ROUND(MEDIAN(m.price), 2)   AS median_price_now
FROM first_seen fs
LEFT JOIN first_tracked f
       ON f.store_id = fs.store_id
      AND fs.event_date BETWEEN f.tracked_on AND DATEADD(day, 7, f.tracked_on)
LEFT JOIN menuitems m USING (menu_item_id)
WHERE fs.event_type = 'PRODUCT_ADDED'
  AND fs.event_date >= DATEADD(day, -14, CURRENT_DATE())
  AND f.store_id IS NULL                                 -- skip a store's onboarding week
  AND NOT EXISTS (SELECT 1 FROM brand_patterns bp WHERE fs.brand_name ILIKE bp.pattern)
GROUP BY 1, 2
HAVING COUNT(DISTINCT fs.store_id) >= 3
ORDER BY new_stores DESC
LIMIT 50;
```

---

## Your price moves, week by week

How often stores raised or cut your shelf prices, and by how much, per state.

```sql
WITH brand_patterns AS (
  SELECT column1 AS pattern FROM VALUES
    ('%stiiizy%'),
    ('%stizzy%')
)
SELECT
    DATE_TRUNC(week, event_date)                                   AS week,
    state,
    COUNT_IF(event_type = 'PRICE_INCREASE')                        AS increases,
    COUNT_IF(event_type = 'PRICE_DECREASE')                        AS decreases,
    ROUND(AVG((new_value - old_value) / NULLIF(old_value, 0)) * 100, 1) AS avg_pct_change,
    ROUND(MEDIAN(new_value - old_value), 2)                        AS median_dollar_change
FROM events e
WHERE event_type IN ('PRICE_INCREASE', 'PRICE_DECREASE')
  AND event_date >= DATEADD(month, -3, CURRENT_DATE())
  AND EXISTS (SELECT 1 FROM brand_patterns bp WHERE e.brand_name ILIKE bp.pattern)
GROUP BY 1, 2
ORDER BY 1, 2;
```

---

## Temporary markdowns vs. price cuts that stuck

Price events track the shelf price, so a weekend sale shows up as a decrease followed by an increase back. This splits a category's price cuts into **temporary** (back to the old price within 14 days) and **sticky** (not reverted).

```sql
WITH price_moves AS (
  SELECT
      menu_item_id,
      event_date,
      event_type,
      old_value,
      new_value,
      LEAD(event_date) OVER (PARTITION BY menu_item_id ORDER BY event_date) AS next_move_on,
      LEAD(new_value)  OVER (PARTITION BY menu_item_id ORDER BY event_date) AS next_price
  FROM events
  WHERE event_type IN ('PRICE_INCREASE', 'PRICE_DECREASE')
    AND state = 'CA'                                     -- ⬅ swap the state
    AND category = 'Vapor Pens'                          -- ⬅ swap the category
    AND event_date >= DATEADD(day, -120, CURRENT_DATE())
)
SELECT
    IFF(next_price = old_value AND DATEDIFF(day, event_date, next_move_on) <= 14,
        'Temporary markdown', 'Sticky cut')                     AS cut_type,
    COUNT(*)                                                    AS cuts,
    ROUND(AVG((old_value - new_value) / NULLIF(old_value, 0)) * 100, 1) AS avg_pct_off,
    ROUND(AVG(IFF(next_price = old_value, DATEDIFF(day, event_date, next_move_on), NULL)), 1) AS avg_days_until_reverted
FROM price_moves
WHERE event_type = 'PRICE_DECREASE'
  AND event_date <= DATEADD(day, -14, CURRENT_DATE())   -- give every cut 14 days to revert
GROUP BY 1;
```

For a sense of scale: across California vape pens from mid-May to October 2026, about four in five price cuts (82%) were back at full price within two weeks, typically the next day. A rising share of sticky cuts is the early sign of real price compression in a category.

---

## Retailer view: who's cutting prices near you

Competitor stores within 10 miles that cut the most prices in the last two weeks, by category.

```sql
SELECT
    e.store_name,
    e.city,
    e.category,
    COUNT(*)                                                    AS price_cuts,
    ROUND(AVG((e.old_value - e.new_value) / NULLIF(e.old_value, 0)) * 100, 1) AS avg_pct_cut,
    ROUND(ST_DISTANCE(ANY_VALUE(e.location), ST_POINT(-122.4194, 37.7749)) / 1609.34, 1) AS miles_away
FROM events e
WHERE e.event_type = 'PRICE_DECREASE'
  AND e.event_date >= DATEADD(day, -14, CURRENT_DATE())
  AND ST_DWITHIN(e.location, ST_POINT(-122.4194, 37.7749), 16093.4)   -- 10 miles
  -- ⬅ swap longitude, latitude, radius
GROUP BY 1, 2, 3
ORDER BY price_cuts DESC
LIMIT 50;
```

---

## Price history for a single SKU at a store

Every price move for one listing, oldest first. Grab a `menu_item_id` from `MENUITEMS`.

```sql
SELECT
    event_date,
    event_type,
    old_value AS old_price,
    new_value AS new_price,
    ROUND((new_value - old_value) / NULLIF(old_value, 0) * 100, 1) AS pct_change
FROM events
WHERE menu_item_id = 'some-menu-item-id'      -- ⬅ swap the listing
  AND event_type IN ('PRICE_INCREASE', 'PRICE_DECREASE')
ORDER BY event_date;
```

---

## How long SKUs last on menus

Days from a listing appearing to it being removed, per brand in a category. A short shelf life can mean fast sell-through on limited drops — or assortment churn.

```sql
WITH adds AS (
  SELECT menu_item_id, brand_name, event_date AS added_on
  FROM events
  WHERE event_type = 'PRODUCT_ADDED'
    AND state = 'MI'                        -- ⬅ swap the state
    AND category = 'Pre-Roll'               -- ⬅ swap the category
),
stints AS (
  SELECT
      a.menu_item_id,
      a.brand_name,
      a.added_on,
      MIN(r.event_date) AS removed_on
  FROM adds a
  LEFT JOIN events r
         ON r.menu_item_id = a.menu_item_id
        AND r.event_type = 'PRODUCT_REMOVED'
        AND r.event_date > a.added_on
  GROUP BY 1, 2, 3
)
SELECT
    brand_name,
    COUNT(*)                                           AS listings,
    COUNT(removed_on)                                  AS listings_removed,
    MEDIAN(DATEDIFF(day, added_on, removed_on))        AS median_days_on_menu
FROM stints
GROUP BY 1
HAVING COUNT(removed_on) >= 20
ORDER BY median_days_on_menu;
```

Listings still on the menu have `removed_on = NULL` and are left out of the median.

---

## SKUs that keep running low

Products that drop to 5 units or fewer most often across a market — a sell-through signal from the shelf. `data:threshold` holds the trigger level (5).

```sql
SELECT
    brand_name,
    product_name,
    category,
    COUNT(*)                 AS low_stock_events,
    COUNT(DISTINCT store_id) AS stores
FROM events
WHERE event_type = 'LOW_STOCK'
  AND state = 'MI'                                       -- ⬅ swap the state
  AND event_date >= DATEADD(day, -30, CURRENT_DATE())
GROUP BY 1, 2, 3
ORDER BY low_stock_events DESC
LIMIT 50;
```

Stores that show a fixed stock number rather than a real count (common on some marketplaces) rarely trigger `LOW_STOCK`, so compare brands within a market rather than across sources.

---

## Newly tracked stores

Stores that started appearing in the data recently, with their contact details from `data`.

```sql
SELECT
    event_date                 AS first_tracked_on,
    data:storeName::text       AS store_name,
    data:city::text            AS city,
    state,
    data:dispensaryUrl::text   AS dispensary_url,
    data:website::text         AS website,
    data:phoneNumber::text     AS phone_number
FROM events
WHERE event_type = 'STORE_ADDED'
  AND state = 'MI'                                       -- ⬅ swap the state
  AND event_date >= DATEADD(day, -60, CURRENT_DATE())
ORDER BY first_tracked_on DESC;
```

"First tracked" is when the store's menu entered the data — usually a new store or a newly covered menu platform, not necessarily an opening date.
