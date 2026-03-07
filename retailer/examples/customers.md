# Customer Examples

Queries for understanding your customer base, identifying top spenders, measuring retention, and segmenting by demographics.

---

## Top Customers by Spend

Find your highest-value customers over the last 90 days.

```sql
SELECT
    s.CUSTOMER_KEY,
    s.CUSTOMER_FIRST_NAME,
    s.CUSTOMER_AGE_GROUP,
    COUNT(DISTINCT s.RECEIPT_ID) AS total_visits,
    SUM(s.QUANTITY) AS total_units,
    SUM(s.REVENUE) AS total_spend
FROM SALES s
WHERE s.SOLD_DATE >= DATEADD('day', -90, CURRENT_DATE)
  AND s.CUSTOMER_KEY IS NOT NULL
GROUP BY s.CUSTOMER_KEY, s.CUSTOMER_FIRST_NAME, s.CUSTOMER_AGE_GROUP
ORDER BY total_spend DESC
LIMIT 50;
```

## Customer Visit Frequency

Segment customers by how often they visit.

```sql
WITH customer_visits AS (
    SELECT
        CUSTOMER_KEY,
        COUNT(DISTINCT RECEIPT_ID) AS visits,
        SUM(REVENUE) AS total_spend
    FROM SALES
    WHERE SOLD_DATE >= DATEADD('day', -90, CURRENT_DATE)
      AND CUSTOMER_KEY IS NOT NULL
    GROUP BY CUSTOMER_KEY
)
SELECT
    CASE
        WHEN visits = 1 THEN '1 visit'
        WHEN visits BETWEEN 2 AND 3 THEN '2-3 visits'
        WHEN visits BETWEEN 4 AND 8 THEN '4-8 visits'
        ELSE '9+ visits'
    END AS frequency_bucket,
    COUNT(*) AS customer_count,
    ROUND(AVG(total_spend), 2) AS avg_spend,
    SUM(total_spend) AS total_spend
FROM customer_visits
GROUP BY frequency_bucket
ORDER BY MIN(visits);
```

## Customer Retention — Repeat vs. New

Compare new customers to returning customers each month.

```sql
WITH first_purchase AS (
    SELECT
        CUSTOMER_KEY,
        MIN(SOLD_DATE::DATE) AS first_purchase_date
    FROM SALES
    WHERE CUSTOMER_KEY IS NOT NULL
    GROUP BY CUSTOMER_KEY
)
SELECT
    DATE_TRUNC('month', s.SOLD_DATE) AS month,
    COUNT(DISTINCT CASE
        WHEN fp.first_purchase_date >= DATE_TRUNC('month', s.SOLD_DATE) THEN s.CUSTOMER_KEY
    END) AS new_customers,
    COUNT(DISTINCT CASE
        WHEN fp.first_purchase_date < DATE_TRUNC('month', s.SOLD_DATE) THEN s.CUSTOMER_KEY
    END) AS returning_customers
FROM SALES s
JOIN first_purchase fp ON s.CUSTOMER_KEY = fp.CUSTOMER_KEY
WHERE s.SOLD_DATE >= DATEADD('month', -6, CURRENT_DATE)
GROUP BY month
ORDER BY month;
```

## Customer Demographics — Age Group Breakdown

See who your customers are by age group.

```sql
SELECT
    CUSTOMER_AGE_GROUP,
    COUNT(DISTINCT CUSTOMER_KEY) AS customers,
    COUNT(DISTINCT RECEIPT_ID) AS transactions,
    SUM(REVENUE) AS total_revenue,
    ROUND(SUM(REVENUE) / NULLIF(COUNT(DISTINCT CUSTOMER_KEY), 0), 2) AS revenue_per_customer
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -90, CURRENT_DATE)
  AND CUSTOMER_AGE_GROUP IS NOT NULL
GROUP BY CUSTOMER_AGE_GROUP
ORDER BY CUSTOMER_AGE_GROUP;
```

## Customer Demographics — Gender

Break down spending by gender.

```sql
SELECT
    CUSTOMER_GENDER,
    COUNT(DISTINCT CUSTOMER_KEY) AS customers,
    SUM(REVENUE) AS total_revenue,
    ROUND(SUM(REVENUE) / NULLIF(COUNT(DISTINCT CUSTOMER_KEY), 0), 2) AS revenue_per_customer,
    ROUND(AVG(QUANTITY), 1) AS avg_units_per_line_item
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -90, CURRENT_DATE)
  AND CUSTOMER_GENDER IS NOT NULL
GROUP BY CUSTOMER_GENDER
ORDER BY total_revenue DESC;
```

## Referral Source Analysis

See which referral sources bring in the most valuable customers.

```sql
SELECT
    c.REFERRAL_SOURCE,
    COUNT(DISTINCT c.CUSTOMER_KEY) AS customer_count,
    AVG(c.NUM_TOTAL_TRANSACTIONS) AS avg_lifetime_transactions,
    SUM(s.REVENUE) AS total_revenue_90d
FROM CUSTOMERS c
JOIN SALES s ON c.CUSTOMER_KEY = s.CUSTOMER_KEY AND c.STORE_ID = s.STORE_ID
WHERE c.REFERRAL_SOURCE IS NOT NULL
  AND s.SOLD_DATE >= DATEADD('day', -90, CURRENT_DATE)
GROUP BY c.REFERRAL_SOURCE
ORDER BY total_revenue_90d DESC;
```

## Customer Category Preferences

See what categories each customer buys to understand preferences.

```sql
SELECT
    CUSTOMER_KEY,
    CUSTOMER_FIRST_NAME,
    CATEGORY,
    COUNT(DISTINCT RECEIPT_ID) AS purchases,
    SUM(REVENUE) AS category_spend
FROM SALES
WHERE SOLD_DATE >= DATEADD('day', -90, CURRENT_DATE)
  AND CUSTOMER_KEY IS NOT NULL
GROUP BY CUSTOMER_KEY, CUSTOMER_FIRST_NAME, CATEGORY
ORDER BY CUSTOMER_KEY, category_spend DESC;
```

## Customers by Postal Code

Understand where your customers are coming from.

```sql
SELECT
    c.POSTAL_CODE,
    COUNT(DISTINCT c.CUSTOMER_KEY) AS customer_count,
    AVG(c.NUM_TOTAL_TRANSACTIONS) AS avg_lifetime_transactions
FROM CUSTOMERS c
WHERE c.POSTAL_CODE IS NOT NULL
GROUP BY c.POSTAL_CODE
ORDER BY customer_count DESC
LIMIT 25;
```

## Lapsed Customers

Find customers who used to purchase regularly but haven't been back recently.

```sql
SELECT
    c.CUSTOMER_KEY,
    c.FIRST_NAME,
    c.EMAIL,
    c.NUM_TOTAL_TRANSACTIONS,
    MAX(s.SOLD_DATE)::DATE AS last_purchase,
    DATEDIFF('day', MAX(s.SOLD_DATE), CURRENT_DATE) AS days_since_last_purchase
FROM CUSTOMERS c
JOIN SALES s ON c.CUSTOMER_KEY = s.CUSTOMER_KEY AND c.STORE_ID = s.STORE_ID
WHERE c.NUM_TOTAL_TRANSACTIONS >= 5
GROUP BY c.CUSTOMER_KEY, c.FIRST_NAME, c.EMAIL, c.NUM_TOTAL_TRANSACTIONS
HAVING MAX(s.SOLD_DATE) < DATEADD('day', -60, CURRENT_DATE)
ORDER BY c.NUM_TOTAL_TRANSACTIONS DESC;
```
