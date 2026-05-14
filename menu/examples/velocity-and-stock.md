# Velocity and Stock Examples

Find your fastest movers, catch out-of-stock and low-stock SKUs, and trace inventory history for individual products.

> If you haven't yet, run the [name-variant discovery query](./distribution.md#discover-your-brands-name-variants-in-the-data) first to confirm you're matching the right SKUs.

---

## Your fastest-moving SKUs by store

Highest `avg_daily_units_sold` for your brand, broken out by store.

```sql
WITH brand_patterns AS (
  SELECT column1 AS pattern FROM VALUES
    ('%stiiizy%'),
    ('%stizzy%')
)
SELECT
    s.name AS store_name,
    s.state,
    m.product_name,
    m.subcategory,
    m.price,
    m.avg_daily_units_sold,
    m.avg_daily_sales,
    m.quantity_in_stock,
    m.est_days_on_hand
FROM menuitems m
JOIN stores s USING (store_id)
WHERE EXISTS (
    SELECT 1 FROM brand_patterns bp WHERE m.brand_name ILIKE bp.pattern
)
  AND m.avg_daily_units_sold IS NOT NULL
ORDER BY m.avg_daily_units_sold DESC
LIMIT 100;
```

---

## Currently out-of-stock listings for your brand

```sql
WITH brand_patterns AS (
  SELECT column1 AS pattern FROM VALUES
    ('%stiiizy%'),
    ('%stizzy%')
)
SELECT
    s.name AS store_name,
    s.city,
    s.state,
    m.product_name,
    m.subcategory,
    m.price,
    m.last_seen
FROM menuitems m
JOIN stores s USING (store_id)
WHERE EXISTS (
    SELECT 1 FROM brand_patterns bp WHERE m.brand_name ILIKE bp.pattern
)
  AND m.quantity_in_stock = 0
ORDER BY s.state, s.name, m.product_name;
```

---

## Low-stock alert list

Listings projected to stock out within 7 days based on `est_days_on_hand`.

```sql
WITH brand_patterns AS (
  SELECT column1 AS pattern FROM VALUES
    ('%stiiizy%'),
    ('%stizzy%')
)
SELECT
    s.name AS store_name,
    s.city,
    s.state,
    m.product_name,
    m.quantity_in_stock,
    m.est_days_on_hand,
    m.avg_daily_units_sold
FROM menuitems m
JOIN stores s USING (store_id)
WHERE EXISTS (
    SELECT 1 FROM brand_patterns bp WHERE m.brand_name ILIKE bp.pattern
)
  AND m.est_days_on_hand IS NOT NULL
  AND m.est_days_on_hand < 7      -- ⬅ swap threshold
  AND m.quantity_in_stock > 0
ORDER BY m.est_days_on_hand;
```

---

## Velocity-band mix at stores carrying your brand

Are the stores carrying you mostly high-velocity, or are you concentrated in slower stores?

```sql
WITH brand_patterns AS (
  SELECT column1 AS pattern FROM VALUES
    ('%stiiizy%'),
    ('%stizzy%')
),
my_stores AS (
    SELECT DISTINCT store_id
    FROM menuitems m
    WHERE EXISTS (
        SELECT 1 FROM brand_patterns bp WHERE m.brand_name ILIKE bp.pattern
    )
)
SELECT
    s.state,
    s.velocity_band,
    COUNT(*) AS stores
FROM stores s
JOIN my_stores ms USING (store_id)
WHERE s.velocity_band IS NOT NULL
GROUP BY 1, 2
ORDER BY s.state, s.velocity_band;
```

---

## Stale listings: probably delisted

Items not observed in 14+ days — they may have been pulled from the menu.

```sql
WITH brand_patterns AS (
  SELECT column1 AS pattern FROM VALUES
    ('%stiiizy%'),
    ('%stizzy%')
)
SELECT
    s.name AS store_name,
    s.state,
    m.product_name,
    m.last_seen,
    DATEDIFF('day', m.last_seen, CURRENT_DATE) AS days_since_seen,
    m.quantity_in_stock,
    m.price
FROM menuitems m
JOIN stores s USING (store_id)
WHERE EXISTS (
    SELECT 1 FROM brand_patterns bp WHERE m.brand_name ILIKE bp.pattern
)
  AND m.last_seen < DATEADD('day', -14, CURRENT_DATE)   -- ⬅ swap threshold
ORDER BY days_since_seen DESC
LIMIT 100;
```

---

## Inventory trend for a single SKU at a store

Flatten `inventory_history` into a per-day timeseries. Use this to validate the `est_days_on_hand` model or to inspect anomalies.

```sql
SELECT
    m.product_name,
    s.name AS store_name,
    ih.value:DATE::date     AS history_date,
    ih.value:VALUE::number  AS stock_level
FROM menuitems m
JOIN stores s USING (store_id),
     LATERAL FLATTEN(input => m.inventory_history) ih
WHERE m.menu_item_id = '<paste-a-menu-item-id-here>'   -- ⬅ specific menu_item_id
ORDER BY history_date;
```

Tip: find a `menu_item_id` for your brand with:
```sql
SELECT menu_item_id, product_name FROM menuitems
WHERE brand_name ILIKE '%stiiizy%' LIMIT 5;
```

---

## Retailer view: category turnover nearby

Which categories move fastest at stores within X miles, for a retailer benchmarking against neighbors.

```sql
SELECT
    s.name AS competitor_store,
    s.city,
    ROUND(ST_DISTANCE(s.location, ST_POINT(-122.4194, 37.7749)) / 1609.34, 2) AS miles_away,
    m.category,
    SUM(m.avg_daily_units_sold) AS modeled_daily_units,
    SUM(m.avg_daily_sales)      AS modeled_daily_sales,
    COUNT(*)                    AS skus
FROM menuitems m
JOIN stores s USING (store_id)
WHERE ST_DWITHIN(s.location, ST_POINT(-122.4194, 37.7749), 16093.4)    -- 10 miles
  -- ⬅ swap longitude, latitude, radius
  AND m.avg_daily_units_sold IS NOT NULL
GROUP BY 1, 2, 3, 4
HAVING SUM(m.avg_daily_units_sold) > 0
ORDER BY miles_away, modeled_daily_sales DESC;
```
