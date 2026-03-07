# Inventory Examples

Queries for monitoring stock levels, tracking movement, and identifying inventory risks across your retail partners.

---

## Current Stock by Store

See how much of each product is currently in stock at each store.

```sql
SELECT
    s.STORE_NAME,
    p.PRODUCT_NAME,
    p.BRAND,
    p.CATEGORY,
    p.QUANTITY_IN_STOCK
FROM PRODUCTS p
JOIN STORES s ON p.STORE_ID = s.STORE_ID
WHERE p.QUANTITY_IN_STOCK > 0
ORDER BY s.STORE_NAME, p.QUANTITY_IN_STOCK DESC;
```

## Low Stock Alert

Find products that are running low (fewer than 10 units remaining).

```sql
SELECT
    s.STORE_NAME,
    p.PRODUCT_NAME,
    p.BRAND,
    p.QUANTITY_IN_STOCK,
    p.LAST_STOCK_UPDATE
FROM PRODUCTS p
JOIN STORES s ON p.STORE_ID = s.STORE_ID
WHERE p.QUANTITY_IN_STOCK > 0
  AND p.QUANTITY_IN_STOCK < 10
ORDER BY p.QUANTITY_IN_STOCK ASC;
```

## Out-of-Stock Products

Identify products that have sold before but currently have zero stock.

```sql
SELECT
    s.STORE_NAME,
    p.PRODUCT_NAME,
    p.BRAND,
    p.LAST_SALE,
    p.LAST_STOCK_UPDATE
FROM PRODUCTS p
JOIN STORES s ON p.STORE_ID = s.STORE_ID
WHERE p.QUANTITY_IN_STOCK = 0
  AND p.LAST_SALE IS NOT NULL
ORDER BY p.LAST_SALE DESC;
```

## Stock Trend Over Time

Track how stock levels for a product change day by day.

```sql
SELECT
    STOCK_DATE,
    STORE_NAME,
    QUANTITY_IN_STOCK,
    AVG_PRICE_PER_UNIT,
    AVG_COST_PER_UNIT
FROM INVENTORY_HISTORY
WHERE PRODUCT_ID = 'your-product-id'
  AND STOCK_DATE >= DATEADD('day', -60, CURRENT_DATE)
ORDER BY STOCK_DATE;
```

## Days of Supply Estimate

Estimate how many days of stock remain for each product at each store, based on average daily sales over the last 30 days.

```sql
WITH daily_sales AS (
    SELECT
        STORE_ID,
        PRODUCT_ID,
        SUM(QUANTITY) / 30.0 AS avg_daily_units
    FROM SALES
    WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
    GROUP BY STORE_ID, PRODUCT_ID
)
SELECT
    s.STORE_NAME,
    p.PRODUCT_NAME,
    p.QUANTITY_IN_STOCK,
    ds.avg_daily_units,
    CASE
        WHEN ds.avg_daily_units > 0
        THEN ROUND(p.QUANTITY_IN_STOCK / ds.avg_daily_units, 1)
    END AS estimated_days_of_supply
FROM PRODUCTS p
JOIN STORES s ON p.STORE_ID = s.STORE_ID
LEFT JOIN daily_sales ds ON p.STORE_ID = ds.STORE_ID AND p.PRODUCT_ID = ds.PRODUCT_ID
WHERE p.QUANTITY_IN_STOCK > 0
ORDER BY estimated_days_of_supply ASC NULLS LAST;
```

## Sell-Through Rate

Calculate the sell-through rate for each product over the last 30 days: units sold as a percentage of (units sold + units still in stock).

```sql
WITH recent_sales AS (
    SELECT
        STORE_ID,
        PRODUCT_ID,
        SUM(QUANTITY) AS units_sold
    FROM SALES
    WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
    GROUP BY STORE_ID, PRODUCT_ID
)
SELECT
    s.STORE_NAME,
    p.PRODUCT_NAME,
    p.BRAND,
    rs.units_sold,
    p.QUANTITY_IN_STOCK,
    ROUND(
        rs.units_sold / NULLIF(rs.units_sold + p.QUANTITY_IN_STOCK, 0) * 100,
        1
    ) AS sell_through_pct
FROM PRODUCTS p
JOIN STORES s ON p.STORE_ID = s.STORE_ID
JOIN recent_sales rs ON p.STORE_ID = rs.STORE_ID AND p.PRODUCT_ID = rs.PRODUCT_ID
ORDER BY sell_through_pct DESC;
```

## Inventory Value by Store

Calculate the total inventory value at each store using average cost per unit.

```sql
SELECT
    ih.STORE_NAME,
    COUNT(DISTINCT ih.PRODUCT_ID) AS product_count,
    SUM(ih.QUANTITY_IN_STOCK) AS total_units,
    SUM(ih.QUANTITY_IN_STOCK * ih.AVG_COST_PER_UNIT) AS total_inventory_value
FROM INVENTORY_HISTORY ih
WHERE ih.STOCK_DATE = (SELECT MAX(STOCK_DATE) FROM INVENTORY_HISTORY)
  AND ih.QUANTITY_IN_STOCK > 0
GROUP BY ih.STORE_NAME
ORDER BY total_inventory_value DESC;
```

## Stock Received Over Time

Track how often and how much stock stores are receiving from you.

```sql
SELECT
    DATE_TRUNC('week', RECEIVE_DATE_LOCAL) AS week_received,
    COUNT(DISTINCT BATCH_ID) AS batches_received,
    SUM(QUANTITY) AS total_units_received,
    SUM(QUANTITY * COST_PER_UNIT) AS total_cost
FROM INVENTORY_BATCHES
WHERE RECEIVE_DATE_LOCAL >= DATEADD('month', -3, CURRENT_DATE)
GROUP BY week_received
ORDER BY week_received;
```
