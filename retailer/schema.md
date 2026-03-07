# Retailer Data Dictionary

Below are the secure views available in the Retailer product, with columns and example values for each.

## View: `CUSTOMERS`
| Column                   | Type           | Description                                    | Example         |
|--------------------------|----------------|------------------------------------------------|-----------------|
| CUSTOMER_KEY             | TEXT           | Customer unique key                            | CUST-001        |
| FIRST_NAME               | TEXT           | First name                                     | Jane            |
| FIRST_TRANSACTION_DATE   | DATE           | First transaction date                         | 2023-01-15      |
| GENDER                   | TEXT           | Gender                                         | F               |
| BIRTH_YEAR               | NUMBER         | Birth year                                     | 1988            |
| AGE                      | TEXT           | Age                                            | 35              |
| REFERRAL_SOURCE          | TEXT           | Referral source                                | Friend          |
| POS_CUSTOMER_ID          | TEXT           | POS system customer ID                         | POS1234         |
| PHONE_NUMBER             | TEXT           | Phone number                                   | 555-123-4567    |
| EMAIL                    | TEXT           | Email address                                  | jane@email.com  |
| NUM_TOTAL_TRANSACTIONS   | NUMBER         | Number of transactions                         | 42              |
| EXTRA_DATA               | VARIANT        | Extra customer data                            | {...}           |
| COMPANY_ID               | NUMBER         | Company identifier                             | 4521            |
| POSTAL_CODE              | TEXT           | Customer postal code                           | 98101           |
| STORE_ID                 | NUMBER         | Store ID                                       | 101             |
| ACCOUNT_ID               | TEXT           | Account ID                                     | ACCT-123        |

## View: `INVENTORY_BATCHES`
| Column                   | Type           | Description                                    | Example         |
|--------------------------|----------------|------------------------------------------------|-----------------|
| INVENTORY_EVENT_ID       | NUMBER         | Headset record identifier                      | 10029291        |
| STORE_ID                 | NUMBER         | Store ID                                       | 101             |
| PRODUCT_ID               | TEXT           | Product ID                                     | PROD-001        |
| BATCH_ID                 | TEXT           | Batch ID                                       | BATCH-01        |
| BATCH_ITEM_ID            | TEXT           | Batch item ID                                  | BITEM-001       |
| POS_PRODUCT_ID           | TEXT           | POS system product ID                          | POSP-001        |
| QUANTITY                 | NUMBER         | Quantity in batch                              | 100             |
| COST_PER_UNIT            | NUMBER         | Cost per unit                                  | 12.50           |
| RECEIVE_DATE_UTC         | TIMESTAMP_NTZ  | Received date (UTC)                            | 2024-01-01 08:00:00 |
| RECEIVE_DATE_LOCAL       | DATE           | Received date (local)                          | 2024-01-01      |
| VENDOR_NAME              | TEXT           | Vendor name                                    | VendorX         |
| QUANTITY_IN_STOCK        | NUMBER         | Quantity in stock                              | 75              |
| TEST_RESULTS             | VARIANT        | Test results                                   | {...}           |
| ACCOUNT_ID               | TEXT           | Account ID                                     | ACCT-123        |
| EXTRA_DATA               | VARIANT        | Extra batch data                               | {...}           |
| _LAST_UPDATE             | TIMESTAMP_LTZ  | Last update timestamp                          | 2024-04-01 10:00:00 |

## View: `INVENTORY_HISTORY`
| Column                   | Type           | Description                                    | Example         |
|--------------------------|----------------|------------------------------------------------|-----------------|
| STOCK_DATE               | DATE           | Stock date                                     | 2024-01-15      |
| STORE_ID                 | NUMBER         | Store ID                                       | 101             |
| STORE_NAME               | TEXT           | Store name                                     | Store A         |
| PRODUCT_ID               | TEXT           | Product ID                                     | PROD-001        |
| QUANTITY_IN_STOCK        | FLOAT          | Quantity in stock                              | 50.0            |
| AVG_COST_PER_UNIT        | FLOAT          | Average cost per unit                          | 11.75           |
| ACCOUNT_ID               | TEXT           | Account ID                                     | ACCT-123        |

