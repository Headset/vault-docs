# Loyalty and Cross-Shop Examples

Queries for understanding customer retention, repeat purchase, and which brands compete for the same customers.

Every metric in these views is pre-computed at a fixed grain — one row is one answer — so most of these are a plain `SELECT … WHERE` with no aggregation. Percentages are already 0–100. Replace `'Your Brand'`, `MARKET`, and `CATEGORY` with your own.

Loyalty is measured over a trailing 365-day window; only brands with at least 1,000 returning customers in a market/category appear. See the [Schema Reference](../schema.md#brand-loyalty-views) for the loyal / retained / lost definitions.

---

## Loyalty at a Glance

How loyal is a brand's customer base in a market and category?

```sql
SELECT
    RETURNING_CUSTOMERS,
    PCT_LOYAL,          -- repurchased only this brand
    PCT_RETAINED,       -- repurchased this brand and others
    PCT_LOST,           -- returned to the category but not this brand
    PCT_REPEAT_BUYERS   -- loyal + retained
FROM BRAND_LOYALTY
WHERE TARGET_BRAND = 'Your Brand'
  AND MARKET = 'CA'
  AND CATEGORY = 'Vapor Pens';
```

## Loyalty by Gender

Break the same loyalty profile down by customer gender.

```sql
SELECT
    GENDER,
    RETURNING_CUSTOMERS,
    PCT_LOYAL,
    PCT_REPEAT_BUYERS
FROM BRAND_LOYALTY_BY_GENDER
WHERE TARGET_BRAND = 'Your Brand'
  AND MARKET = 'CA'
  AND CATEGORY = 'Vapor Pens'
ORDER BY GENDER;
```

## Loyalty by Age Group

The same, sliced by generational cohort — useful for spotting which segments are most and least loyal.

```sql
SELECT
    AGE_GROUP,
    RETURNING_CUSTOMERS,
    PCT_LOYAL,
    PCT_LOST,
    PCT_REPEAT_BUYERS
FROM BRAND_LOYALTY_BY_AGE_GROUP
WHERE TARGET_BRAND = 'Your Brand'
  AND MARKET = 'CA'
  AND CATEGORY = 'Vapor Pens'
ORDER BY PCT_REPEAT_BUYERS DESC;
```

## Top Competitors (Cross-Shop)

Which other brands do your customers also buy? Ranked, with the share of your returning customers who cross-shop each one.

```sql
SELECT
    OTHER_BRAND,
    OTHER_BRAND_RANK,
    CROSS_SHOP_CUSTOMERS,
    CROSS_SHOP_PCT_OF_BRAND
FROM BRAND_CROSS_SHOP
WHERE TARGET_BRAND = 'Your Brand'
  AND MARKET = 'CA'
  AND CATEGORY = 'Vapor Pens'
ORDER BY OTHER_BRAND_RANK
LIMIT 10;
```

## Who Cross-Shops Your Brand (Reverse)

For which brands is *your* brand a top competitor? Flip the filter to `OTHER_BRAND` to see whose customers are drifting toward you.

```sql
SELECT
    TARGET_BRAND,
    OTHER_BRAND_RANK,          -- your rank as a competitor for that brand
    CROSS_SHOP_CUSTOMERS,
    CROSS_SHOP_PCT_OF_BRAND
FROM BRAND_CROSS_SHOP
WHERE OTHER_BRAND = 'Your Brand'
  AND MARKET = 'CA'
  AND CATEGORY = 'Vapor Pens'
ORDER BY CROSS_SHOP_CUSTOMERS DESC;
```

## Brand vs Category Baseline

Compare a brand's retention against the category average, to tell brand-specific softness from broad category churn.

```sql
SELECT
    b.TARGET_BRAND,
    b.PCT_LOST                         AS brand_pct_lost,
    a.AVG_PCT_LOST                     AS category_pct_lost,
    ROUND(b.PCT_LOST - a.AVG_PCT_LOST, 1) AS pts_vs_category   -- positive = churns more than the category
FROM BRAND_LOYALTY b
JOIN BRAND_LOYALTY_CATEGORY_AVG a
  ON a.MARKET = b.MARKET
 AND a.CATEGORY = b.CATEGORY
WHERE b.TARGET_BRAND = 'Your Brand'
  AND b.MARKET = 'CA'
  AND b.CATEGORY = 'Vapor Pens';
```

## Most Loyal Brands in a Category

Rank every qualifying brand in a market and category by loyalty.

```sql
SELECT
    TARGET_BRAND,
    RETURNING_CUSTOMERS,
    PCT_LOYAL,
    PCT_REPEAT_BUYERS
FROM BRAND_LOYALTY
WHERE MARKET = 'CA'
  AND CATEGORY = 'Vapor Pens'
ORDER BY PCT_LOYAL DESC
LIMIT 25;
```

## Brands Losing the Most Customers vs the Category

Find the brands whose churn is furthest above the category baseline — the most at-risk in a market.

```sql
SELECT
    b.TARGET_BRAND,
    b.RETURNING_CUSTOMERS,
    b.PCT_LOST,
    ROUND(b.PCT_LOST - a.AVG_PCT_LOST, 1) AS pts_above_category
FROM BRAND_LOYALTY b
JOIN BRAND_LOYALTY_CATEGORY_AVG a
  ON a.MARKET = b.MARKET
 AND a.CATEGORY = b.CATEGORY
WHERE b.MARKET = 'CA'
  AND b.CATEGORY = 'Vapor Pens'
ORDER BY pts_above_category DESC
LIMIT 25;
```
