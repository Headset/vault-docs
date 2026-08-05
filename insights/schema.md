# Insights Data Dictionary

Below are the secure views available in the Insights product, with columns and example values for each.

## View: `CATEGORY`
| Column         | Type   | Description                    | Example      |
|----------------|--------|--------------------------------|--------------|
| SOLD_ON        | DATE   | Date of sale                   | 2024-01-01   |
| CURRENCY_CODE  | TEXT   | Currency code                  | USD          |
| MARKET         | TEXT   | Market or region               | CA           |
| CATEGORY       | TEXT   | Product category               | Beverages    |
| TOTAL_SALES    | FLOAT  | Total sales for the category   | 15800.00     |
| TOTAL_UNITS    | FLOAT  | Total units sold               | 5000         |
| ACCOUNT_ID     | TEXT   | Account identifier             | ACCT-123     |

## View: `DAILY_SALES`
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

## View: `MONTHLY_SALES`
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

## View: `PACKAGE_SIZE`
| Column          | Type     | Description                                 | Example        |
|-----------------|----------|---------------------------------------------|----------------|
| SOLD_ON         | DATE     | Date of sale                                | 2024-03-01     |
| CURRENCY_CODE   | TEXT     | Currency code                               | USD            |
| MARKET          | TEXT     | Market or region                            | Oregon         |
| CATEGORY        | TEXT     | Product category                            | Concentrates   |
| SEGMENT         | TEXT     | Product segment or subcategory              | Wax            |
| BRAND_NAME      | TEXT     | Brand name                                  | BrandZ         |
| PACKAGE_SIZE    | TEXT     | Package size                                | 1g             |
| TOTAL_SALES     | NUMBER   | Total sales                                 | 1200           |
| TOTAL_UNITS     | NUMBER   | Total units sold                            | 120            |
| ACCOUNT_ID      | TEXT     | Account identifier                          | ACCT-123       |

---

## Brand Loyalty Views

The four views below describe brand loyalty, retention, and cross-shopping — how a brand's customers behave over time — rather than sales dollars. They are built from a rolling trailing-365-day window split into two halves:

- **Prior window** (365–183 days ago): customers who bought the target brand here form its cohort.
- **Current window** (last 183 days): whether each of those customers returned, and to which brands, is measured here.

Each returning customer falls into exactly one bucket:

- **Loyal** — repurchased *only* the target brand.
- **Retained** — repurchased the target brand *and* at least one competitor.
- **Lost** — returned to the category but did *not* repurchase the target brand.

All customer counts are **sample counts** from connected retailers, not total market customers — use them for shares and comparisons, never as market sizing. Only brand / market / category combinations with **at least 1,000 returning customers** are included, so smaller brands will not appear.

## View: `BRAND_LOYALTY`

One row per market / category / brand.

| Column              | Type   | Description                                                                                  | Example    |
|---------------------|--------|----------------------------------------------------------------------------------------------|------------|
| MARKET              | TEXT   | Market or region (US state / Canadian province)                                              | CA         |
| CATEGORY            | TEXT   | Product category                                                                             | Vapor Pens |
| TARGET_BRAND        | TEXT   | The brand whose customer loyalty is described                                                 | BrandX     |
| RETURNING_CUSTOMERS | NUMBER | Distinct customers from the brand's prior cohort who returned to the category (loyal+retained+lost) | 4000       |
| LOYAL_CUSTOMERS     | NUMBER | Returning customers who repurchased only this brand                                           | 1520       |
| RETAINED_CUSTOMERS  | NUMBER | Returning customers who repurchased this brand and at least one competitor                    | 1240       |
| LOST_CUSTOMERS      | NUMBER | Returning customers who came back to the category but did not repurchase this brand           | 1240       |
| PCT_LOYAL           | NUMBER | LOYAL_CUSTOMERS as a percent of RETURNING_CUSTOMERS (0–100)                                   | 38.0       |
| PCT_RETAINED        | NUMBER | RETAINED_CUSTOMERS as a percent of RETURNING_CUSTOMERS                                        | 31.0       |
| PCT_LOST            | NUMBER | LOST_CUSTOMERS as a percent of RETURNING_CUSTOMERS                                            | 31.0       |
| PCT_REPEAT_BUYERS   | NUMBER | Share who repurchased the brand at all — PCT_LOYAL + PCT_RETAINED                             | 69.0       |
| ACCOUNT_ID          | TEXT   | Account identifier                                                                           | ACCT-123   |

## View: `BRAND_LOYALTY_BY_GENDER`

One row per market / category / brand / gender. Percentages are computed **within** each gender slice. A brand appears only if it clears the 1,000-customer threshold across all genders combined; individual gender rows may be smaller.