## View: `PRODUCTS`
| Column                   | Type           | Description                                    | Example         |
|--------------------------|----------------|------------------------------------------------|-----------------|
| PRODUCT_ID               | TEXT           | Product ID                                     | PROD-001        |
| STORE_ID                 | NUMBER         | Store ID                                       | 101             |
| POS_PRODUCT_ID           | TEXT           | POS system product ID                          | POSP-001        |
| SKU                      | TEXT           | SKU                                            | SKU-123         |
| NAME                     | TEXT           | Product name                                   | Gummy Bears     |
| BRAND                    | TEXT           | Brand name                                     | BrandX          |
| CATEGORY                 | TEXT           | Product category                               | Edibles         |
| UNIT                     | TEXT           | Unit of measurement                            | Each            |
| VENDOR                   | TEXT           | Vendor name                                    | VendorX         |
| STRAIN                   | TEXT           | Strain                                         | Blue Dream      |
| THC_RESULT               | TEXT           | THC result                                     | 18.0%           |
| CBD_RESULT               | TEXT           | CBD result                                     | 0.2%            |
| PRODUCT_TYPE             | TEXT           | Product type                                   | Candy           |
| QUANTITY_IN_STOCK        | NUMBER         | Quantity in stock                              | 50              |
| PRICE                    | FLOAT          | Price                                          | 19.99           |
| EXTRA_DATA               | VARIANT        | Extra product data                             | {...}           |
| _LAST_UPDATE             | TIMESTAMP_LTZ  | Last update timestamp                          | 2024-04-01 10:00:00 |

