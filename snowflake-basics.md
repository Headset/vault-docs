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

## Example: Connecting in Python
Using the official `snowflake-connector-python` package:
```python
import snowflake.connector

conn = snowflake.connector.connect(
    user='<user>',
    password='<password>',
    account='<account>',
    warehouse='default_wh',
    database='headset_insights',
    schema='insights_share'
)
cur = conn.cursor()
cur.execute('SELECT CURRENT_VERSION()')
print(cur.fetchone())
cur.close()
conn.close()
```

## Example: Connecting in JavaScript (Node.js)
Using the `snowflake-sdk` package:
```js
const snowflake = require('snowflake-sdk');

const connection = snowflake.createConnection({
  account: '<account>',
  username: '<user>',
  password: '<password>',
  warehouse: 'default_wh',
  database: 'headset_retailer',
  schema: 'retailer_share'
});

connection.connect((err, conn) => {
  if (err) {
    console.error('Unable to connect:', err.message);
  } else {
    console.log('Successfully connected!');
    // Run a query...
    connection.destroy();
  }
});
```

## Example: Connecting in C#
Using the [Snowflake.Data](https://www.nuget.org/packages/Snowflake.Data/) ADO.NET provider:
```csharp
using Snowflake.Data.Client;

string connectionString = "account=<account>;user=<user>;password=<password>;warehouse=default_wh;db=headset_bridge;schema=bridge_share";
using (var conn = new SnowflakeDbConnection())
{
    conn.ConnectionString = connectionString;
    conn.Open();
    using (var cmd = conn.CreateCommand())
    {
        cmd.CommandText = "SELECT CURRENT_VERSION()";
        using (var reader = cmd.ExecuteReader())
        {
            while (reader.Read())
            {
                Console.WriteLine(reader.GetString(0));
            }
        }
    }
}
```

## Example: Connecting in Rust
Using the [odbc-api](https://crates.io/crates/odbc-api) crate with an ODBC DSN configured for Snowflake:
```rust
use odbc_api::{Environment, ConnectionOptions};

let env = Environment::new()?;
let mut conn = env.connect("SnowflakeDSN", "<user>", "<password>")?;
let cursor = conn.execute("SELECT CURRENT_VERSION()", ())?.unwrap();
if let Some(mut cursor) = cursor {
    while let Some(row) = cursor.fetch()? {
        let version: &str = row.get(0)?;
        println!("Snowflake version: {}", version);
    }
}
```

## Example: Connecting in Go
Using the [gosnowflake](https://github.com/snowflakedb/gosnowflake) driver with the `database/sql` package:
```go
package main

import (
    "database/sql"
    "fmt"
    _ "github.com/snowflakedb/gosnowflake"
)

func main() {
    dsn := "<user>:<password>@<account>/<database>/<schema>?warehouse=default_wh"
    db, err := sql.Open("snowflake", dsn)
    if err != nil {
        panic(err)
    }
    defer db.Close()

    row := db.QueryRow("SELECT CURRENT_VERSION()")
    var version string
    if err := row.Scan(&version); err != nil {
        panic(err)
    }
    fmt.Println("Snowflake version:", version)
}
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
