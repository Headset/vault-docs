# Glossary

A quick reference for key terms you’ll encounter in Vault’s data products.

| Term               | Definition                                                                 |
|--------------------|---------------------------------------------------------------------------|
| SKU                | Stock Keeping Unit – a unique identifier for each distinct product/item    |
| Basket             | A single transaction or purchase event (may contain multiple SKUs)         |
| Brand Affinity     | Measure of customer preference or loyalty toward a brand                   |
| Market Penetration | The extent to which a product or brand is recognized within its market     |
| Category           | A classification grouping similar products                                 |
| Segment            | A secondary classification of products, or subcategory            |
| Store              | A retail location or online shop                                           |
| Repeat Purchase    | When a customer buys the same SKU more than once                          |
| Bridge Table       | A table connecting two datasets (e.g., customers and products)             |
| Insights           | Actionable findings derived from data analysis                             |
| Velocity Score     | Headset's relative measure of how fast a store moves product, modeled from observed menu activity     |
| Velocity Band      | Categorical bucket of velocity (`High` / `Medium` / `Low`) used for quick filtering and segmenting     |
| Est. Days on Hand  | Projected days until a product runs out at the current observed sell rate                              |
| On-shelf           | A product currently visible on a dispensary menu — distinct from sold-through (actual sales)           |
| Fuzzy Brand Match  | Web-scraped `brand_name` values vary in spelling/case; use `ILIKE '%pattern%'` rather than `=`         |
| Menu Event         | One observed change to a dispensary menu (product added/removed, price move, low stock, brand added/removed, new store) — a row in Menu Data's `EVENTS` view |
| Door               | A store carrying at least one of a brand's listings; a door is gained on `BRAND_ADDED` and lost on `BRAND_REMOVED` |
| Returning Customer | In brand-loyalty views: a customer from a brand's prior-window cohort who came back to the category in the current window (the denominator for loyalty rates) |
| Loyal Customer     | A returning customer who repurchased *only* the target brand                                          |
| Retained Customer  | A returning customer who repurchased the target brand *and* at least one competitor                   |
| Lost Customer      | A returning customer who came back to the category but did *not* repurchase the target brand          |
| Repeat Buyer Rate  | Share of a brand's returning customers who repurchased it at all — loyal + retained                    |
| Cross-Shop         | When a brand's customers also buy competing brands; `BRAND_CROSS_SHOP` ranks a brand's top competitors |
