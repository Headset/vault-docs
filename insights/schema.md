# Insights Data Dictionary

Below are the main tables available in the Insights product, with key columns and example values for each. Column descriptions are provided where possible, otherwise left for your team to fill in.

## Table: `CATEGORY`
| Column         | Type   | Description                    | Example      |
|----------------|--------|--------------------------------|--------------|
| SOLD_ON        | DATE   | Date of sale                   | 2024-01-01   |
| CURRENCY_CODE  | TEXT   | Currency code                  | USD          |
| MARKET         | TEXT   | Market or region               | CA           |
| CATEGORY       | TEXT   | Product category               | Beverages    |
| TOTAL_SALES    | FLOAT  | Total sales for the category   | 15800.00     |
| TOTAL_UNITS    | FLOAT  | Total units sold               | 5000         |
| ACCOUNT_ID     | TEXT   | Account identifier             | ACCT-123     |

## Table: `DAILY_SALES`
| Column                    | Type    | Description                         | Example          |
|---------------------------|---------|-------------------------------------|------------------|
| SOLD_ON                   | DATE    | Date of sale                        | 2024-04-01       |
| INSIGHTS_PRODUCT_KEY      | TEXT    | Product key                         | PROD-001         |
| PRODUCT_NAME              | TEXT    | Product name                        | ProductX         |
| CURRENCY_CODE             | TEXT    | Currency code                       | USD              |
| MARKET                    | TEXT    | Market or region                    | CA               |
| CATEGORY                  | TEXT    | Product category                    | Edibles          |
| SEGMENT                   | TEXT    | Product segment or subcategory      | Gummies          |
| BRAND_NAME                | TEXT    | Brand name                          | BrandX           |
| UNIT_VALUE                | TEXT    | Unit value (e.g., size, weight)     | 10mg             |
| UNIT_MEASUREMENT          | TEXT    | Unit of measurement                 | Each             |
| PACKAGE_SIZE              | TEXT    | Package size                        | 10-pack          |
| 'trait_THC Max (%)'       | TEXT    | Max THC percent                     | 22.5             |
| 'trait_THC Max (mg/ml)'   | TEXT    | Max THC mg/ml                       | 18.0             |
| 'trait_THC Min (%)'       | TEXT    | Min THC percent                     | 18.0             |
| 'trait_THC Min (mg/ml)'   | TEXT    | Min THC mg/ml                       | 12.5             |
| 'trait_Total THC (mg)'    | TEXT    | Total THC mg                        | 200              |
| 'trait_CBD Focused'       | TEXT    | CBD focused (Y/N)                   | Y                |
| 'trait_CBD Max (%)'       | TEXT    | Max CBD percent                     | 5.0              |
| 'trait_CBD Max (mg/ml)'   | TEXT    | Max CBD mg/ml                       | 4.2              |
| 'trait_CBD Min (%)'       | TEXT    | Min CBD percent                     | 2.0              |
| 'trait_CBD Min (mg/ml)'   | TEXT    | Min CBD mg/ml                       | 1.2              |
| 'trait_Total CBD (mg)'    | TEXT    | Total CBD mg                        | 40               |
| 'trait_Volume (ml)'       | TEXT    | Volume in ml                        | 30               |
| CARTRIDGE_TYPE            | TEXT    | Cartridge type                      | 510-thread       |
| NUMBER_OF_SERVINGS        | NUMBER  | Number of servings                  | 10               |
| FLAVOR                    | TEXT    | Flavor description                  | Berry            |
| STRAIN_NAME               | TEXT    | Strain name                         | Blue Dream       |
| TOTAL_SALES               | NUMBER  | Total sales                         | 1200             |
| TOTAL_UNITS               | NUMBER  | Total units sold                    | 500              |
| TOTAL_COST                | NUMBER  | Total cost                          | 800              |
| TOTAL_DISCOUNT            | NUMBER  | Total discount                      | 50               |
| AVERAGE_PRICE_PER_ITEM    | NUMBER  | Average price per item              | 24.00            |
| AVERAGE_COST_PER_ITEM     | NUMBER  | Average cost per item               | 16.00            |
| ACCOUNT_ID                | TEXT    | Account identifier                  | ACCT-123         |

## Table: `MONTHLY_SALES`
| Column                    | Type    | Description                         | Example          |
|---------------------------|---------|-------------------------------------|------------------|
| SOLD_MONTH                | DATE    | Month of sale                       | 2024-04-01       |
| INSIGHTS_PRODUCT_KEY      | TEXT    | Product key                         | PROD-001         |
| PRODUCT_NAME              | TEXT    | Product name                        | ProductY         |
| CURRENCY_CODE             | TEXT    | Currency code                       | USD              |
| MARKET                    | TEXT    | Market or region                    | CO               |
| CATEGORY                  | TEXT    | Product category                    | Flower           |
| SEGMENT                   | TEXT    | Product segment or subcategory      | Sativa           |
| BRAND_NAME                | TEXT    | Brand name                          | BrandY           |
| UNIT_VALUE                | TEXT    | Unit value (e.g., size, weight)     | 1g               |
| UNIT_MEASUREMENT          | TEXT    | Unit of measurement                 | Gram             |
| PACKAGE_SIZE              | TEXT    | Package size                        | 1g               |
| 'trait_THC Max (%)'       | TEXT    | Max THC percent                     | 22.5             |
| 'trait_THC Max (mg/ml)'   | TEXT    | Max THC mg/ml                       | 18.0             |
| 'trait_THC Min (%)'       | TEXT    | Min THC percent                     | 18.0             |
| 'trait_THC Min (mg/ml)'   | TEXT    | Min THC mg/ml                       | 12.5             |
| 'trait_Total THC (mg)'    | TEXT    | Total THC mg                        | 200              |
| 'trait_CBD Focused'       | TEXT    | CBD focused (Y/N)                   | Y                |
| 'trait_CBD Max (%)'       | TEXT    | Max CBD percent                     | 5.0              |
| 'trait_CBD Max (mg/ml)'   | TEXT    | Max CBD mg/ml                       | 4.2              |
| 'trait_CBD Min (%)'       | TEXT    | Min CBD percent                     | 2.0              |
| 'trait_CBD Min (mg/ml)'   | TEXT    | Min CBD mg/ml                       | 1.2              |

## Table: `PACKAGE_SIZE`
| Column          | Type     | Description                                 | Example        |
|-----------------|----------|---------------------------------------------|----------------|
| SOLD_ON_DATE    | DATE     | Date of sale                                | 2024-03-01     |
| MARKET          | TEXT     | Market or region                            | Oregon         |
| CATEGORY        | TEXT     | Product category                            | Concentrates   |
| SEGMENT         | TEXT     | Product segment or subcategory              | Wax            |
| BRAND           | TEXT     | Brand name                                  | BrandZ         |
| TOTAL_NUMBER    | NUMBER   | Total units sold                            | 120            |
| TOTAL_UN_NUMBER | NUMBER   | Total unique package sizes                  | 4              |
