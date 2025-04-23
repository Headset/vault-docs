# Snowflake Basics for Vault Users

Welcome to Vault’s Snowflake-powered data platform! This guide will help you get connected, authenticate securely, and make the most of your data access.

## Connecting to Snowflake

Vault provides direct access to your data warehouse via Snowflake. You can connect using your favorite SQL client, BI tool, or programmatically (Python, R, etc).

**Connection details:**
- **Account URL:** _Provided in your onboarding email_
- **Warehouse:** _default_wh_
- **Database:** _varies by product_
- **Schema:** _varies by product_

### Example: Connecting with SnowSQL
```sh
snowsql -a <account> -u <user> -w default_wh -d vault
```

## Authentication Tips
- Use your Vault-provided credentials or SSO if enabled.
- Rotate passwords regularly.
- For programmatic access, use [key pair authentication](https://docs.snowflake.com/en/user-guide/key-pair-auth).

## General Usage Advice
- Always query with `LIMIT` while exploring.
- Mind your warehouse size for cost control.
- Use roles to manage data access.

For more, see the [Snowflake docs](https://docs.snowflake.com/).
