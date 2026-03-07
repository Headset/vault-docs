# Batches and Compliance Examples

Queries for batch-level traceability, METRC package tracking, test result lookups, and expiration monitoring.

---

## Latest METRC Package by Product

Look up the most recent METRC/package ID for each product at each store.

```sql
SELECT
    STORE_NAME,
    PRODUCT_NAME,
    PRODUCT_ID,
    LATEST_PACKAGE_ID
FROM PRODUCT_LATEST_PACKAGE
ORDER BY STORE_NAME, PRODUCT_NAME;
```

## Batch Detail for a Product

View all batches received for a specific product, including test results and cost.

```sql
SELECT
    BATCH_ID,
    BATCH_ITEM_ID,
    VENDOR_NAME,
    RECEIVE_DATE_LOCAL,
    QUANTITY,
    QUANTITY_IN_STOCK,
    COST_PER_UNIT,
    TEST_RESULTS,
    EXTRA_DATA
FROM INVENTORY_BATCHES
WHERE POS_PRODUCT_ID = 'your-pos-product-id'
ORDER BY RECEIVE_DATE_LOCAL DESC;
```

## THC/CBD Test Results Across Batches

Extract potency test results from the TEST_RESULTS variant column.

```sql
SELECT
    BATCH_ID,
    VENDOR_NAME,
    RECEIVE_DATE_LOCAL,
    TEST_RESULTS:"THC9_Milligrams"::FLOAT AS thc_mg,
    TEST_RESULTS:"CBD_Milligrams"::FLOAT AS cbd_mg,
    TEST_RESULTS:"THC_Percent"::FLOAT AS thc_pct,
    TEST_RESULTS:"CBD_Percent"::FLOAT AS cbd_pct
FROM INVENTORY_BATCHES
WHERE TEST_RESULTS IS NOT NULL
ORDER BY RECEIVE_DATE_LOCAL DESC;
```

> **Note:** The keys inside `TEST_RESULTS` vary by POS system and state. Use `SELECT DISTINCT OBJECT_KEYS(TEST_RESULTS) FROM INVENTORY_BATCHES` to discover available keys in your data.

## METRC Package Lookup from Batch Extra Data

Extract METRC package IDs from the EXTRA_DATA variant column.

```sql
SELECT
    BATCH_ID,
    VENDOR_NAME,
    RECEIVE_DATE_LOCAL,
    EXTRA_DATA:"MetrcPackage"::TEXT AS metrc_package_id,
    EXTRA_DATA:"SourcePackageId"::TEXT AS source_package_id,
    QUANTITY,
    QUANTITY_IN_STOCK
FROM INVENTORY_BATCHES
WHERE EXTRA_DATA:"MetrcPackage" IS NOT NULL
ORDER BY RECEIVE_DATE_LOCAL DESC;
```

## METRC Packages on Sales

Find which METRC packages were associated with specific sales transactions.

```sql
SELECT
    SOLD_DATE::DATE AS sale_date,
    STORE_NAME,
    PRODUCT_NAME,
    RECEIPT_ID,
    QUANTITY,
    REVENUE,
    METRC_PACKAGES
FROM SALES
WHERE ARRAY_SIZE(METRC_PACKAGES) > 0
  AND SOLD_DATE >= DATEADD('day', -30, CURRENT_DATE)
ORDER BY SOLD_DATE DESC;
```

## Batches Expiring Soon

Find inventory with upcoming USE_UNTIL dates using INVENTORY_HISTORY.

```sql
SELECT
    STORE_NAME,
    PRODUCT_ID,
    QUANTITY_IN_STOCK,
    USE_UNTIL,
    DATEDIFF('day', CURRENT_DATE, USE_UNTIL) AS days_until_expiry
FROM INVENTORY_HISTORY
WHERE USE_UNTIL IS NOT NULL
  AND USE_UNTIL BETWEEN CURRENT_DATE AND DATEADD('day', 30, CURRENT_DATE)
  AND STOCK_DATE = (SELECT MAX(STOCK_DATE) FROM INVENTORY_HISTORY)
  AND QUANTITY_IN_STOCK > 0
ORDER BY USE_UNTIL ASC;
```

## Cost Analysis by Vendor and Batch

Compare cost per unit across vendors and batches.

```sql
SELECT
    VENDOR_NAME,
    COUNT(DISTINCT BATCH_ID) AS batch_count,
    SUM(QUANTITY) AS total_units_received,
    ROUND(AVG(COST_PER_UNIT), 2) AS avg_cost_per_unit,
    MIN(COST_PER_UNIT) AS min_cost,
    MAX(COST_PER_UNIT) AS max_cost
FROM INVENTORY_BATCHES
WHERE RECEIVE_DATE_LOCAL >= DATEADD('month', -6, CURRENT_DATE)
GROUP BY VENDOR_NAME
ORDER BY total_units_received DESC;
```

## Batch Remaining Stock

See which batches still have stock remaining and how much has sold through.

```sql
SELECT
    BATCH_ID,
    VENDOR_NAME,
    RECEIVE_DATE_LOCAL,
    QUANTITY AS original_quantity,
    QUANTITY_IN_STOCK AS remaining,
    QUANTITY - QUANTITY_IN_STOCK AS units_sold,
    ROUND(
        (QUANTITY - QUANTITY_IN_STOCK) / NULLIF(QUANTITY, 0) * 100,
        1
    ) AS sell_through_pct
FROM INVENTORY_BATCHES
WHERE QUANTITY > 0
ORDER BY RECEIVE_DATE_LOCAL DESC;
```
