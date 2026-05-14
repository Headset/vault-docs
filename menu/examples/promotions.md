# Promotions Examples

Track the active and recent deal landscape — on your brand, on competitors, and across stores in your area.

> If you haven't yet, run the [name-variant discovery query](./distribution.md#discover-your-brands-name-variants-in-the-data) first to confirm you're matching the right SKUs.

---

## Active deals featuring your brand

Currently-running deals whose `BRANDS` array includes one of your brand variants.

```sql
WITH brand_patterns AS (
  SELECT column1 AS pattern FROM VALUES
    ('%stiiizy%'),
    ('%stizzy%')
)
SELECT
    d.deal_id,
    d.title,
    d.description,
    d.discount_type,
    d.discount_amount,
    d.storewide,
    d.first_time_customer,
    s.name        AS store_name,
    s.city,
    s.state,
    d.claimed_count
FROM deals d
JOIN stores s USING (store_id),
     LATERAL FLATTEN(input => d.brands) b
WHERE d.is_archived = false
  AND EXISTS (
      SELECT 1 FROM brand_patterns bp WHERE b.value::text ILIKE bp.pattern
  )
  AND s.state = 'CA'    -- ⬅ swap state (omit clause for all states)
ORDER BY d.claimed_count DESC NULLS LAST;
```

---

## Retailer view: active deals by category within 10 miles

```sql
SELECT
    d.title,
    d.discount_type,
    d.discount_amount,
    s.name                         AS competitor_store,
    s.city,
    ROUND(ST_DISTANCE(s.location, ST_POINT(-122.4194, 37.7749)) / 1609.34, 2) AS miles_away,
    pc.value::text                 AS product_category
FROM deals d
JOIN stores s USING (store_id),
     LATERAL FLATTEN(input => d.product_categories) pc
WHERE d.is_archived = false
  AND ST_DWITHIN(s.location, ST_POINT(-122.4194, 37.7749), 16093.4)   -- 10 miles
  -- ⬅ swap longitude, latitude, radius
  AND pc.value::text IN ('Flower', 'Edibles', 'Vapes')                -- ⬅ trim (deals use scraped category names — see schema notes)
ORDER BY miles_away, pc.value::text;
```

---

## Most-claimed deals in a market

What's resonating? Highest engagement deals in a state, regardless of brand.

```sql
SELECT
    d.deal_id,
    d.title,
    d.discount_type,
    d.discount_amount,
    d.storewide,
    s.name        AS store_name,
    s.city,
    d.claimed_count
FROM deals d
JOIN stores s USING (store_id)
WHERE d.is_archived = false
  AND s.state = 'CA'      -- ⬅ swap state
  AND d.claimed_count IS NOT NULL
ORDER BY d.claimed_count DESC
LIMIT 50;
```

---

## Deal mix per store: storewide vs. brand vs. category

How does each store structure their promotions?

```sql
SELECT
    s.name AS store_name,
    s.state,
    SUM(CASE WHEN d.storewide THEN 1 ELSE 0 END)                          AS storewide_deals,
    SUM(CASE WHEN ARRAY_SIZE(d.brands) > 0 THEN 1 ELSE 0 END)             AS brand_specific_deals,
    SUM(CASE WHEN ARRAY_SIZE(d.product_categories) > 0 THEN 1 ELSE 0 END) AS category_specific_deals,
    SUM(CASE WHEN d.first_time_customer THEN 1 ELSE 0 END)                AS ftc_deals,
    COUNT(*)                                                              AS total_active_deals
FROM deals d
JOIN stores s USING (store_id)
WHERE d.is_archived = false
  AND s.state = 'CA'    -- ⬅ swap state
GROUP BY 1, 2
HAVING total_active_deals >= 3
ORDER BY total_active_deals DESC
LIMIT 50;
```

---

## First-time-customer offers in a market

Useful for studying acquisition-incentive intensity in a market.

```sql
SELECT
    s.name AS store_name,
    s.city,
    s.state,
    d.title,
    d.description,
    d.discount_type,
    d.discount_amount,
    d.restrictions
FROM deals d
JOIN stores s USING (store_id)
WHERE d.is_archived = false
  AND d.first_time_customer = true
  AND s.state = 'CA'    -- ⬅ swap state
ORDER BY s.city, s.name;
```

---

## Deal volume snapshot

The `deals` view has no creation timestamp, so we can't see "new deals" directly — but we can count currently active deals and recently archived ones to gauge churn.

```sql
SELECT
    SUM(CASE WHEN is_archived = false                                     THEN 1 ELSE 0 END) AS currently_active,
    SUM(CASE WHEN is_archived = true
              AND archived_at >= DATEADD('day', -7, CURRENT_DATE)          THEN 1 ELSE 0 END) AS archived_last_7d
FROM deals;
```

Same idea, per store, for a single market:

```sql
SELECT
    s.name,
    s.state,
    SUM(CASE WHEN d.is_archived = false                                     THEN 1 ELSE 0 END) AS active_now,
    SUM(CASE WHEN d.is_archived = true
              AND d.archived_at >= DATEADD('day', -7, CURRENT_DATE)          THEN 1 ELSE 0 END) AS archived_last_7d
FROM deals d
JOIN stores s USING (store_id)
WHERE s.state = 'CA'    -- ⬅ swap state
GROUP BY 1, 2
ORDER BY active_now DESC
LIMIT 50;
```
