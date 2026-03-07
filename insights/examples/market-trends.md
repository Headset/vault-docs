# Market Trends Examples

Queries for understanding category performance, market sizing, and how the industry is shifting over time.

---

## Category Sales by Market

See total sales and units for each category in a given market.

```sql
SELECT
    CATEGORY,
    SUM(TOTAL_SALES) AS total_sales,
    SUM(TOTAL_UNITS) AS total_units,
    ROUND(SUM(TOTAL_SALES) / NULLIF(SUM(TOTAL_UNITS), 0), 2) AS avg_price_per_unit
FROM CATEGORY
WHERE MARKET = 'CA'
  AND SOLD_ON >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY CATEGORY
ORDER BY total_sales DESC;
```

## Category Trend Over Time

Track how a category is growing or declining month over month.

```sql
SELECT
    DATE_TRUNC('month', SOLD_ON) AS month,
    SUM(TOTAL_SALES) AS total_sales,
    SUM(TOTAL_UNITS) AS total_units
FROM CATEGORY
WHERE MARKET = 'CO'
  AND CATEGORY = 'Edibles'
  AND SOLD_ON >= DATEADD('year', -1, CURRENT_DATE)
GROUP BY month
ORDER BY month;
```

## Market Comparison — Category Sales Across States

Compare a category's performance across multiple markets.

```sql
SELECT
    MARKET,
    SUM(TOTAL_SALES) AS total_sales,
    SUM(TOTAL_UNITS) AS total_units,
    ROUND(SUM(TOTAL_SALES) / NULLIF(SUM(TOTAL_UNITS), 0), 2) AS avg_price_per_unit
FROM CATEGORY
WHERE CATEGORY = 'Flower'
  AND SOLD_ON >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY MARKET
ORDER BY total_sales DESC;
```

## Category Market Share

See each category's share of total sales in a market.

```sql
SELECT
    CATEGORY,
    SUM(TOTAL_SALES) AS category_sales,
    ROUND(
        SUM(TOTAL_SALES) / NULLIF(SUM(SUM(TOTAL_SALES)) OVER (), 0) * 100,
        1
    ) AS market_share_pct
FROM CATEGORY
WHERE MARKET = 'CA'
  AND SOLD_ON >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY CATEGORY
ORDER BY market_share_pct DESC;
```

## Year-over-Year Category Growth

Compare category performance this year vs. last year.

```sql
SELECT
    CATEGORY,
    SUM(CASE WHEN YEAR(SOLD_ON) = YEAR(CURRENT_DATE) THEN TOTAL_SALES END) AS sales_this_year,
    SUM(CASE WHEN YEAR(SOLD_ON) = YEAR(CURRENT_DATE) - 1 THEN TOTAL_SALES END) AS sales_last_year,
    ROUND(
        (sales_this_year - sales_last_year) / NULLIF(sales_last_year, 0) * 100,
        1
    ) AS yoy_change_pct
FROM CATEGORY
WHERE MARKET = 'CA'
  AND MONTH(SOLD_ON) <= MONTH(CURRENT_DATE)
GROUP BY CATEGORY
ORDER BY yoy_change_pct DESC;
```

## Segment Breakdown Within a Category

See which segments are driving a category's performance.

```sql
SELECT
    SEGMENT,
    SUM(TOTAL_SALES) AS total_sales,
    SUM(TOTAL_UNITS) AS total_units,
    ROUND(AVG(AVERAGE_PRICE_PER_ITEM), 2) AS avg_price
FROM DAILY_SALES
WHERE MARKET = 'CA'
  AND CATEGORY = 'Concentrates'
  AND SOLD_ON >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY SEGMENT
ORDER BY total_sales DESC;
```

## Fastest Growing Segments

Find the segments with the largest month-over-month sales increase.

```sql
WITH monthly AS (
    SELECT
        CATEGORY,
        SEGMENT,
        DATE_TRUNC('month', SOLD_ON) AS month,
        SUM(TOTAL_SALES) AS total_sales
    FROM DAILY_SALES
    WHERE MARKET = 'CA'
      AND SOLD_ON >= DATEADD('month', -3, CURRENT_DATE)
    GROUP BY CATEGORY, SEGMENT, month
),
growth AS (
    SELECT
        CATEGORY,
        SEGMENT,
        month,
        total_sales,
        LAG(total_sales) OVER (PARTITION BY CATEGORY, SEGMENT ORDER BY month) AS prev_month_sales
    FROM monthly
)
SELECT
    CATEGORY,
    SEGMENT,
    month,
    total_sales,
    prev_month_sales,
    ROUND(
        (total_sales - prev_month_sales) / NULLIF(prev_month_sales, 0) * 100,
        1
    ) AS mom_growth_pct
FROM growth
WHERE prev_month_sales IS NOT NULL
ORDER BY mom_growth_pct DESC
LIMIT 20;
```
