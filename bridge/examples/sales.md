# Sales Examples

Queries for analyzing sell-through performance across your retail partners.

---

## Revenue by Store

See which stores are driving the most revenue for your products.

```sql
SELECT
    STORE_ID,
    STORE_NAME,
    COUNT(DISTINCT RECEIPT_ID) AS total_transactions,
    SUM(QUANTITY) AS total_units,
    SUM(REVENUE) AS total_revenue,
    SUM(REVENUE) / NULLIF(SUM(QUANTITY), 0) AS avg_revenue_per_unit
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY STORE_ID, STORE_NAME
ORDER BY total_revenue DESC;
```

## Top Products by Revenue

Find your best-selling products over the last 30 days.

```sql
SELECT
    PRODUCT_NAME,
    PRODUCT_ID,
    SUM(QUANTITY) AS total_units,
    SUM(REVENUE) AS total_revenue,
    SUM(DISCOUNT) AS total_discounts,
    COUNT(DISTINCT STORE_ID) AS stores_sold_in
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY PRODUCT_NAME, PRODUCT_ID
ORDER BY total_revenue DESC
LIMIT 25;
```

## Weekly Sales Trend

Track revenue and units week over week.

```sql
SELECT
    DATE_TRUNC('week', SOLD_DATE) AS week_start,
    SUM(QUANTITY) AS total_units,
    SUM(REVENUE) AS total_revenue,
    COUNT(DISTINCT RECEIPT_ID) AS total_transactions
FROM SALES
WHERE SOLD_DATE >= DATEADD('month', -3, CURRENT_DATE)
GROUP BY week_start
ORDER BY week_start;
```

## Daily Sales for a Specific Product

Drill into daily performance for a single product.

```sql
SELECT
    SOLD_DATE::DATE AS sale_date,
    STORE_NAME,
    SUM(QUANTITY) AS units,
    SUM(REVENUE) AS revenue,
    SUM(DISCOUNT) AS discounts
FROM SALES
WHERE PRODUCT_NAME ILIKE '%gummy%'
  AND SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY sale_date, STORE_NAME
ORDER BY sale_date DESC, revenue DESC;
```

## Revenue by Day of Week

Identify which days of the week sell the most.

```sql
SELECT
    DAYNAME(SOLD_DATE) AS day_of_week,
    DAYOFWEEK(SOLD_DATE) AS day_num,
    SUM(REVENUE) AS total_revenue,
    SUM(QUANTITY) AS total_units,
    COUNT(DISTINCT RECEIPT_ID) AS total_transactions
FROM SALES
WHERE SOLD_DATE >= DATEADD('month', -3, CURRENT_DATE)
GROUP BY day_of_week, day_num
ORDER BY day_num;
```

## Year-over-Year Monthly Comparison

Compare monthly revenue this year vs. last year.

```sql
SELECT
    MONTH(SOLD_DATE) AS month_num,
    MONTHNAME(SOLD_DATE) AS month_name,
    SUM(CASE WHEN YEAR(SOLD_DATE) = YEAR(CURRENT_DATE) THEN REVENUE END) AS revenue_this_year,
    SUM(CASE WHEN YEAR(SOLD_DATE) = YEAR(CURRENT_DATE) - 1 THEN REVENUE END) AS revenue_last_year,
    ROUND(
        (revenue_this_year - revenue_last_year) / NULLIF(revenue_last_year, 0) * 100,
        1
    ) AS yoy_change_pct
FROM SALES
WHERE SOLD_DATE >= DATEADD('year', -2, DATE_TRUNC('year', CURRENT_DATE))
GROUP BY month_num, month_name
ORDER BY month_num;
```

## Discount Impact

Understand how discounts affect your revenue.

```sql
SELECT
    PRODUCT_NAME,
    SUM(QUANTITY) AS total_units,
    SUM(BASE_PRICE) AS gross_sales,
    SUM(DISCOUNT) AS total_discounts,
    SUM(REVENUE) AS net_revenue,
    ROUND(SUM(DISCOUNT) / NULLIF(SUM(BASE_PRICE), 0) * 100, 1) AS discount_rate_pct
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
  AND DISCOUNT > 0
GROUP BY PRODUCT_NAME
ORDER BY total_discounts DESC
LIMIT 20;
```

## Customer Age Demographics

See the age distribution of customers buying your products.

```sql
SELECT
    AGE_AT_PURCHASE,
    COUNT(*) AS purchase_count,
    SUM(REVENUE) AS total_revenue,
    ROUND(AVG(REVENUE), 2) AS avg_revenue_per_purchase
FROM SALES
WHERE AGE_AT_PURCHASE IS NOT NULL
  AND SOLD_DATE >= DATEADD('month', -3, CURRENT_DATE)
GROUP BY AGE_AT_PURCHASE
ORDER BY AGE_AT_PURCHASE;
```

## Sales by Order Source

Break down sales by in-store vs. online or other channels.

```sql
SELECT
    COALESCE(ORDER_SOURCE, 'Unknown') AS order_source,
    COUNT(DISTINCT RECEIPT_ID) AS transactions,
    SUM(QUANTITY) AS total_units,
    SUM(REVENUE) AS total_revenue
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY order_source
ORDER BY total_revenue DESC;
```
