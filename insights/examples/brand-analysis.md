# Brand Analysis Examples

Queries for benchmarking brands, tracking market share, and evaluating product launches.

---

## Top Brands by Revenue

Find the top-selling brands in a market and category.

```sql
SELECT
    BRAND_NAME,
    SUM(TOTAL_SALES) AS total_sales,
    SUM(TOTAL_UNITS) AS total_units,
    ROUND(AVG(AVERAGE_PRICE_PER_ITEM), 2) AS avg_price,
    COUNT(DISTINCT INSIGHTS_PRODUCT_KEY) AS product_count
FROM MONTHLY_SALES
WHERE MARKET = 'CA'
  AND CATEGORY = 'Edibles'
  AND SOLD_MONTH >= DATEADD('month', -3, CURRENT_DATE)
GROUP BY BRAND_NAME
ORDER BY total_sales DESC
LIMIT 25;
```

## Brand Market Share Within a Category

See how brands rank by share of category sales.

```sql
SELECT
    BRAND_NAME,
    SUM(TOTAL_SALES) AS brand_sales,
    ROUND(
        SUM(TOTAL_SALES) / NULLIF(SUM(SUM(TOTAL_SALES)) OVER (), 0) * 100,
        1
    ) AS market_share_pct
FROM MONTHLY_SALES
WHERE MARKET = 'CA'
  AND CATEGORY = 'Flower'
  AND SOLD_MONTH >= DATEADD('month', -3, CURRENT_DATE)
GROUP BY BRAND_NAME
ORDER BY market_share_pct DESC
LIMIT 20;
```

## Brand Market Share Trend

Track how a brand's share changes month over month.

```sql
WITH category_totals AS (
    SELECT
        SOLD_MONTH,
        SUM(TOTAL_SALES) AS category_sales
    FROM MONTHLY_SALES
    WHERE MARKET = 'CA'
      AND CATEGORY = 'Edibles'
      AND SOLD_MONTH >= DATEADD('year', -1, CURRENT_DATE)
    GROUP BY SOLD_MONTH
)
SELECT
    ms.SOLD_MONTH,
    SUM(ms.TOTAL_SALES) AS brand_sales,
    ct.category_sales,
    ROUND(SUM(ms.TOTAL_SALES) / NULLIF(ct.category_sales, 0) * 100, 1) AS share_pct
FROM MONTHLY_SALES ms
JOIN category_totals ct ON ms.SOLD_MONTH = ct.SOLD_MONTH
WHERE ms.MARKET = 'CA'
  AND ms.CATEGORY = 'Edibles'
  AND ms.BRAND_NAME = 'Your Brand'
GROUP BY ms.SOLD_MONTH, ct.category_sales
ORDER BY ms.SOLD_MONTH;
```

## Brand Comparison

Compare two or more brands side by side.

```sql
SELECT
    BRAND_NAME,
    SUM(TOTAL_SALES) AS total_sales,
    SUM(TOTAL_UNITS) AS total_units,
    ROUND(AVG(AVERAGE_PRICE_PER_ITEM), 2) AS avg_price,
    ROUND(AVG(AVERAGE_COST_PER_ITEM), 2) AS avg_cost,
    SUM(TOTAL_DISCOUNT) AS total_discounts,
    COUNT(DISTINCT INSIGHTS_PRODUCT_KEY) AS sku_count
FROM MONTHLY_SALES
WHERE MARKET = 'CA'
  AND CATEGORY = 'Edibles'
  AND BRAND_NAME IN ('Brand A', 'Brand B', 'Brand C')
  AND SOLD_MONTH >= DATEADD('month', -3, CURRENT_DATE)
GROUP BY BRAND_NAME
ORDER BY total_sales DESC;
```

## New Product Launches

Find recently launched products and track their early performance.

```sql
WITH first_seen AS (
    SELECT
        INSIGHTS_PRODUCT_KEY,
        PRODUCT_NAME,
        BRAND_NAME,
        CATEGORY,
        SEGMENT,
        MIN(SOLD_ON) AS launch_date
    FROM DAILY_SALES
    WHERE MARKET = 'CA'
    GROUP BY INSIGHTS_PRODUCT_KEY, PRODUCT_NAME, BRAND_NAME, CATEGORY, SEGMENT
    HAVING MIN(SOLD_ON) >= DATEADD('day', -60, CURRENT_DATE)
)
SELECT
    fs.PRODUCT_NAME,
    fs.BRAND_NAME,
    fs.CATEGORY,
    fs.SEGMENT,
    fs.launch_date,
    SUM(ds.TOTAL_SALES) AS total_sales_since_launch,
    SUM(ds.TOTAL_UNITS) AS total_units_since_launch
FROM first_seen fs
JOIN DAILY_SALES ds
  ON fs.INSIGHTS_PRODUCT_KEY = ds.INSIGHTS_PRODUCT_KEY
  AND ds.MARKET = 'CA'
  AND ds.SOLD_ON >= fs.launch_date
GROUP BY fs.PRODUCT_NAME, fs.BRAND_NAME, fs.CATEGORY, fs.SEGMENT, fs.launch_date
ORDER BY total_sales_since_launch DESC
LIMIT 25;
```

## Brand Performance Across Markets

See how a brand performs in every market it's sold in.

```sql
SELECT
    MARKET,
    SUM(TOTAL_SALES) AS total_sales,
    SUM(TOTAL_UNITS) AS total_units,
    ROUND(AVG(AVERAGE_PRICE_PER_ITEM), 2) AS avg_price,
    COUNT(DISTINCT INSIGHTS_PRODUCT_KEY) AS product_count
FROM MONTHLY_SALES
WHERE BRAND_NAME = 'Your Brand'
  AND SOLD_MONTH >= DATEADD('month', -3, CURRENT_DATE)
GROUP BY MARKET
ORDER BY total_sales DESC;
```

## Top Products for a Brand

Drill into which specific products are driving a brand's sales.

```sql
SELECT
    PRODUCT_NAME,
    SEGMENT,
    PACKAGE_SIZE,
    SUM(TOTAL_SALES) AS total_sales,
    SUM(TOTAL_UNITS) AS total_units,
    ROUND(AVG(AVERAGE_PRICE_PER_ITEM), 2) AS avg_price
FROM DAILY_SALES
WHERE MARKET = 'CA'
  AND BRAND_NAME = 'Your Brand'
  AND SOLD_ON >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY PRODUCT_NAME, SEGMENT, PACKAGE_SIZE
ORDER BY total_sales DESC;
```
