# Competitive Landscape Examples

Understand who shares the shelves with your brand and how prices compare across stores and markets.

> If you haven't yet, run the [name-variant discovery query](./distribution.md#discover-your-brands-name-variants-in-the-data) first to confirm you're matching the right SKUs.

---

## Shelf price range for a category in a market

Cheapest, average, and most expensive prices for a given category — useful for positioning your own SKUs.

```sql
SELECT
    category,
    subcategory,
    COUNT(*)                    AS listings,
    MIN(price)                  AS min_price,
    ROUND(AVG(price), 2)        AS avg_price,
    MAX(price)                  AS max_price,
    APPROX_PERCENTILE(price, 0.5)  AS median_price
FROM menuitems m
JOIN stores s USING (store_id)
WHERE s.state = 'CA'           -- ⬅ swap state
  AND m.category = 'Vapor Pens' -- ⬅ swap category
  AND m.price > 0
GROUP BY 1, 2
ORDER BY listings DESC;
```

---

## Shelf-mate report: brands sharing your category

Which other brands appear in the same `category` / `subcategory` as you, and at how many overlapping stores.

```sql
WITH brand_patterns AS (
  SELECT column1 AS pattern FROM VALUES
    ('%stiiizy%'),
    ('%stizzy%')
),
my_stores AS (
    SELECT DISTINCT m.store_id, m.category, m.subcategory
    FROM menuitems m
    WHERE EXISTS (
        SELECT 1 FROM brand_patterns bp WHERE m.brand_name ILIKE bp.pattern
    )
)
SELECT
    m.brand_name              AS other_brand,
    m.category,
    m.subcategory,
    COUNT(DISTINCT m.store_id) AS shared_stores,
    ROUND(AVG(m.price), 2)    AS avg_shelf_price
FROM menuitems m
JOIN my_stores ms
  ON m.store_id    = ms.store_id
 AND m.category    = ms.category
 AND m.subcategory = ms.subcategory
WHERE NOT EXISTS (
    SELECT 1 FROM brand_patterns bp WHERE m.brand_name ILIKE bp.pattern
)
GROUP BY 1, 2, 3
HAVING shared_stores >= 5    -- ⬅ tune to noise floor
ORDER BY shared_stores DESC
LIMIT 50;
```

---

## Share-of-shelves leaderboard for a category

Top brands by how many stores carry them, within a category and market.

```sql
SELECT
    brand_name,
    COUNT(DISTINCT store_id)   AS store_count,
    COUNT(DISTINCT product_id) AS distinct_skus,
    ROUND(AVG(price), 2)       AS avg_price
FROM menuitems m
JOIN stores s USING (store_id)
WHERE s.state = 'CA'              -- ⬅ swap state
  AND m.category = 'Edibles'      -- ⬅ swap category
  AND brand_name IS NOT NULL
GROUP BY 1
ORDER BY store_count DESC
LIMIT 25;
```

---

## Price percentile for your brand in a category

Where does your brand sit in the price distribution? Helpful for premium / value positioning conversations.

```sql
WITH brand_patterns AS (
  SELECT column1 AS pattern FROM VALUES
    ('%stiiizy%'),
    ('%stizzy%')
),
category_prices AS (
    SELECT price
    FROM menuitems m
    JOIN stores s USING (store_id)
    WHERE s.state    = 'CA'                -- ⬅ swap state
      AND m.category = 'Vapor Pens'        -- ⬅ swap category
      AND m.price > 0
),
my_prices AS (
    SELECT m.price
    FROM menuitems m
    JOIN stores s USING (store_id)
    WHERE s.state    = 'CA'
      AND m.category = 'Vapor Pens'
      AND m.price > 0
      AND EXISTS (
          SELECT 1 FROM brand_patterns bp WHERE m.brand_name ILIKE bp.pattern
      )
)
SELECT
    (SELECT ROUND(AVG(price), 2)                              FROM my_prices)       AS my_avg_price,
    (SELECT ROUND(APPROX_PERCENTILE(price, 0.5), 2)           FROM category_prices) AS category_median,
    (SELECT ROUND(APPROX_PERCENTILE(price, 0.25), 2)          FROM category_prices) AS category_p25,
    (SELECT ROUND(APPROX_PERCENTILE(price, 0.75), 2)          FROM category_prices) AS category_p75;
```

---

## Retailer view: category pricing at all stores within 10 miles

For a dispensary scoping their neighborhood. Replace the lat/lng with your store's coordinates.

```sql
SELECT
    s.name                         AS competitor_store,
    s.city,
    ROUND(ST_DISTANCE(s.location, ST_POINT(-122.4194, 37.7749)) / 1609.34, 2) AS miles_away,
    m.category,
    COUNT(*)                       AS skus,
    ROUND(AVG(m.price), 2)         AS avg_price,
    MIN(m.price)                   AS min_price,
    MAX(m.price)                   AS max_price
FROM menuitems m
JOIN stores s USING (store_id)
WHERE ST_DWITHIN(s.location, ST_POINT(-122.4194, 37.7749), 16093.4)    -- 10 miles
  -- ⬅ swap longitude, latitude, and radius (meters)
  AND m.category IN ('Flower', 'Vapor Pens', 'Edibles')                 -- ⬅ trim categories
  AND m.price > 0
GROUP BY 1, 2, 3, 4
ORDER BY miles_away, m.category;
```

---

## Outlier prices

Items priced above their `original_price` by more than X percent — usually a scrape artifact or a misconfigured menu.

```sql
SELECT
    m.menu_item_id,
    m.product_name,
    m.brand_name,
    s.name AS store_name,
    s.state,
    m.price,
    m.original_price,
    ROUND(((m.price - m.original_price) / NULLIF(m.original_price, 0)) * 100, 1) AS pct_over_original
FROM menuitems m
JOIN stores s USING (store_id)
WHERE m.original_price IS NOT NULL
  AND m.original_price > 0
  AND m.price > m.original_price * 1.10    -- ⬅ tune the threshold
ORDER BY pct_over_original DESC
LIMIT 100;
```