## View: `SALES`
| Column                   | Type           | Description                                    | Example         |
|--------------------------|----------------|------------------------------------------------|-----------------|
| RECEIPT_ITEM_ID          | NUMBER         | Receipt item ID                                | 1001            |
| RECEIPT_ID               | NUMBER         | Receipt ID                                     | 5001            |
| RECEIPT_TYPE             | TEXT           | Receipt type                                   | Sale            |
| PAYMENT_TYPE             | TEXT           | Payment type                                   | Credit Card     |
| SOLD_DATE                | TIMESTAMP_NTZ  | Sold date                                      | 2024-04-01 13:45:00 |
| PRODUCT_ID               | TEXT           | Product ID                                     | PROD-001        |
| POS_RECEIPT_ID           | TEXT           | POS system receipt ID                          | POSR-001        |
| PRODUCT_NAME             | TEXT           | Product name                                   | Gummy Bears     |
| BRAND                    | TEXT           | Brand name                                     | BrandX          |
| CATEGORY                 | TEXT           | Product category                               | Edibles         |
| UNIT                     | TEXT           | Unit of measurement                            | Each            |
| VENDOR                   | TEXT           | Vendor name                                    | VendorX         |
| SKU                      | TEXT           | SKU                                            | SKU-123         |
| CUSTOM_GROUP             | TEXT           | Custom grouping label                          | Group A         |
| STORE_ID                 | NUMBER         | Store ID                                       | 101             |
| STORE_NAME               | TEXT           | Store name                                     | Store A         |
| CITY                     | TEXT           | City                                           | Seattle         |
| STATE                    | TEXT           | State                                          | WA              |
| COUNTRY                  | TEXT           | Country                                        | US              |
| IS_MEDICAL               | BOOLEAN        | Whether the sale was medical                   | false           |
| EMPLOYEE_KEY             | TEXT           | Employee key                                   | EMP-001         |
| DISCOUNTS_USED           | ARRAY          | Discounts applied to the sale                  | ["10OFF", "BOGO"] |
| EMPLOYEE_FIRST_NAME      | TEXT           | Employee first name                            | Alex            |
| EMPLOYEE_LAST_NAME       | TEXT           | Employee last name                             | Smith           |
| EMPLOYEE_NAME            | TEXT           | Employee full name                             | Alex Smith      |
| QUANTITY                 | FLOAT          | Quantity sold                                  | 2.0             |
| WEIGHT                   | FLOAT          | Weight sold                                    | 3.5             |
| BASE_PRICE               | NUMBER         | Base price                                     | 19.99           |
| TAX                      | NUMBER         | Tax amount                                     | 1.50            |
| DISCOUNT                 | NUMBER         | Discount amount                                | 2.00            |
| TOTAL_COST               | FLOAT          | Total cost                                     | 17.99           |
| REVENUE                  | NUMBER         | Revenue                                        | 17.99           |
| PROFIT                   | FLOAT          | Profit                                         | 8.49            |
| CUSTOMER_KEY             | TEXT           | Customer key                                   | CUST-001        |
| CUSTOMER_FIRST_NAME      | TEXT           | Customer first name                            | Jane            |
| CUSTOMER_GENDER          | TEXT           | Customer gender                                | F               |
| CUSTOMER_YEAR_OF_BIRTH   | DATE           | Customer year of birth                         | 1988-01-01      |
| CUSTOMER_AGE_GROUP       | TEXT           | Customer age group                             | 30-39           |
| CUSTOMER_POSTAL_CODE     | TEXT           | Customer postal code                           | 98101           |
| CUSTOMER_REFERRAL_SOURCE | TEXT           | Customer referral source                       | Friend          |
| CUSTOMER_PHONE_NUMBER    | TEXT           | Customer phone number                          | 555-123-4567    |
| CUSTOMER_EMAIL           | TEXT           | Customer email                                 | jane@email.com  |
| CUSTOMER_EXTRA_DATA      | VARIANT        | Extra customer data                            | {...}           |
| POS_CUSTOMER_ID          | TEXT           | POS system customer ID                         | POS1234         |
| IS_DELIVERY              | BOOLEAN        | Whether the sale was a delivery                | false           |
| ORDER_SOURCE             | TEXT           | Order source (e.g., in-store, online)          | In-Store        |
| IS_WHOLESALE             | BOOLEAN        | Whether the sale was wholesale                 | false           |
| EXTRA_DATA               | VARIANT        | Extra sale data                                | {...}           |
| DATA_SOURCE_NAME         | TEXT           | Data source name                               | POS             |
| BATCH_ITEM_ID            | TEXT           | Batch item ID                                  | BITEM-001       |
| _LAST_UPDATE             | TIMESTAMP_LTZ  | Last update timestamp                          | 2024-04-01 15:00:00 |
| ACCOUNT_ID               | TEXT           | Account ID                                     | ACCT-123        |

## View: `STORES`
| Column                   | Type           | Description                                    | Example         |
|--------------------------|----------------|------------------------------------------------|-----------------|
| STORE_ID                 | NUMBER         | Store ID                                       | 101             |
| NAME                     | TEXT           | Store name                                     | Store A         |
| COMPANY_NAME             | TEXT           | Company name                                   | Cannabis Co     |
| ADDRESS                  | TEXT           | Address                                        | 123 Main St     |
| CITY                     | TEXT           | City                                           | Seattle         |
| STATE                    | TEXT           | State                                          | WA              |
| COUNTRY                  | TEXT           | Country                                        | US              |
| POSTAL_CODE              | TEXT           | Postal code                                    | 98101           |
| CUSTOM_GROUP             | TEXT           | Custom grouping label                          | Group A         |
| TIME_ZONE                | TEXT           | Store time zone                                | America/Los_Angeles |
| CURRENT_CONNECTION       | BOOLEAN        | Whether the store is currently connected       | true            |
| SQUARE_FEET              | NUMBER         | Store square footage                           | 2500            |
