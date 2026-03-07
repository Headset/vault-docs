# Pricing and Product Attributes Examples

Queries for analyzing pricing trends, potency attributes, package sizes, and product characteristics that drive sales.

---

## Average Price Trends by Category

Track how pricing shifts over time.

```sql
SELECT
    SOLD_MONTH,
    CATEGORY,
    ROUND(AVG(AVERAGE_PRICE_PER_ITEM), 2) AS avg_price,
    ROUND(AVG(AVERAGE_COST_PER_ITEM), 2) AS avg_cost,
    ROUND(AVG(AVERAGE_PRICE_PER_ITEM) - AVG(AVERAGE_COST_PER_ITEM), 2) AS avg_margin
FROM MONTHLY_SALES
WHERE MARKET = 'CA'
  AND SOLD_MONTH >= DATEADD('year', -1, CURRENT_DATE)
GROUP BY SOLD_MONTH, CATEGORY
ORDER BY CATEGORY, SOLD_MONTH;
```

## Price Distribution by Segment

Understand the price range for products within a segment.

```sql
SELECT
    SEGMENT,
    COUNT(DISTINCT INSIGHTS_PRODUCT_KEY) AS product_count,
    ROUND(MIN(AVERAGE_PRICE_PER_ITEM), 2) AS min_price,
    ROUND(PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY AVERAGE_PRICE_PER_ITEM), 2) AS p25_price,
    ROUND(MEDIAN(AVERAGE_PRICE_PER_ITEM), 2) AS median_price,
    ROUND(PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY AVERAGE_PRICE_PER_ITEM), 2) AS p75_price,
    ROUND(MAX(AVERAGE_PRICE_PER_ITEM), 2) AS max_price
FROM DAILY_SALES
WHERE MARKET = 'CA'
  AND CATEGORY = 'Flower'
  AND SOLD_ON >= DATEADD('day', -30, CURRENT_DATE)
  AND AVERAGE_PRICE_PER_ITEM > 0
GROUP BY SEGMENT
ORDER BY median_price DESC;
```

## Top Package Sizes by Category

See which package sizes drive the most volume.

```sql
SELECT
    CATEGORY,
    SEGMENT,
    PACKAGE_SIZE,
    SUM(TOTAL_SALES) AS total_sales,
    SUM(TOTAL_UNITS) AS total_units,
    ROUND(
        SUM(TOTAL_UNITS) / NULLIF(SUM(SUM(TOTAL_UNITS)) OVER (PARTITION BY CATEGORY), 0) * 100,
        1
    ) AS pct_of_category_units
FROM PACKAGE_SIZE
WHERE MARKET = 'CA'
  AND SOLD_ON >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY CATEGORY, SEGMENT, PACKAGE_SIZE
ORDER BY CATEGORY, total_units DESC;
```

## Package Size Trends Over Time

Track whether consumers are shifting to larger or smaller sizes.

```sql
SELECT
    DATE_TRUNC('month', SOLD_ON) AS month,
    PACKAGE_SIZE,
    SUM(TOTAL_SALES) AS total_sales,
    SUM(TOTAL_UNITS) AS total_units
FROM PACKAGE_SIZE
WHERE MARKET = 'CA'
  AND CATEGORY = 'Flower'
  AND SOLD_ON >= DATEADD('year', -1, CURRENT_DATE)
GROUP BY month, PACKAGE_SIZE
ORDER BY month, total_units DESC;
```

## Package Size by Brand

See which package sizes a brand offers and how they sell.

```sql
SELECT
    BRAND_NAME,
    PACKAGE_SIZE,
    SUM(TOTAL_SALES) AS total_sales,
    SUM(TOTAL_UNITS) AS total_units
FROM PACKAGE_SIZE
WHERE MARKET = 'CA'
  AND CATEGORY = 'Concentrates'
  AND SOLD_ON >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY BRAND_NAME, PACKAGE_SIZE
ORDER BY BRAND_NAME, total_sales DESC;
```

## THC Potency Distribution

Analyze how THC content correlates with sales in a segment.

