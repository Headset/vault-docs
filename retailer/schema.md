# Retailer Data Dictionary

Below are the main tables available in the Retailer product, with key columns and example values for each. Column descriptions are provided where possible, otherwise left for your team to fill in.


## Table: `stores`
| Column     | Type    | Description                       | Example      |
|------------|---------|-----------------------------------|--------------|
| store_id   | STRING  | Unique store identifier           | STORE-001    |
| region     | STRING  | Geographic region                 | Northwest    |
| opened_on  | DATE    | Store opening date                | 2020-08-01   |

## Table: `CUSTOMERS`
| Column                   | Type      | Description                        | Example         |
|--------------------------|-----------|------------------------------------|-----------------|
| CUSTOMER_KEY             | TEXT      | Customer unique key                | CUST-001        |
| FIRST_NAME               | TEXT      | First name                         | Jane            |
| FIRST_TRANSACTION_DATE   | DATE      | First transaction date             | 2023-01-15      |
| GENDER                   | TEXT      | Gender                             | F               |
| BIRTH_YEAR               | NUMBER    | Birth year                         | 1988            |
| AGE                      | TEXT      | Age                                | 35              |
| REFERRAL_SOURCE          | TEXT      | Referral source                    | Friend          |
| POS_CUSTOMER_ID          | TEXT      | POS system customer ID             | POS1234         |
| PHONE_NUMBER             | TEXT      | Phone number                       | 555-123-4567    |
| EMAIL                    | TEXT      | Email address                      | jane@email.com  |
| NUM_TOTAL_TRANSACTIONS   | NUMBER    | Number of transactions             | 42              |
| EXTRA_DATA               | VARIANT   | Extra customer data                | {...}           |
| STORE_ID                 | NUMBER    | Store ID                           | 101             |
| ACCOUNT_ID               | TEXT      | Account ID                         | ACCT-123        |

## Table: `INVENTORY_BATCHES`
| Column                   | Type           | Description                        | Example         |
|--------------------------|----------------|------------------------------------|-----------------|
| STORE_ID                 | NUMBER         | Store ID                           | 101             |
| PRODUCT_ID               | TEXT           | Product ID                         | PROD-001        |
| BATCH_ID                 | TEXT           | Batch ID                           | BATCH-01        |
| BATCH_ITEM_ID            | TEXT           | Batch item ID                      | BITEM-001       |
| POS_PRODUCT_ID           | TEXT           | POS system product ID              | POSP-001        |
| QUANTITY                 | NUMBER         | Quantity in batch                  | 100             |
| COST_PER_UNIT            | NUMBER         | Cost per unit                      | 12.50           |
| RECEIVE_DATE_UTC         | TIMESTAMP_NTZ  | Received date (UTC)                | 2024-01-01 08:00:00 |
| RECEIVE_DATE_LOCAL       | DATE           | Received date (local)              | 2024-01-01      |
| VENDOR_NAME              | TEXT           | Vendor name                        | VendorX         |
| QUANTITY_IN_STOCK        | NUMBER         | Quantity in stock                  | 75              |
| TEST_RESULTS             | VARIANT        | Test results                       | {...}           |
| ACCOUNT_ID               | TEXT           | Account ID                         | ACCT-123        |
| EXTRA_DATA               | VARIANT        | Extra batch data                   | {...}           |

## Table: `INVENTORY_HISTORY`
| Column                   | Type      | Description                        | Example         |
|--------------------------|-----------|------------------------------------|-----------------|
| STOCK_DATE               | DATE      | Stock date                         | 2024-01-15      |
| STORE_ID                 | NUMBER    | Store ID                           | 101             |
| STORE_NAME               | TEXT      | Store name                         | Store A         |
| PRODUCT_ID               | TEXT      | Product ID                         | PROD-001        |
| QUANTITY_IN_STOCK        | FLOAT     | Quantity in stock                  | 50.0            |
| AVG_COST_PER_UNIT        | FLOAT     | Average cost per unit              | 11.75           |
| ACCOUNT_ID               | TEXT      | Account ID                         | ACCT-123        |

## Table: `PRODUCTS`
| Column                   | Type           | Description                        | Example         |
|--------------------------|----------------|------------------------------------|-----------------|
| PRODUCT_ID               | TEXT           | Product ID                         | PROD-001        |
| STORE_ID                 | NUMBER         | Store ID                           | 101             |
| POS_PRODUCT_ID           | TEXT           | POS system product ID              | POSP-001        |
| SKU                      | TEXT           | SKU                                | SKU-123         |
| NAME                     | TEXT           | Product name                       | Gummy Bears     |
| BRAND                    | TEXT           | Brand name                         | BrandX          |
| CATEGORY                 | TEXT           | Product category                   | Edibles         |
| UNIT                     | TEXT           | Unit of measurement                | Each            |
| VENDOR                   | TEXT           | Vendor name                        | VendorX         |
| STRAIN                   | TEXT           | Strain                             | Blue Dream      |
| THC_RESULT               | TEXT           | THC result                         | 18.0%           |
| CBD_RESULT               | TEXT           | CBD result                         | 0.2%            |
| PRODUCT_TYPE             | TEXT           | Product type                       | Candy           |
| QUANTITY_IN_STOCK        | NUMBER         | Quantity in stock                  | 50              |
| PRICE                    | FLOAT          | Price                              | 19.99           |
| EXTRA_DATA               | VARIANT        | Extra product data                 | {...}           |
| _LAST_UPDATE             | TIMESTAMP_LTZ  | Last update timestamp              | 2024-04-01 10:00:00 |

## Table: `SALES`
| Column                   | Type           | Description                        | Example         |
|--------------------------|----------------|------------------------------------|-----------------|
| RECEIPT_ITEM_ID          | NUMBER         | Receipt item ID                    | 1001            |
| RECEIPT_ID               | NUMBER         | Receipt ID                         | 5001            |
| RECEIPT_TYPE             | TEXT           | Receipt type                       | Sale            |
| PAYMENT_TYPE             | TEXT           | Payment type                       | Credit Card     |
| SOLD_DATE                | TIMESTAMP_NTZ  | Sold date                          | 2024-04-01 13:45:00 |
| PRODUCT_ID               | TEXT           | Product ID                         | PROD-001        |
| POS_RECEIPT_ID           | TEXT           | POS system receipt ID              | POSR-001        |
| PRODUCT_NAME             | TEXT           | Product name                       | Gummy Bears     |
| BRAND                    | TEXT           | Brand name                         | BrandX          |
