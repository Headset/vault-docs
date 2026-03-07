# Sales Examples

Queries for analyzing transaction data, revenue trends, basket composition, and channel mix.

---

## Revenue by Store

Compare performance across your locations.

```sql
SELECT
    STORE_NAME,
    CITY,
    STATE,
    COUNT(DISTINCT RECEIPT_ID) AS transactions,
    SUM(QUANTITY) AS total_units,
    SUM(REVENUE) AS total_revenue,
    SUM(PROFIT) AS total_profit
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY STORE_NAME, CITY, STATE
ORDER BY total_revenue DESC;
```

## Top Products by Revenue

Find your best sellers over the last 30 days.

```sql
SELECT
    PRODUCT_NAME,
    BRAND,
    CATEGORY,
    SUM(QUANTITY) AS total_units,
    SUM(REVENUE) AS total_revenue,
    SUM(PROFIT) AS total_profit,
    COUNT(DISTINCT STORE_ID) AS stores_sold_in
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY PRODUCT_NAME, BRAND, CATEGORY
ORDER BY total_revenue DESC
LIMIT 25;
```

## Weekly Sales Trend

Track revenue and transactions week over week.

```sql
SELECT
    DATE_TRUNC('week', SOLD_DATE) AS week_start,
    COUNT(DISTINCT RECEIPT_ID) AS transactions,
    SUM(QUANTITY) AS total_units,
    SUM(REVENUE) AS total_revenue,
    SUM(PROFIT) AS total_profit
FROM SALES
WHERE SOLD_DATE >= DATEADD('month', -3, CURRENT_DATE)
GROUP BY week_start
ORDER BY week_start;
```

## Basket Analysis — Items per Transaction

Understand how many items customers are buying per visit.

```sql
SELECT
    RECEIPT_ID,
    STORE_NAME,
    SOLD_DATE::DATE AS sale_date,
    COUNT(*) AS items_in_basket,
    SUM(REVENUE) AS basket_total,
    COUNT(DISTINCT CATEGORY) AS categories_in_basket
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY RECEIPT_ID, STORE_NAME, sale_date
ORDER BY items_in_basket DESC
LIMIT 100;
```

## Average Basket Size by Store

Compare average basket size across locations.

```sql
WITH baskets AS (
    SELECT
        STORE_NAME,
        RECEIPT_ID,
        COUNT(*) AS items,
        SUM(REVENUE) AS basket_total
    FROM SALES
    WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
    GROUP BY STORE_NAME, RECEIPT_ID
)
SELECT
    STORE_NAME,
    COUNT(*) AS total_baskets,
    ROUND(AVG(items), 1) AS avg_items_per_basket,
    ROUND(AVG(basket_total), 2) AS avg_basket_value
FROM baskets
GROUP BY STORE_NAME
ORDER BY avg_basket_value DESC;
```

## Sales by Category and Brand

See which categories and brands drive the most volume.

```sql
SELECT
    CATEGORY,
    BRAND,
    SUM(QUANTITY) AS total_units,
    SUM(REVENUE) AS total_revenue,
    SUM(PROFIT) AS total_profit,
    ROUND(SUM(DISCOUNT) / NULLIF(SUM(BASE_PRICE), 0) * 100, 1) AS discount_rate_pct
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY CATEGORY, BRAND
ORDER BY total_revenue DESC;
```

## Medical vs. Recreational Sales

Break down sales by medical and recreational.

```sql
SELECT
    CASE WHEN IS_MEDICAL THEN 'Medical' ELSE 'Recreational' END AS sale_type,
    COUNT(DISTINCT RECEIPT_ID) AS transactions,
    SUM(QUANTITY) AS total_units,
    SUM(REVENUE) AS total_revenue,
    ROUND(AVG(REVENUE), 2) AS avg_line_item_revenue
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY sale_type
ORDER BY total_revenue DESC;
```

## Delivery vs. In-Store

Compare delivery and in-store performance.

```sql
SELECT
    CASE WHEN IS_DELIVERY THEN 'Delivery' ELSE 'In-Store' END AS channel,
    COALESCE(ORDER_SOURCE, 'Unknown') AS order_source,
    COUNT(DISTINCT RECEIPT_ID) AS transactions,
    SUM(QUANTITY) AS total_units,
    SUM(REVENUE) AS total_revenue
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY channel, order_source
ORDER BY total_revenue DESC;
```

## Payment Type Breakdown

See how customers are paying.

```sql
SELECT
    PAYMENT_TYPE,
    COUNT(DISTINCT RECEIPT_ID) AS transactions,
    SUM(REVENUE) AS total_revenue,
    ROUND(SUM(REVENUE) / NULLIF(COUNT(DISTINCT RECEIPT_ID), 0), 2) AS avg_transaction_value
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY PAYMENT_TYPE
ORDER BY total_revenue DESC;
```

## Discount Analysis

Understand which products are being discounted the most.

```sql
SELECT
    PRODUCT_NAME,
    BRAND,
    SUM(QUANTITY) AS total_units,
    SUM(BASE_PRICE) AS gross_sales,
    SUM(DISCOUNT) AS total_discounts,
    SUM(REVENUE) AS net_revenue,
    ROUND(SUM(DISCOUNT) / NULLIF(SUM(BASE_PRICE), 0) * 100, 1) AS discount_rate_pct
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
  AND DISCOUNT > 0
GROUP BY PRODUCT_NAME, BRAND
ORDER BY total_discounts DESC
LIMIT 20;
```

## Discounts Used

See which named discounts are applied most often.

```sql
SELECT
    d.VALUE::TEXT AS discount_name,
    COUNT(*) AS times_used,
    SUM(s.DISCOUNT) AS total_discount_value
FROM SALES s,
    LATERAL FLATTEN(input => s.DISCOUNTS_USED) d
WHERE s.SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY discount_name
ORDER BY times_used DESC;
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
