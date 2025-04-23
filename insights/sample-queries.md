# Insights Sample Queries

## 📊 **Category Sales Summary**
Summarize total sales and units by category and market for a given date range.

```sql
SELECT CATEGORY, MARKET, SUM(TOTAL_SALES) AS total_sales, SUM(TOTAL_UNITS) AS total_units
FROM CATEGORY
WHERE SOLD_ON BETWEEN '2024-01-01' AND '2024-01-31'
GROUP BY CATEGORY, MARKET
ORDER BY total_sales DESC;
```

## 📅 **Daily Product Performance**
Track daily sales, units, and average price per item for a product.

```sql
SELECT SOLD_ON, PRODUCT_NAME, BRAND_NAME, CATEGORY, SUM(TOTAL_SALES) AS sales, SUM(TOTAL_UNITS) AS units, AVG(AVERAGE_PRICE_PER_ITEM) AS avg_price
FROM DAILY_SALES
WHERE PRODUCT_NAME = 'ProductX'
  AND SOLD_ON BETWEEN '2024-04-01' AND '2024-04-07'
GROUP BY SOLD_ON, PRODUCT_NAME, BRAND_NAME, CATEGORY
ORDER BY SOLD_ON;
```

## 📆 **Monthly Brand Trends by Market**
Compare monthly sales and units for a brand across markets.

```sql
SELECT MARKET, SOLD_MONTH, SUM(TOTAL_SALES) AS sales, SUM(TOTAL_UNITS) AS units
FROM MONTHLY_SALES
WHERE BRAND_NAME = 'BrandY'
  AND SOLD_MONTH >= '2024-01-01'
GROUP BY MARKET, SOLD_MONTH
ORDER BY MARKET, SOLD_MONTH;
```

## 🧪 **THC & CBD Trait Distribution**
Analyze the distribution of THC and CBD content for products in a segment.

```sql
SELECT SEGMENT, PRODUCT_NAME, "trait_THC Max (%)", "trait_CBD Max (%)"
FROM DAILY_SALES
WHERE SEGMENT = 'Gummies' AND CATEGORY = 'Edibles'
ORDER BY "trait_THC Max (%)" DESC;
```

## 📦 **Popular Package Sizes**
See which package sizes are most common for a category and market.

```sql
SELECT CATEGORY, PACKAGE_SIZE, COUNT(*) AS sku_count, SUM(TOTAL_UNITS) AS total_units
FROM DAILY_SALES
WHERE MARKET = 'CA' AND CATEGORY = 'Concentrates'
GROUP BY CATEGORY, PACKAGE_SIZE
ORDER BY total_units DESC;
```
