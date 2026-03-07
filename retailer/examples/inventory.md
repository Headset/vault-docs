# Inventory Examples

Queries for monitoring stock levels, tracking batches, and managing inventory costs.

---

## Current Stock by Product

See what's on your shelves right now.

```sql
SELECT
    p.NAME AS product_name,
    p.BRAND,
    p.CATEGORY,
    p.QUANTITY_IN_STOCK,
    p.PRICE,
    p.QUANTITY_IN_STOCK * p.PRICE AS retail_value
FROM PRODUCTS p
WHERE p.QUANTITY_IN_STOCK > 0
ORDER BY retail_value DESC;
```

## Low Stock Alert

Find products running low that need reordering.

```sql
SELECT
    p.NAME AS product_name,
    p.BRAND,
    p.CATEGORY,
    p.QUANTITY_IN_STOCK,
    p.PRICE,
    p._LAST_UPDATE
FROM PRODUCTS p
WHERE p.QUANTITY_IN_STOCK > 0
  AND p.QUANTITY_IN_STOCK < 10
ORDER BY p.QUANTITY_IN_STOCK ASC;
```

## Stock Trend Over Time

Track how stock levels for a product change day by day.

```sql
SELECT
    STOCK_DATE,
    STORE_NAME,
    PRODUCT_ID,
    QUANTITY_IN_STOCK,
    AVG_COST_PER_UNIT
FROM INVENTORY_HISTORY
WHERE PRODUCT_ID = 'your-product-id'
  AND STOCK_DATE >= DATEADD('day', -60, CURRENT_DATE)
ORDER BY STOCK_DATE;
```

## Days of Supply Estimate

Estimate how many days of stock remain based on recent sales velocity.

```sql
WITH daily_sales AS (
    SELECT
        PRODUCT_ID,
        SUM(QUANTITY) / 30.0 AS avg_daily_units
    FROM SALES
    WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
    GROUP BY PRODUCT_ID
)
SELECT
    p.NAME AS product_name,
    p.BRAND,
    p.QUANTITY_IN_STOCK,
    ROUND(ds.avg_daily_units, 1) AS avg_daily_units,
    CASE
        WHEN ds.avg_daily_units > 0
        THEN ROUND(p.QUANTITY_IN_STOCK / ds.avg_daily_units, 1)
    END AS estimated_days_of_supply
FROM PRODUCTS p
LEFT JOIN daily_sales ds ON p.PRODUCT_ID = ds.PRODUCT_ID
WHERE p.QUANTITY_IN_STOCK > 0
ORDER BY estimated_days_of_supply ASC NULLS LAST;
```

## Total Inventory Value

Calculate total retail and cost value of your current inventory.

```sql
SELECT
    p.CATEGORY,
    COUNT(DISTINCT p.PRODUCT_ID) AS sku_count,
    SUM(p.QUANTITY_IN_STOCK) AS total_units,
    SUM(p.QUANTITY_IN_STOCK * p.PRICE) AS total_retail_value
FROM PRODUCTS p
WHERE p.QUANTITY_IN_STOCK > 0
GROUP BY p.CATEGORY
ORDER BY total_retail_value DESC;
```

## Batch Receive History

Track batches received over time.

```sql
SELECT
    DATE_TRUNC('week', RECEIVE_DATE_LOCAL) AS week_received,
    VENDOR_NAME,
    COUNT(DISTINCT BATCH_ID) AS batches,
    SUM(QUANTITY) AS total_units,
    ROUND(AVG(COST_PER_UNIT), 2) AS avg_cost_per_unit,
    SUM(QUANTITY * COST_PER_UNIT) AS total_cost
FROM INVENTORY_BATCHES
WHERE RECEIVE_DATE_LOCAL >= DATEADD('month', -3, CURRENT_DATE)
GROUP BY week_received, VENDOR_NAME
ORDER BY week_received DESC, total_cost DESC;
```

## Batch Sell-Through

See how much of each batch has sold and what's still on the shelf.

```sql
SELECT
    BATCH_ID,
    VENDOR_NAME,
    RECEIVE_DATE_LOCAL,
    QUANTITY AS original_quantity,
    QUANTITY_IN_STOCK AS remaining,
    QUANTITY - QUANTITY_IN_STOCK AS units_sold,
    ROUND(
        (QUANTITY - QUANTITY_IN_STOCK) / NULLIF(QUANTITY, 0) * 100,
        1
    ) AS sell_through_pct,
    COST_PER_UNIT
FROM INVENTORY_BATCHES
WHERE QUANTITY > 0
ORDER BY RECEIVE_DATE_LOCAL DESC;
```

## Vendor Cost Comparison

Compare costs across vendors for the same types of products.

```sql
SELECT
    ib.VENDOR_NAME,
    p.CATEGORY,
    COUNT(DISTINCT ib.BATCH_ID) AS batch_count,
    SUM(ib.QUANTITY) AS total_units,
    ROUND(AVG(ib.COST_PER_UNIT), 2) AS avg_cost,
    MIN(ib.COST_PER_UNIT) AS min_cost,
    MAX(ib.COST_PER_UNIT) AS max_cost
FROM INVENTORY_BATCHES ib
JOIN PRODUCTS p ON ib.PRODUCT_ID = p.PRODUCT_ID AND ib.STORE_ID = p.STORE_ID
WHERE ib.RECEIVE_DATE_LOCAL >= DATEADD('month', -6, CURRENT_DATE)
GROUP BY ib.VENDOR_NAME, p.CATEGORY
ORDER BY p.CATEGORY, avg_cost;
```

## Dead Stock

Find products with stock that haven't sold in over 90 days.

```sql
WITH last_sale AS (
    SELECT
        PRODUCT_ID,
        STORE_ID,
        MAX(SOLD_DATE)::DATE AS last_sold
    FROM SALES
    GROUP BY PRODUCT_ID, STORE_ID
)
SELECT
    p.NAME AS product_name,
    p.BRAND,
    p.CATEGORY,
    p.QUANTITY_IN_STOCK,
    p.PRICE,
    p.QUANTITY_IN_STOCK * p.PRICE AS stuck_retail_value,
    ls.last_sold,
    DATEDIFF('day', ls.last_sold, CURRENT_DATE) AS days_since_last_sale
FROM PRODUCTS p
LEFT JOIN last_sale ls ON p.PRODUCT_ID = ls.PRODUCT_ID AND p.STORE_ID = ls.STORE_ID
WHERE p.QUANTITY_IN_STOCK > 0
  AND (ls.last_sold < DATEADD('day', -90, CURRENT_DATE) OR ls.last_sold IS NULL)
ORDER BY stuck_retail_value DESC;
```
