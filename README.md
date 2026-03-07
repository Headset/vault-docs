<p align="center">
  <a href="https://www.headset.io">
    <img src="https://cdn.prod.website-files.com/640fe83bf3f9dfb92669d456/640fed396e869ffe886cf87b_Real-time%20Cannabis%20Data%2C%20Analytics%20%26%20Intelligence%20_%20Headset-1%20(1).svg" alt="Headset" width="280" />
  </a>
</p>

<h1 align="center">Vault</h1>

<p align="center">
  <strong>Your data. Your warehouse. No middleman.</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Powered%20by-Snowflake-29B5E8?style=for-the-badge&logo=snowflake&logoColor=white" alt="Powered by Snowflake" />
  <img src="https://img.shields.io/badge/SQL-Ready-4B8BBE?style=for-the-badge&logo=postgresql&logoColor=white" alt="SQL Ready" />
  <img src="https://img.shields.io/badge/Secure%20Data%20Share-Enabled-10B981?style=for-the-badge&logo=snowflake&logoColor=white" alt="Secure Data Share" />
</p>

---

Vault delivers the same cannabis market data that powers Headset's analytics — directly into your Snowflake account via [Secure Data Sharing](https://docs.snowflake.com/en/user-guide/data-sharing-intro). No ETL, no pipelines, no copies. The data stays in Snowflake and updates automatically.

## Data Products

Vault is organized into three products, each tailored to a different use case:

<table>
<tr>
<td width="33%" valign="top">

### [Retailer](./retailer)
**Your stores, your data**

Transaction-level sales, customer profiles, product catalog, and inventory for retailers who want full access to their own operational data.

6 views &middot; [Schema](./retailer/schema.md) &middot; [Queries](./retailer/sample-queries.md)

</td>
<td width="33%" valign="top">

### [Bridge](./bridge)
**Brand sell-through at retail**

See how your products move through retail partners — sales, inventory, batches, and METRC package tracking for brands and distributors.

6 views &middot; [Schema](./bridge/schema.md) &middot; [Queries](./bridge/sample-queries.md)

</td>
<td width="33%" valign="top">

### [Insights](./insights)
**The whole market**

Aggregated, anonymized market data — category trends, brand benchmarking, pricing intelligence, and package size analysis across all tracked markets.

4 views &middot; [Schema](./insights/schema.md) &middot; [Queries](./insights/sample-queries.md)

</td>
</tr>
</table>

## Getting Started

**1. Accept the share** — Your Headset onboarding team will share a database to your Snowflake account. Accept it and give it whatever name you like. Don't have a Snowflake account? No problem — we can provision and manage one for you.

**2. Connect** — Use any Snowflake-compatible tool: SQL worksheets, Python, BI platforms, dbt, or anything else that speaks SQL. See [Snowflake Basics](./snowflake-basics.md) for connection examples in Python, Node.js, C#, Go, and Rust.

**3. Query** — All data is in secure views, ready to `SELECT` immediately. Check the sample queries in each product section for a head start.

## Works With Your Stack

Vault data lives in Snowflake, so it works with any tool that has a Snowflake connector — no extra setup required.

<p align="center">
  <img src="https://img.shields.io/badge/Tableau-E97627?style=flat-square&logo=tableau&logoColor=white" alt="Tableau" />
  <img src="https://img.shields.io/badge/Looker-4285F4?style=flat-square&logo=looker&logoColor=white" alt="Looker" />
  <img src="https://img.shields.io/badge/Power%20BI-F2C811?style=flat-square&logo=powerbi&logoColor=black" alt="Power BI" />
  <img src="https://img.shields.io/badge/dbt-FF694B?style=flat-square&logo=dbt&logoColor=white" alt="dbt" />
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white" alt="Python" />
  <img src="https://img.shields.io/badge/Sigma-000000?style=flat-square&logo=sigma&logoColor=white" alt="Sigma" />
  <img src="https://img.shields.io/badge/Hex-F97316?style=flat-square&logo=hex&logoColor=white" alt="Hex" />
  <img src="https://img.shields.io/badge/Mode-5856D6?style=flat-square&logo=mode&logoColor=white" alt="Mode" />
  <img src="https://img.shields.io/badge/Metabase-509EE3?style=flat-square&logo=metabase&logoColor=white" alt="Metabase" />
  <img src="https://img.shields.io/badge/Excel-217346?style=flat-square&logo=microsoftexcel&logoColor=white" alt="Excel" />
</p>

## Quick Example

```sql
-- Top 10 products by revenue this month (Retailer)
SELECT
    PRODUCT_NAME,
    BRAND,
    CATEGORY,
    SUM(REVENUE) AS total_revenue,
    SUM(QUANTITY) AS total_units
FROM SALES
WHERE SOLD_DATE >= DATE_TRUNC('MONTH', CURRENT_DATE)
GROUP BY PRODUCT_NAME, BRAND, CATEGORY
ORDER BY total_revenue DESC
LIMIT 10;
```

## Resources

| Resource | Description |
|----------|-------------|
| [Snowflake Basics](./snowflake-basics.md) | Connection setup, authentication, and language-specific examples |
| [Glossary](./glossary.md) | Definitions of key terms used across Vault |
| [Headset Support](https://help.headset.io) | Help center for Headset products |

---

<p align="center">
  <sub>Built with care by the <a href="https://www.headset.io">Headset</a> team</sub>
</p>
