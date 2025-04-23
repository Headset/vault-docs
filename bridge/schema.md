# Bridge Data Dictionary

Below are the main tables available in the Bridge product, with key columns and example values for each. Column descriptions are provided where possible, otherwise left for your team to fill in.

## Table: `INVENTORY_HISTORY`
| Column                   | Type           | Description                  | Example         |
|--------------------------|----------------|------------------------------|-----------------|
| STORE_ID                 | NUMBER         | Store ID                     | 101             |
| STORE_NAME               | TEXT           | Store name                   | Store A         |
| STOCK_DATE               | DATE           | Stock date                   | 2024-01-15      |
| USE_UNTIL                | DATE           | Use until date               | 2024-02-15      |
| PRODUCT_ID               | TEXT           | Product ID                   | PROD-001        |
| CURRENT_QUANTITY_IN_STOCK| NUMBER         | Current quantity in stock    | 30              |
| AVG_PRICE_PER_UNIT       | FLOAT          | Average price per unit       | 10.50           |
| QUANTITY_IN_STOCK        | FLOAT          | Quantity in stock            | 50.0            |
| AVG_COST_PER_UNIT        | FLOAT          | Average cost per unit        | 9.75            |
| LAST_UPDATED             | TIMESTAMP_LTZ  | Last updated timestamp       | 2024-01-15 09:00:00 |
| BRIDGE_SHARE_STOCK_ID    | TEXT           | Bridge share stock ID        | BRIDGE-001      |
| ACCOUNT_ID               | TEXT           | Account ID                   | ACCT-123        |

## Table: `PRODUCTS`
| Column                   | Type           | Description                  | Example         |
|--------------------------|----------------|------------------------------|-----------------|
| PRODUCT_ID               | TEXT           | Product ID                   | PROD-001        |
| STORE_ID                 | NUMBER         | Store ID                     | 101             |
| STORE_NAME               | TEXT           | Store name                   | Store A         |
| PRODUCT_NAME             | TEXT           | Product name                 | Gummy Bears     |
| BRAND                    | TEXT           | Brand name                   | BrandX          |
| CATEGORY                 | TEXT           | Product category             | Edibles         |
| UNIT                     | TEXT           | Unit of measurement          | Each            |
| VENDOR                   | TEXT           | Vendor name                  | VendorX         |
| STRAIN                   | TEXT           | Strain                       | Blue Dream      |
| THC_RESULT               | TEXT           | THC result                   | 18.0%           |
| CBD_RESULT               | TEXT           | CBD result                   | 0.2%            |
| NON_CANNABIS             | NUMBER         | Non-cannabis flag            | 0               |
| PREDICTED_CATEGORY       | TEXT           | Predicted category           | Candy           |
| FIRST_SALE               | TIMESTAMP_NTZ  | First sale timestamp         | 2024-01-01 10:00:00 |
| LAST_SALE                | TIMESTAMP_NTZ  | Last sale timestamp          | 2024-04-01 12:00:00 |
| QUANTITY_IN_STOCK        | NUMBER         | Quantity in stock            | 50              |
| FIRST_STOCK_UPDATE       | TIMESTAMP_NTZ  | First stock update           | 2024-01-01 09:00:00 |
| LAST_STOCK_UPDATE        | TIMESTAMP_NTZ  | Last stock update            | 2024-04-01 09:00:00 |
| SKU_AGG                  | ARRAY          | SKU aggregation              | [SKU-1, SKU-2]  |
| _LAST_UPDATE             | TIMESTAMP_LTZ  | Last update timestamp        | 2024-04-01 10:00:00 |
| VENDOR_LOWER             | TEXT           | Vendor (lowercase)           | vendorx         |
| BRAND_LOWER              | TEXT           | Brand (lowercase)            | brandx          |
| VENDOR_BRAND_LOWER       | TEXT           | Vendor/Brand (lowercase)     | vendorx_brandx  |
| ACCOUNT_ID               | TEXT           | Account ID                   | ACCT-123        |

## Table: `SALES`
| Column                   | Type           | Description                  | Example         |
|--------------------------|----------------|------------------------------|-----------------|
| STORE_ID                 | NUMBER         | Store ID                     | 101             |
| STORE_NAME               | TEXT           | Store name                   | Store A         |
| RECEIPT_ITEM_ID          | NUMBER         | Receipt item ID              | 1001            |
| RECEIPT_ID               | NUMBER         | Receipt ID                   | 5001            |
| SOLD_DATE                | TIMESTAMP_NTZ  | Sold date                    | 2024-04-01 13:45:00 |
| PRODUCT_NAME             | TEXT           | Product name                 | Gummy Bears     |
| QUANTITY                 | FLOAT          | Quantity sold                | 2.0             |
| UNIT_COST                | FLOAT          | Unit cost                    | 9.50            |
| BASE_PRICE               | NUMBER         | Base price                   | 19.99           |
| DISCOUNT                 | NUMBER         | Discount                     | 2.00            |
| TOTAL_COST               | FLOAT          | Total cost                   | 17.99           |
| REVENUE                  | NUMBER         | Revenue                      | 17.99           |
| CUSTOMER_KEY             | TEXT           | Customer key                 | CUST-001        |
| AGE_AT_PURCHASE          | NUMBER         | Age at purchase              | 35              |
| PRODUCT_ID               | TEXT           | Product ID                   | PROD-001        |
| SKU                      | TEXT           | SKU                          | SKU-123         |
| DATA_SOURCE_NAME         | TEXT           | Data source name             | POS             |
| CREATED_ON               | TIMESTAMP_NTZ  | Created on                   | 2024-04-01 14:00:00 |
| METRC_PACKAGES           | ARRAY          | METRC packages               | [PKG-1, PKG-2]  |
| _LAST_UPDATE             | TIMESTAMP_LTZ  | Last update timestamp        | 2024-04-01 15:00:00 |
| ACCOUNT_ID               | TEXT           | Account ID                   | ACCT-123        |

## Table: `STORES`
| Column                   | Type           | Description                  | Example         |
|--------------------------|----------------|------------------------------|-----------------|
| STORE_ID                 | NUMBER         | Store ID                     | 101             |
| STORE_NAME               | TEXT           | Store name                   | Store A         |
| ADDRESS                  | TEXT           | Address                      | 123 Main St     |
| CITY                     | TEXT           | City                         | Seattle         |
| STATE                    | TEXT           | State                        | WA              |

## Table: `brand_affinity`
| Column         | Type      | Description                              | Example         |
|---------------|-----------|------------------------------------------|----------------|
| customer_id   | STRING    | Unique customer identifier               | CUST-1001      |
| brand         | STRING    | Brand name                               | BrandX         |
| affinity_score| FLOAT     | Calculated brand affinity score          | 0.87           |

## Table: `segment_overlap`
| Column         | Type      | Description                              | Example         |
|---------------|-----------|------------------------------------------|----------------|
| segment_a     | STRING    | First segment name                       | "Loyal Shoppers" |
| segment_b     | STRING    | Second segment name                      | "BrandX Fans"    |
| overlap_pct   | FLOAT     | Percentage overlap between segments      | 42.5           |

## Table: `market_penetration`
| Column         | Type      | Description                              | Example         |
|---------------|-----------|------------------------------------------|----------------|
| brand         | STRING    | Brand name                               | BrandX         |
| region        | STRING    | Geographic region                        | Northwest      |
| penetration   | FLOAT     | Market penetration percentage            | 18.2           |
