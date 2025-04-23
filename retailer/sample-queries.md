# Retailer Sample Queries

## 🛒 **Customer Lifetime Transactions**
Find the total number of transactions for each customer.

```sql
SELECT CUSTOMER_KEY, FIRST_NAME, NUM_TOTAL_TRANSACTIONS
FROM CUSTOMERS
ORDER BY NUM_TOTAL_TRANSACTIONS DESC;
```

## 🏬 **Inventory On Hand by Store**
See current inventory in stock for each product at each store.

```sql
SELECT STORE_ID, PRODUCT_ID, QUANTITY_IN_STOCK
FROM INVENTORY_BATCHES
ORDER BY STORE_ID, PRODUCT_ID;
```

## 📦 **Inventory History: Stock Levels Over Time**
Track how inventory levels change over time for a given product.

```sql
SELECT STOCK_DATE, PRODUCT_ID, QUANTITY_IN_STOCK
FROM INVENTORY_HISTORY
WHERE PRODUCT_ID = 'PROD-001'
ORDER BY STOCK_DATE;
```

## 🏷️ **Product Catalog: Pricing and Brand**
List all products with their price and brand.

```sql
SELECT PRODUCT_ID, NAME, BRAND, CATEGORY, PRICE
FROM PRODUCTS
ORDER BY BRAND, NAME;
```

## 💳 **Recent Sales by Payment Type**
Show recent sales transactions and payment types.

```sql
SELECT RECEIPT_ID, SOLD_DATE, PRODUCT_NAME, BRAND, PAYMENT_TYPE
FROM SALES
WHERE SOLD_DATE >= '2024-04-01'
ORDER BY SOLD_DATE DESC;
```
