# Insights Data Product

Insights provides aggregated market-level sales data across the cannabis industry. Unlike Bridge and Retailer — which deliver your own store-level data — Insights gives you a view of the broader market, so you can benchmark your performance, track category trends, and understand competitive dynamics.

All data is aggregated and anonymized. You will never see individual store or transaction-level detail in Insights.

## What's Included

Insights delivers nine secure views, in two families.

**Sales & products** — market sales activity:

| View | Description |
|------|-------------|
| **CATEGORY** | Daily category-level sales and units by market — useful for high-level trend analysis |
| **DAILY_SALES** | Daily product-level sales data with full product attributes: brand, category, segment, package size, potency traits, and pricing metrics |
| **MONTHLY_SALES** | The same product-level detail as `DAILY_SALES`, rolled up to the month — better for trend analysis over longer time horizons |
| **PACKAGE_SIZE** | Sales and units broken down by package size, brand, category, and segment |

**Brand loyalty** — how a brand's customers behave over time (retention, repeat purchase, and cross-shopping), measured over a trailing 365-day window:

| View | Description |
|------|-------------|
| **BRAND_LOYALTY** | Per-brand loyalty: what share of a brand's returning customers are loyal, retained, or lost, with the underlying counts |
| **BRAND_LOYALTY_BY_GENDER** | The same loyalty breakdown, sliced by customer gender |
| **BRAND_LOYALTY_BY_AGE_GROUP** | The same loyalty breakdown, sliced by generational cohort |
| **BRAND_CROSS_SHOP** | A brand's top competitors — the other brands its customers also buy, ranked, with cross-shop share |
| **BRAND_LOYALTY_CATEGORY_AVG** | The category-wide average loyalty profile, as a baseline to compare a specific brand against |

## Data Structure

Each view is pre-aggregated at a fixed grain, and every metric is already computed — you read a row, you don't have to assemble the answer. The **sales views** are independent (not designed to be joined). The **brand-loyalty views** are a related family: they share the `MARKET` / `CATEGORY` / `TARGET_BRAND` keys, so you can move between them (a brand's overall loyalty → by gender or age → its competitors → the category baseline).

- **CATEGORY** — One row per market/category/date. The broadest sales view.
- **DAILY_SALES** — One row per product/market/date. The most granular sales view.
- **MONTHLY_SALES** — One row per product/market/month. Same grain as daily, rolled up.
- **PACKAGE_SIZE** — One row per brand/category/segment/package size/date.
- **BRAND_LOYALTY** — One row per market/category/brand.
- **BRAND_LOYALTY_BY_GENDER** — One row per market/category/brand/gender.
- **BRAND_LOYALTY_BY_AGE_GROUP** — One row per market/category/brand/age group.
- **BRAND_CROSS_SHOP** — One row per market/category/brand/competitor (top 10 competitors per brand).
- **BRAND_LOYALTY_CATEGORY_AVG** — One row per market/category.

### About the brand-loyalty views

Loyalty is measured over a rolling trailing-365-day window: a brand's customers from the **prior** half (365–183 days ago) are the cohort, and their behavior in the **current** half (last 183 days) sorts each into one bucket — **loyal** (repurchased only this brand), **retained** (this brand and others), or **lost** (returned to the category but not this brand). "Repeat buyers" = loyal + retained. Customer figures are **sample counts** from connected retailers, not total market counts, so use them for shares and comparisons — never as market sizing. Only brands with **at least 1,000 returning customers** in a market/category are included, so smaller brands won't appear; the demographic views apply that threshold at the brand level and then show every gender/age slice. See the [Schema Reference](./schema.md#brand-loyalty-views) for full column definitions.

## Common Use Cases

- **Market sizing** — Use `CATEGORY` to understand total sales and unit volume by market and category over time
- **Brand benchmarking** — Compare your brand's sales against the broader market or specific competitors using `DAILY_SALES` or `MONTHLY_SALES`
- **Category and segment trends** — Track how categories and segments are growing or declining across markets
- **Product attribute analysis** — Filter by potency traits (THC/CBD), package size, strain, flavor, or cartridge type to understand what product attributes are driving sales
- **New product tracking** — Monitor how recently launched products are performing relative to established SKUs
- **Pricing intelligence** — Use `AVERAGE_PRICE_PER_ITEM` and `AVERAGE_COST_PER_ITEM` to understand pricing trends by category, segment, or brand
- **Package size optimization** — Use `PACKAGE_SIZE` to see which sizes drive the most volume in a given category and market
- **Customer retention & churn** — Use `BRAND_LOYALTY` to see what share of a brand's customers are loyal, cross-shopping, or lost to competitors
- **Competitive dynamics** — Use `BRAND_CROSS_SHOP` to see which brands a target brand's customers also buy — and, in reverse, which brands treat yours as a top competitor
- **Loyalty benchmarking** — Compare a brand's retention against the category baseline with `BRAND_LOYALTY_CATEGORY_AVG`
- **Loyalty by demographic** — Use `BRAND_LOYALTY_BY_GENDER` / `BRAND_LOYALTY_BY_AGE_GROUP` to see how retention differs across customer segments

## Choosing Between Daily and Monthly

`DAILY_SALES` and `MONTHLY_SALES` share the same columns (except `SOLD_ON` vs `SOLD_MONTH`) and the same product-level grain. Use daily when you need precise timing — launch dates, promotional impact, day-of-week patterns. Use monthly when you're looking at longer-term trends, as it's smaller and faster to query.

## Example Queries

Ready-to-run SQL organized by topic:

- [Market Trends](./examples/market-trends.md) — Category sizing, market share, segment growth, cross-market comparisons
- [Brand Analysis](./examples/brand-analysis.md) — Brand rankings, share trends, new launches, competitive benchmarking
- [Pricing and Products](./examples/pricing-and-products.md) — Price trends, package sizes, potency, strains, flavors, cartridge types
- [Loyalty and Cross-Shop](./examples/loyalty-and-cross-shop.md) — Retention, repeat-purchase rate, demographic splits, top competitors, brand-vs-category baseline

## Further Reading

- [Schema Reference](./schema.md) — Full column definitions and data types for every view
- [Snowflake Basics](../snowflake-basics.md) — Connection setup and authentication
- [Glossary](../glossary.md) — Definitions of key terms
