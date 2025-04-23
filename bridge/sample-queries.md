# Bridge Sample Queries

## 🏪 **Current Inventory by Store and Product**
Get the current quantity in stock for each product at each store.

```sql
SELECT STORE_ID, STORE_NAME, PRODUCT_ID, CURRENT_QUANTITY_IN_STOCK
FROM INVENTORY_HISTORY
ORDER BY STORE_ID, PRODUCT_ID;
```

## 🕒 **Inventory History: Stock and Price Trends**
Track inventory levels and average price per unit for a product over time.

```sql
SELECT STOCK_DATE, PRODUCT_ID, QUANTITY_IN_STOCK, AVG_PRICE_PER_UNIT
FROM INVENTORY_HISTORY
WHERE PRODUCT_ID = 'PROD-001'
ORDER BY STOCK_DATE;
```

## 🏷️ **Product Catalog: Brand and Predicted Category**
List all products with their brand and predicted category.

```sql
SELECT PRODUCT_ID, PRODUCT_NAME, BRAND, PREDICTED_CATEGORY
FROM PRODUCTS
ORDER BY BRAND, PRODUCT_NAME;
```

## 💲 **Sales Revenue by Store and Product**
Summarize sales revenue for each product and store.

```sql
SELECT STORE_ID, STORE_NAME, PRODUCT_NAME, SUM(REVENUE) AS total_revenue
FROM SALES
GROUP BY STORE_ID, STORE_NAME, PRODUCT_NAME
ORDER BY total_revenue DESC;
```

## 🧑‍🤝‍🧑 **Customer Demographics: Age at Purchase**
Analyze customer age distribution for purchases of a specific product.

```sql
SELECT PRODUCT_NAME, AGE_AT_PURCHASE, COUNT(*) AS purchase_count
FROM SALES
WHERE PRODUCT_NAME = 'Gummy Bears'
GROUP BY PRODUCT_NAME, AGE_AT_PURCHASE
ORDER BY AGE_AT_PURCHASE;
```

## 🏬 **Store Directory**
List all stores and their locations.

```sql
SELECT STORE_ID, STORE_NAME, ADDRESS, CITY, STATE
FROM STORES
ORDER BY STORE_NAME;
```
