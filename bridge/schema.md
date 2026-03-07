# Bridge Data Dictionary

Below are the secure views available in the Bridge product, with columns and example values for each.

## View: `INVENTORY_BATCHES`
| Column                   | Type           | Description                                    | Example         |
|--------------------------|----------------|------------------------------------------------|-----------------|
| INVENTORY_EVENT_ID       | NUMBER         | Headset record identifier                      | 10029291        |
| BRIDGE_COMPANY_ID        | NUMBER         | Bridge company identifier                      | 4521            |
| POS_PRODUCT_ID           | TEXT           | Point of sale product identifier               | ABC123          |
| PRODUCT_HASH             | TEXT           | Hashed product identifier                      | a1b2c3d4        |
| DATA_SOURCE_ID           | NUMBER         | Data source identifier                         | 3               |
| STORE_ID                 | NUMBER         | Store ID                                       | 101             |
| RETAILER_PRODUCT_ID      | TEXT           | Retailer-assigned product identifier           | RP-0042         |
| BATCH_ID                 | TEXT           | POS package ID                                 | 100001          |
| BATCH_ITEM_ID            | TEXT           | POS line item ID                               | 100001-ABC      |
| VENDOR_NAME              | TEXT           | Name of vendor for batch                       | Herby's Distro  |
| RECEIVE_DATE_UTC         | TIMESTAMP_NTZ  | Timestamp package was received (UTC)           | 2025-05-20 20:18:00.000 |
| RECEIVE_DATE_LOCAL       | DATE           | Day package was received (store's local time)  | 2025-04-20      |
| QUANTITY                 | NUMBER         | Original quantity received                     | 100             |
| QUANTITY_IN_STOCK        | NUMBER         | Current quantity in stock                      | 90              |
| COST_PER_UNIT            | NUMBER         | Unit cost                                      | 37.50           |
| TEST_RESULTS             | VARIANT        | Test results for batch                         | `{"THC9_Milligrams": 4.69}` |
| EXTRA_DATA               | VARIANT        | Additional data                                | `{"MetrcPackage": "1A406..."}` |
| _LAST_UPDATE             | TIMESTAMP_LTZ  | Last update timestamp                          | 2025-05-29 00:16:47.057 |
| ACCOUNT_ID               | TEXT           | Account ID                                     | ACCT-123        |

## View: `INVENTORY_HISTORY`
| Column                   | Type           | Description                                    | Example         |
|--------------------------|----------------|------------------------------------------------|-----------------|
| STORE_ID                 | NUMBER         | Store ID                                       | 101             |
| STORE_NAME               | TEXT           | Store name                                     | Store A         |
| STOCK_DATE               | DATE           | Stock date                                     | 2024-01-15      |
| USE_UNTIL                | DATE           | Use until date                                 | 2024-02-15      |
| PRODUCT_ID               | TEXT           | Product ID                                     | PROD-001        |
| CURRENT_QUANTITY_IN_STOCK| NUMBER         | Current quantity in stock                      | 30              |
| AVG_PRICE_PER_UNIT       | FLOAT          | Average price per unit                         | 10.50           |
| QUANTITY_IN_STOCK        | FLOAT          | Quantity in stock                              | 50.0            |
| AVG_COST_PER_UNIT        | FLOAT          | Average cost per unit                          | 9.75            |
| LAST_UPDATED             | TIMESTAMP_LTZ  | Last updated timestamp                         | 2024-01-15 09:00:00 |
| BRIDGE_SHARE_STOCK_ID    | TEXT           | Bridge share stock ID                          | BRIDGE-001      |
| ACCOUNT_ID               | TEXT           | Account ID                                     | ACCT-123        |
| POS_PRODUCT_ID           | TEXT           | Point of sale product identifier               | ABC123          |

## View: `PRODUCTS`
| Column                   | Type           | Description                                    | Example         |
|--------------------------|----------------|------------------------------------------------|-----------------|
| PRODUCT_ID               | TEXT           | Product ID                                     | PROD-001        |
| STORE_ID                 | NUMBER         | Store ID                                       | 101             |
| STORE_NAME               | TEXT           | Store name                                     | Store A         |
| PRODUCT_NAME             | TEXT           | Product name                                   | Gummy Bears     |
| PRODUCT_TYPE             | TEXT           | Product type                                   | Candy           |
| BRAND                    | TEXT           | Brand name                                     | BrandX          |
| CATEGORY                 | TEXT           | Product category                               | Edibles         |
| UNIT                     | TEXT           | Unit of measurement                            | Each            |
| ROOM_INVENTORY           | TEXT           | Room or storage location                       | Display         |
| VENDOR                   | TEXT           | Vendor name                                    | VendorX         |
| STRAIN                   | TEXT           | Strain                                         | Blue Dream      |
| THC_RESULT               | TEXT           | THC result                                     | 18.0%           |
| CBD_RESULT               | TEXT           | CBD result                                     | 0.2%            |
| NON_CANNABIS             | NUMBER         | Non-cannabis flag                              | 0               |
| PREDICTED_CATEGORY       | TEXT           | Predicted category                             | Candy           |
| FIRST_SALE               | TIMESTAMP_NTZ  | First sale timestamp                           | 2024-01-01 10:00:00 |
| LAST_SALE                | TIMESTAMP_NTZ  | Last sale timestamp                            | 2024-04-01 12:00:00 |
| QUANTITY_IN_STOCK        | NUMBER         | Quantity in stock                              | 50              |
| FIRST_STOCK_UPDATE       | TIMESTAMP_NTZ  | First stock update                             | 2024-01-01 09:00:00 |
| LAST_STOCK_UPDATE        | TIMESTAMP_NTZ  | Last stock update                              | 2024-04-01 09:00:00 |
| SKU_AGG                  | ARRAY          | SKU aggregation                                | ["SKU-1", "SKU-2"] |
| _LAST_UPDATE             | TIMESTAMP_LTZ  | Last update timestamp                          | 2024-04-01 10:00:00 |
| VENDOR_LOWER             | TEXT           | Vendor (lowercase)                             | vendorx         |
| BRAND_LOWER              | TEXT           | Brand (lowercase)                              | brandx          |
| VENDOR_BRAND_LOWER       | TEXT           | Vendor/Brand (lowercase)                       | vendorx_brandx  |
| ACCOUNT_ID               | TEXT           | Account ID                                     | ACCT-123        |
| POS_PRODUCT_ID           | TEXT           | Point of sale product identifier               | ABC123          |

## View: `PRODUCT_LATEST_PACKAGE`
| Column                   | Type           | Description                                    | Example         |
|--------------------------|----------------|------------------------------------------------|-----------------|
| STORE_ID                 | NUMBER         | Store ID                                       | 101             |
| STORE_NAME               | TEXT           | Store name                                     | Store A         |
| PRODUCT_ID               | TEXT           | Product ID                                     | PROD-001        |
| PRODUCT_NAME             | TEXT           | Product name                                   | Gummy Bears     |
| LATEST_PACKAGE_ID        | TEXT           | Most recent METRC/package ID for the product   | 1A406030000B... |

## View: `SALES`
| Column                   | Type           | Description                                    | Example         |
|--------------------------|----------------|------------------------------------------------|-----------------|
| STORE_ID                 | NUMBER         | Store ID                                       | 101             |
| STORE_NAME               | TEXT           | Store name                                     | Store A         |
| RECEIPT_ITEM_ID          | NUMBER         | Receipt item ID                                | 1001            |
| RECEIPT_ID               | NUMBER         | Receipt ID                                     | 5001            |
| POS_RECEIPT_ID           | TEXT           | POS system receipt ID                          | POSR-001        |
| SOLD_DATE                | TIMESTAMP_NTZ  | Sold date                                      | 2024-04-01 13:45:00 |
| PRODUCT_NAME             | TEXT           | Product name                                   | Gummy Bears     |
| QUANTITY                 | FLOAT          | Quantity sold                                  | 2.0             |
| UNIT_COST                | FLOAT          | Unit cost                                      | 9.50            |
| BASE_PRICE               | NUMBER         | Base price                                     | 19.99           |
| DISCOUNT                 | NUMBER         | Discount                                       | 2.00            |
| TAX                      | NUMBER         | Tax amount                                     | 1.50            |
| TOTAL_COST               | FLOAT          | Total cost                                     | 17.99           |
| REVENUE                  | NUMBER         | Revenue                                        | 17.99           |
| CUSTOMER_KEY             | TEXT           | Customer key                                   | CUST-001        |
| AGE_AT_PURCHASE          | NUMBER         | Age at purchase                                | 35              |
| PRODUCT_ID               | TEXT           | Product ID                                     | PROD-001        |
| SKU                      | TEXT           | SKU                                            | SKU-123         |
| DATA_SOURCE_NAME         | TEXT           | Data source name                               | POS             |
| CREATED_ON               | TIMESTAMP_NTZ  | Created on                                     | 2024-04-01 14:00:00 |
| METRC_PACKAGES           | ARRAY          | METRC packages                                 | ["PKG-1", "PKG-2"] |
| _LAST_UPDATE             | TIMESTAMP_LTZ  | Last update timestamp                          | 2024-04-01 15:00:00 |
| ACCOUNT_ID               | TEXT           | Account ID                                     | ACCT-123        |
| POS_PRODUCT_ID           | TEXT           | Point of sale product identifier               | ABC123          |
| ORDER_SOURCE             | TEXT           | Order source (e.g., in-store, online)          | In-Store        |

## View: `STORES`
| Column                   | Type           | Description                                    | Example         |
|--------------------------|----------------|------------------------------------------------|-----------------|
| STORE_ID                 | NUMBER         | Store ID                                       | 101             |
| STORE_NAME               | TEXT           | Store name                                     | Store A         |
| ADDRESS                  | TEXT           | Address                                        | 123 Main St     |
| CITY                     | TEXT           | City                                           | Seattle         |
| STATE                    | TEXT           | State                                          | WA              |
| POSTAL_CODE              | TEXT           | Postal code                                    | 98101           |
| SIGNUP_DATE              | TIMESTAMP_NTZ  | Date the store signed up                       | 2023-06-15 00:00:00 |
| QUANTITY_IN_STOCK        | NUMBER         | Total quantity in stock at the store           | 5000            |
| LAST_STORE_STOCK_UPDATE  | TIMESTAMP_NTZ  | Last stock update for the store                | 2024-04-01 09:00:00 |
| CUSTOM_ID                | TEXT           | Custom store identifier                        | CUST-STORE-01   |
| ACCOUNT_ID               | TEXT           | Account ID                                     | ACCT-123        |
