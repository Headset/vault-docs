# Products and Stores Examples

Queries for exploring your product catalog, comparing store performance, and analyzing employee sales.

---

## Product Catalog Overview

Get a high-level view of your catalog by category.

```sql
SELECT
    CATEGORY,
    COUNT(DISTINCT PRODUCT_ID) AS sku_count,
    COUNT(DISTINCT BRAND) AS brand_count,
    SUM(QUANTITY_IN_STOCK) AS total_stock,
    ROUND(AVG(PRICE), 2) AS avg_price
FROM PRODUCTS
GROUP BY CATEGORY
ORDER BY sku_count DESC;
```

## Products by Vendor

See what each vendor supplies.

```sql
SELECT
    VENDOR,
    BRAND,
    COUNT(DISTINCT PRODUCT_ID) AS sku_count,
    SUM(QUANTITY_IN_STOCK) AS total_stock,
    ROUND(AVG(PRICE), 2) AS avg_price
FROM PRODUCTS
WHERE VENDOR IS NOT NULL
GROUP BY VENDOR, BRAND
ORDER BY VENDOR, sku_count DESC;
```

## Products by Potency

Filter your catalog by THC or CBD content.

```sql
SELECT
    NAME AS product_name,
    BRAND,
    CATEGORY,
    STRAIN,
    THC_RESULT,
    CBD_RESULT,
    PRICE,
    QUANTITY_IN_STOCK
FROM PRODUCTS
WHERE THC_RESULT IS NOT NULL
ORDER BY NAME;
```

## Store Performance Comparison

Compare key metrics across your locations.

```sql
SELECT
    STORE_NAME,
    CITY,
    STATE,
    COUNT(DISTINCT RECEIPT_ID) AS transactions,
    COUNT(DISTINCT CUSTOMER_KEY) AS unique_customers,
    SUM(REVENUE) AS total_revenue,
    SUM(PROFIT) AS total_profit,
    ROUND(SUM(REVENUE) / NULLIF(COUNT(DISTINCT RECEIPT_ID), 0), 2) AS avg_transaction_value
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY STORE_NAME, CITY, STATE
ORDER BY total_revenue DESC;
```

## Store Directory

List all stores with connection status and metadata.

```sql
SELECT
    STORE_ID,
    NAME,
    COMPANY_NAME,
    ADDRESS,
    CITY,
    STATE,
    COUNTRY,
    POSTAL_CODE,
    TIME_ZONE,
    CURRENT_CONNECTION,
    SQUARE_FEET
FROM STORES
ORDER BY STATE, CITY, NAME;
```

## Revenue per Square Foot

Compare store efficiency using square footage.

```sql
SELECT
    st.NAME AS store_name,
    st.CITY,
    st.STATE,
    st.SQUARE_FEET,
    SUM(sa.REVENUE) AS monthly_revenue,
    ROUND(SUM(sa.REVENUE) / NULLIF(st.SQUARE_FEET, 0), 2) AS revenue_per_sqft
FROM STORES st
JOIN SALES sa ON st.STORE_ID = sa.STORE_ID
WHERE sa.SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
  AND st.SQUARE_FEET > 0
GROUP BY st.NAME, st.CITY, st.STATE, st.SQUARE_FEET
ORDER BY revenue_per_sqft DESC;
```

## Employee Performance

See which employees are driving the most sales.

```sql
SELECT
    EMPLOYEE_NAME,
    STORE_NAME,
    COUNT(DISTINCT RECEIPT_ID) AS transactions,
    SUM(QUANTITY) AS total_units,
    SUM(REVENUE) AS total_revenue,
    ROUND(SUM(REVENUE) / NULLIF(COUNT(DISTINCT RECEIPT_ID), 0), 2) AS avg_transaction_value
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
  AND EMPLOYEE_NAME IS NOT NULL
GROUP BY EMPLOYEE_NAME, STORE_NAME
ORDER BY total_revenue DESC;
```

## Employee Performance by Hour

Identify peak hours and employee coverage.

```sql
SELECT
    EMPLOYEE_NAME,
    HOUR(SOLD_DATE) AS hour_of_day,
    COUNT(DISTINCT RECEIPT_ID) AS transactions,
    SUM(REVENUE) AS revenue
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -7, CURRENT_DATE)
  AND EMPLOYEE_NAME IS NOT NULL
GROUP BY EMPLOYEE_NAME, hour_of_day
ORDER BY EMPLOYEE_NAME, hour_of_day;
```

## Category Mix by Store

See how each store's sales break down by category.

```sql
SELECT
    STORE_NAME,
    CATEGORY,
    SUM(REVENUE) AS category_revenue,
    ROUND(
        SUM(REVENUE) / NULLIF(SUM(SUM(REVENUE)) OVER (PARTITION BY STORE_NAME), 0) * 100,
        1
    ) AS pct_of_store_revenue
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY STORE_NAME, CATEGORY
ORDER BY STORE_NAME, category_revenue DESC;
```

## Wholesale vs. Retail Sales

Compare wholesale and retail transaction volumes.

```sql
SELECT
    CASE WHEN IS_WHOLESALE THEN 'Wholesale' ELSE 'Retail' END AS sale_type,
    COUNT(DISTINCT RECEIPT_ID) AS transactions,
    SUM(QUANTITY) AS total_units,
    SUM(REVENUE) AS total_revenue,
    SUM(PROFIT) AS total_profit
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY sale_type
ORDER BY total_revenue DESC;
```
