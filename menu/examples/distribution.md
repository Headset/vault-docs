# Distribution Examples

Find where your brand is carried, where it isn't, and how your geographic reach compares to peers. Most of these queries use the multi-variant brand-pattern CTE — copy it once and tweak the patterns at the top.

> **First time querying?** Run the [name-variant discovery query](#discover-your-brands-name-variants-in-the-data) before anything else. Web-scraped brand names vary more than you expect — your first guess often returns zero rows.

---

## Discover your brand's name variants in the data

Start broad. Find every spelling the scraper has captured. Then plug the variants you care about into the `brand_patterns` CTE used by every other query in this file.

```sql
-- Start with a short fragment that's unlikely to collide (3-5 letters is good).
-- The output is every distinct brand_name containing that fragment, with SKU counts.
SELECT brand_name, COUNT(*) AS skus
FROM menuitems
WHERE brand_name ILIKE '%stii%'    -- ⬅ swap a fragment of your brand
GROUP BY 1
ORDER BY skus DESC;
```

If you get zero rows, try a shorter / different fragment. `STIIIZY` (three i's) lives in the data, but `%stiizy%` (two i's) finds nothing — case-insensitive but spelling-exact.

---

## Every store carrying your brand, by state

```sql
WITH brand_patterns AS (
  SELECT column1 AS pattern FROM VALUES
    ('%stiiizy%'),    -- ⬅ swap your variants (see discovery query above)
    ('%stizzy%')
)
SELECT
    s.state,
    s.name        AS store_name,
    s.city,
    s.dispensary_url,
    s.last_crawl_utc,
    COUNT(*)      AS skus_carried
FROM menuitems m
JOIN stores s USING (store_id)
WHERE EXISTS (
    SELECT 1 FROM brand_patterns bp WHERE m.brand_name ILIKE bp.pattern
)
GROUP BY 1, 2, 3, 4, 5
ORDER BY s.state, skus_carried DESC;
```

---

## Store and SKU counts per market

How many stores carry your brand in each state, and how deep is the assortment?

```sql
WITH brand_patterns AS (
  SELECT column1 AS pattern FROM VALUES
    ('%stiiizy%'),
    ('%stizzy%')
)
SELECT
    s.state,
    COUNT(DISTINCT s.store_id)   AS store_count,
    COUNT(DISTINCT m.product_id) AS distinct_skus,
    COUNT(*)                     AS menu_listings
FROM menuitems m
JOIN stores s USING (store_id)
WHERE EXISTS (
    SELECT 1 FROM brand_patterns bp WHERE m.brand_name ILIKE bp.pattern
)
GROUP BY 1
ORDER BY store_count DESC;
```

`store_count` is your market footprint. `distinct_skus` tells you whether you're going deep at each store or shallow.

---

## Reach comparison: your brand vs. up to three competitors

Side-by-side store count and SKU count in one market.

```sql
WITH targets AS (
    SELECT 'My brand'      AS label, '%stiiizy%' AS pattern  -- ⬅ your brand
    UNION ALL SELECT 'Competitor A', '%wyld%'                -- ⬅ competitor 1
    UNION ALL SELECT 'Competitor B', '%kiva%'                -- ⬅ competitor 2
    UNION ALL SELECT 'Competitor C', '%select%'              -- ⬅ competitor 3
)
SELECT
    t.label                       AS brand,
    COUNT(DISTINCT s.store_id)    AS store_count,
    COUNT(DISTINCT m.product_id)  AS distinct_skus
FROM targets t
JOIN menuitems m ON m.brand_name ILIKE t.pattern
JOIN stores    s ON s.store_id = m.store_id
WHERE s.state = 'CA'    -- ⬅ swap state
GROUP BY 1
ORDER BY store_count DESC;
```

---

## White-space: stores in a state that don't carry your brand

Lead list for sales — every store in the target market not yet selling you.

```sql
WITH brand_patterns AS (
  SELECT column1 AS pattern FROM VALUES
    ('%stiiizy%'),
    ('%stizzy%')
),
carrying_stores AS (
    SELECT DISTINCT m.store_id
    FROM menuitems m
    WHERE EXISTS (
        SELECT 1 FROM brand_patterns bp WHERE m.brand_name ILIKE bp.pattern
    )
)
SELECT
    s.store_id,
    s.name,
    s.city,
    s.state,
    s.product_count,
    s.brand_count,
    s.velocity_band,
    s.dispensary_url,
    s.phone_number
FROM stores s
LEFT JOIN carrying_stores c ON c.store_id = s.store_id
WHERE s.state = 'CA'              -- ⬅ swap state
  AND c.store_id IS NULL          -- not carrying
  AND s.last_crawl_utc >= DATEADD('day', -30, CURRENT_DATE)  -- recently observed
ORDER BY s.velocity_band, s.product_count DESC;
```

`velocity_band = 'High'` first surfaces the most attractive accounts.

---

## Stores within X miles of a target location, carrying your brand

Useful for field-sales territory planning or for a retailer scoping a competitive zone.

```sql
WITH brand_patterns AS (
  SELECT column1 AS pattern FROM VALUES
    ('%stiiizy%'),
    ('%stizzy%')
)
SELECT
    s.name,
    s.city,
    s.state,
    ROUND(ST_DISTANCE(s.location, ST_POINT(-122.4194, 37.7749)) / 1609.34, 2) AS miles_away,
    COUNT(*) AS skus
FROM menuitems m
JOIN stores s USING (store_id)
WHERE EXISTS (
    SELECT 1 FROM brand_patterns bp WHERE m.brand_name ILIKE bp.pattern
)
  AND ST_DWITHIN(s.location, ST_POINT(-122.4194, 37.7749), 16093.4)  -- 10 miles in meters
  -- ⬅ swap longitude (-122.4194), latitude (37.7749), and radius (10 miles → 16093.4 m)
GROUP BY 1, 2, 3, 4
ORDER BY miles_away;
```

`ST_POINT(longitude, latitude)` — longitude first. `ST_DWITHIN` and `ST_DISTANCE` use meters; 1 mile ≈ 1609.34 m.

---

## Recently-seen stores

Stores observed for the first time (or freshly re-crawled) in the last 30 days. Use this to catch new dispensaries entering a market.

```sql
SELECT
    s.store_id,
    s.name,
    s.city,
    s.state,
    s.last_crawl_utc,
    s.product_count,
    s.brand_count,
    s.velocity_band,
    s.dispensary_url
FROM stores s
WHERE s.last_crawl_utc >= DATEADD('day', -30, CURRENT_DATE)    -- ⬅ swap window
  AND s.state = 'CA'                                            -- ⬅ swap state
ORDER BY s.last_crawl_utc DESC;
```
