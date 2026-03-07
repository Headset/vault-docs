# Products and Stores Examples

Queries for understanding your distribution footprint, product catalog, and store-level performance.

---

## Distribution Coverage

See which stores carry your products and how many SKUs each store has.

```sql
SELECT
    s.STORE_NAME,
    s.CITY,
    s.STATE,
    COUNT(DISTINCT p.PRODUCT_ID) AS sku_count,
    SUM(p.QUANTITY_IN_STOCK) AS total_stock
FROM STORES s
JOIN PRODUCTS p ON s.STORE_ID = p.STORE_ID
WHERE p.QUANTITY_IN_STOCK > 0
GROUP BY s.STORE_NAME, s.CITY, s.STATE
ORDER BY sku_count DESC;
```

## Products Not Carried by a Store

Find products that exist at other stores but are missing from a specific store — useful for identifying distribution gaps.

```sql
WITH all_products AS (
    SELECT DISTINCT PRODUCT_NAME, BRAND, CATEGORY
    FROM PRODUCTS
),
store_products AS (
    SELECT DISTINCT PRODUCT_NAME
    FROM PRODUCTS
    WHERE STORE_NAME = 'Your Store Name'
)
SELECT
    ap.PRODUCT_NAME,
    ap.BRAND,
    ap.CATEGORY
FROM all_products ap
LEFT JOIN store_products sp ON ap.PRODUCT_NAME = sp.PRODUCT_NAME
WHERE sp.PRODUCT_NAME IS NULL
ORDER BY ap.BRAND, ap.PRODUCT_NAME;
```

## New Product Launches

Find products that started selling recently, based on their first sale date.

```sql
SELECT
    PRODUCT_NAME,
    BRAND,
    CATEGORY,
    STORE_NAME,
    FIRST_SALE,
    QUANTITY_IN_STOCK
FROM PRODUCTS
WHERE FIRST_SALE >= DATEADD('day', -30, CURRENT_DATE)
ORDER BY FIRST_SALE DESC;
```

## Stale or Discontinued Products

Find products that haven't sold in a while but may still be on shelves.

```sql
SELECT
    STORE_NAME,
    PRODUCT_NAME,
    BRAND,
    CATEGORY,
    LAST_SALE,
    QUANTITY_IN_STOCK,
    DATEDIFF('day', LAST_SALE, CURRENT_DATE) AS days_since_last_sale
FROM PRODUCTS
WHERE LAST_SALE < DATEADD('day', -90, CURRENT_DATE)
  AND QUANTITY_IN_STOCK > 0
ORDER BY days_since_last_sale DESC;
```

## Product Catalog by Category and Brand

Get a breakdown of your catalog by category.

```sql
SELECT
    CATEGORY,
    BRAND,
    COUNT(DISTINCT PRODUCT_ID) AS sku_count,
    SUM(QUANTITY_IN_STOCK) AS total_stock,
    COUNT(DISTINCT STORE_ID) AS stores_carrying
FROM PRODUCTS
GROUP BY CATEGORY, BRAND
ORDER BY CATEGORY, sku_count DESC;
```

## Store Directory

List all stores with their locations and current stock summary.

```sql
SELECT
    STORE_ID,
    STORE_NAME,
    ADDRESS,
    CITY,
    STATE,
    POSTAL_CODE,
    QUANTITY_IN_STOCK AS total_store_stock,
    LAST_STORE_STOCK_UPDATE
FROM STORES
ORDER BY STATE, CITY, STORE_NAME;
```

## Store Performance Comparison

Compare revenue, units, and transaction count across stores over the last 30 days.

```sql
SELECT
    sa.STORE_NAME,
    st.CITY,
    st.STATE,
    COUNT(DISTINCT sa.RECEIPT_ID) AS transactions,
    SUM(sa.QUANTITY) AS total_units,
    SUM(sa.REVENUE) AS total_revenue,
    ROUND(SUM(sa.REVENUE) / NULLIF(COUNT(DISTINCT sa.RECEIPT_ID), 0), 2) AS avg_revenue_per_transaction
FROM SALES sa
JOIN STORES st ON sa.STORE_ID = st.STORE_ID
WHERE sa.SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY sa.STORE_NAME, st.CITY, st.STATE
ORDER BY total_revenue DESC;
```

## Products by Potency

Find products filtered by THC or CBD content.

```sql
SELECT
    PRODUCT_NAME,
    BRAND,
    CATEGORY,
    STRAIN,
    THC_RESULT,
    CBD_RESULT,
    QUANTITY_IN_STOCK,
    COUNT(DISTINCT STORE_ID) AS stores_carrying
FROM PRODUCTS
WHERE THC_RESULT IS NOT NULL
GROUP BY PRODUCT_NAME, BRAND, CATEGORY, STRAIN, THC_RESULT, CBD_RESULT, QUANTITY_IN_STOCK
ORDER BY PRODUCT_NAME;
```