```sql
SELECT
    "trait_THC Max (%)" AS thc_max_pct,
    COUNT(DISTINCT INSIGHTS_PRODUCT_KEY) AS product_count,
    SUM(TOTAL_SALES) AS total_sales,
    SUM(TOTAL_UNITS) AS total_units,
    ROUND(AVG(AVERAGE_PRICE_PER_ITEM), 2) AS avg_price
FROM DAILY_SALES
WHERE MARKET = 'CA'
  AND CATEGORY = 'Flower'
  AND "trait_THC Max (%)" IS NOT NULL
  AND SOLD_ON >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY thc_max_pct
ORDER BY thc_max_pct;
```

## CBD-Focused Products

Find CBD-focused products and their sales performance.

```sql
SELECT
    PRODUCT_NAME,
    BRAND_NAME,
    CATEGORY,
    SEGMENT,
    "trait_CBD Max (%)" AS cbd_max_pct,
    "trait_Total CBD (mg)" AS total_cbd_mg,
    SUM(TOTAL_SALES) AS total_sales,
    SUM(TOTAL_UNITS) AS total_units
FROM DAILY_SALES
WHERE MARKET = 'CA'
  AND "trait_CBD Focused" = 'Y'
  AND SOLD_ON >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY PRODUCT_NAME, BRAND_NAME, CATEGORY, SEGMENT, cbd_max_pct, total_cbd_mg
ORDER BY total_sales DESC
LIMIT 25;
```

## Strain Performance

See which strains are most popular.

```sql
SELECT
    STRAIN_NAME,
    CATEGORY,
    COUNT(DISTINCT INSIGHTS_PRODUCT_KEY) AS product_count,
    COUNT(DISTINCT BRAND_NAME) AS brand_count,
    SUM(TOTAL_SALES) AS total_sales,
    SUM(TOTAL_UNITS) AS total_units
FROM DAILY_SALES
WHERE MARKET = 'CA'
  AND STRAIN_NAME IS NOT NULL
  AND SOLD_ON >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY STRAIN_NAME, CATEGORY
ORDER BY total_sales DESC
LIMIT 25;
```

## Flavor Analysis

See which flavors consumers prefer in a segment.

```sql
SELECT
    FLAVOR,
    COUNT(DISTINCT INSIGHTS_PRODUCT_KEY) AS product_count,
    SUM(TOTAL_SALES) AS total_sales,
    SUM(TOTAL_UNITS) AS total_units,
    ROUND(AVG(AVERAGE_PRICE_PER_ITEM), 2) AS avg_price
FROM DAILY_SALES
WHERE MARKET = 'CA'
  AND CATEGORY = 'Edibles'
  AND SEGMENT = 'Gummies'
  AND FLAVOR IS NOT NULL
  AND SOLD_ON >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY FLAVOR
ORDER BY total_sales DESC;
```

## Cartridge Type Breakdown

Compare sales across cartridge types in the vape category.

```sql
SELECT
    CARTRIDGE_TYPE,
    COUNT(DISTINCT BRAND_NAME) AS brand_count,
    SUM(TOTAL_SALES) AS total_sales,
    SUM(TOTAL_UNITS) AS total_units,
    ROUND(AVG(AVERAGE_PRICE_PER_ITEM), 2) AS avg_price
FROM DAILY_SALES
WHERE MARKET = 'CA'
  AND CATEGORY = 'Vapor Pens'
  AND CARTRIDGE_TYPE IS NOT NULL
  AND SOLD_ON >= DATEADD('day', -30, CURRENT_DATE)
GROUP BY CARTRIDGE_TYPE
ORDER BY total_sales DESC;
```

## Discount Rate by Category

Understand how heavily each category is being discounted.

```sql
SELECT
    CATEGORY,
    SUM(TOTAL_SALES) AS net_sales,
    SUM(TOTAL_DISCOUNT) AS total_discounts,
    SUM(TOTAL_COST) AS total_cost,
    ROUND(
        SUM(TOTAL_DISCOUNT) / NULLIF(SUM(TOTAL_SALES) + SUM(TOTAL_DISCOUNT), 0) * 100,
        1
    ) AS discount_rate_pct
FROM MONTHLY_SALES
WHERE MARKET = 'CA'
  AND SOLD_MONTH >= DATEADD('month', -3, CURRENT_DATE)
GROUP BY CATEGORY
ORDER BY discount_rate_pct DESC;
```