| Column              | Type   | Description                                                              | Example    |
|---------------------|--------|--------------------------------------------------------------------------|------------|
| MARKET              | TEXT   | Market or region                                                         | CA         |
| CATEGORY            | TEXT   | Product category                                                        | Vapor Pens |
| TARGET_BRAND        | TEXT   | The brand whose customer loyalty is described                            | BrandX     |
| GENDER              | TEXT   | Customer gender (`M`, `F`; may be null where unknown)                    | F          |
| RETURNING_CUSTOMERS | NUMBER | Returning customers in this gender slice (loyal+retained+lost)           | 1400       |
| LOYAL_CUSTOMERS     | NUMBER | Repurchased only this brand                                             | 520        |
| RETAINED_CUSTOMERS  | NUMBER | Repurchased this brand and at least one competitor                       | 430        |
| LOST_CUSTOMERS      | NUMBER | Returned to the category but not this brand                              | 450        |
| PCT_LOYAL           | NUMBER | LOYAL_CUSTOMERS as a percent of this slice's RETURNING_CUSTOMERS         | 37.1       |
| PCT_RETAINED        | NUMBER | RETAINED_CUSTOMERS as a percent of this slice's RETURNING_CUSTOMERS      | 30.7       |
| PCT_LOST            | NUMBER | LOST_CUSTOMERS as a percent of this slice's RETURNING_CUSTOMERS          | 32.1       |
| PCT_REPEAT_BUYERS   | NUMBER | PCT_LOYAL + PCT_RETAINED for this slice                                  | 67.9       |
| ACCOUNT_ID          | TEXT   | Account identifier                                                      | ACCT-123   |

## View: `BRAND_LOYALTY_BY_AGE_GROUP`

One row per market / category / brand / age group. Same columns and semantics as `BRAND_LOYALTY_BY_GENDER`, sliced by generational cohort instead of gender.

| Column              | Type   | Description                                                                      | Example      |
|---------------------|--------|----------------------------------------------------------------------------------|--------------|
| MARKET              | TEXT   | Market or region                                                                 | CA           |
| CATEGORY            | TEXT   | Product category                                                                | Vapor Pens   |
| TARGET_BRAND        | TEXT   | The brand whose customer loyalty is described                                    | BrandX       |
| AGE_GROUP           | TEXT   | Generational cohort (`Generation Z`, `Millennials`, `Generation X`, `Baby Boomers`) | Millennials  |
| RETURNING_CUSTOMERS | NUMBER | Returning customers in this age slice (loyal+retained+lost)                      | 1800         |
| LOYAL_CUSTOMERS     | NUMBER | Repurchased only this brand                                                     | 700          |
| RETAINED_CUSTOMERS  | NUMBER | Repurchased this brand and at least one competitor                               | 560          |
| LOST_CUSTOMERS      | NUMBER | Returned to the category but not this brand                                      | 540          |
| PCT_LOYAL           | NUMBER | LOYAL_CUSTOMERS as a percent of this slice's RETURNING_CUSTOMERS                 | 38.9         |
| PCT_RETAINED        | NUMBER | RETAINED_CUSTOMERS as a percent of this slice's RETURNING_CUSTOMERS              | 31.1         |
| PCT_LOST            | NUMBER | LOST_CUSTOMERS as a percent of this slice's RETURNING_CUSTOMERS                  | 30.0         |
| PCT_REPEAT_BUYERS   | NUMBER | PCT_LOYAL + PCT_RETAINED for this slice                                          | 70.0         |
| ACCOUNT_ID          | TEXT   | Account identifier                                                              | ACCT-123     |

## View: `BRAND_CROSS_SHOP`

One row per market / category / brand / competitor — the other brands a target brand's customers also buy. Only the top 10 competitors per brand are stored. Restricted to target brands that clear the 1,000-customer threshold.

| Column                  | Type   | Description                                                                       | Example    |
|-------------------------|--------|-----------------------------------------------------------------------------------|------------|
| MARKET                  | TEXT   | Market or region                                                                  | CA         |
| CATEGORY                | TEXT   | Product category                                                                 | Vapor Pens |
| TARGET_BRAND            | TEXT   | The brand whose customers' cross-shopping is described                             | BrandX     |
| OTHER_BRAND             | TEXT   | A competing brand that TARGET_BRAND's customers also purchased                     | BrandY     |
| OTHER_BRAND_RANK        | NUMBER | Rank of OTHER_BRAND as a cross-shop destination (1 = most cross-shopped; top 10)   | 1          |
| CROSS_SHOP_CUSTOMERS    | NUMBER | Distinct TARGET_BRAND returning customers who also bought OTHER_BRAND              | 340        |
| CROSS_SHOP_PCT_OF_BRAND | NUMBER | CROSS_SHOP_CUSTOMERS as a percent of TARGET_BRAND's returning customers            | 8.1        |
| ACCOUNT_ID              | TEXT   | Account identifier                                                                | ACCT-123   |

## View: `BRAND_LOYALTY_CATEGORY_AVG`

One row per market / category — the category-wide average loyalty profile, useful as a baseline to compare a specific brand against. Customer-weighted across all brands in the category, so larger brands contribute proportionally more. `AVG_PCT_LOYAL + AVG_PCT_RETAINED + AVG_PCT_LOST ≈ 100`.

| Column           | Type   | Description                                                                 | Example    |
|------------------|--------|------------------------------------------------------------------------------|------------|
| MARKET           | TEXT   | Market or region                                                             | CA         |
| CATEGORY         | TEXT   | Product category                                                            | Vapor Pens |
| AVG_PCT_LOYAL    | NUMBER | Category-average percent of customers loyal to a single brand (0–100)         | 14.6       |
| AVG_PCT_RETAINED | NUMBER | Category-average percent retained (repurchased their brand and others)       | 29.0       |
| AVG_PCT_LOST     | NUMBER | Category-average percent lost to competitors                                 | 56.3       |
| NUM_BRANDS       | NUMBER | Number of brands included in the category average                            | 387        |
| ACCOUNT_ID       | TEXT   | Account identifier                                                          | ACCT-123   |
