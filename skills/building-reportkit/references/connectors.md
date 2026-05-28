# Warehouse Connectors Reference

All warehouse drivers are optional – install only what you need.

## Available connectors

| Type | Package | Install | Status |
|------|---------|---------|--------|
| `duckdb` | `duckdb` | `npx @reportkit/cli install duckdb` | Supported |
| `bigquery` | `@google-cloud/bigquery` | `npx @reportkit/cli install bigquery` | Supported |
| `postgres` | `pg` | `npx @reportkit/cli install postgres` | Supported |
| `snowflake` | `snowflake-sdk` | `npx @reportkit/cli install snowflake` | Coming soon |

Drivers are lazy-loaded at runtime. If a driver is missing, the server prints a clear install instruction. CSV seed files in `seeds/` are loaded into a built-in SQLite engine (no extra install needed).

## Connection file examples

Connection files live in `connections/`.

**DuckDB** (`connections/dwh-prod.rk-conn.json`):
```json
{ "id": "dwh-prod", "type": "duckdb", "label": "DWH Prod",
  "path": "./dev.duckdb" }
```

**BigQuery** (`connections/dwh-prod.rk-conn.json`):
```json
{ "id": "dwh-prod", "type": "bigquery", "label": "DWH Prod",
  "project": "my-gcp-project", "dataset": "reporting",
  "keyFile": "./service-account.json" }
```
Also supports inline `credentials` object instead of `keyFile`.

**Postgres** (`connections/dwh-prod.rk-conn.json`):
```json
{ "id": "dwh-prod", "type": "postgres", "label": "DWH Prod",
  "connectionUrl": "postgres://user:pass@host:5432/mydb" }
```
Or with individual fields: `host`, `port`, `database`, `user`, `password`, `ssl`.

### Coming soon

**Snowflake** (`connections/dwh-prod.rk-conn.json`):
```json
{ "id": "dwh-prod", "type": "snowflake", "label": "DWH Prod",
  "account": "abc123.us-east-1", "user": "analyst", "password": "secret",
  "warehouse": "COMPUTE_WH", "database": "ANALYTICS", "schema": "PUBLIC" }
```

## Activating a new connection

When you create or modify a `.rk-conn.json` file, the server detects the change but does **not** automatically initialize the client. You must call the reconnect endpoint to activate it:

```
POST /api/connections/:id/reconnect
```

This initializes the client and re-materializes all reports that depend on it. Always call this after creating a new connection file.

## Linking models to connections

Models reference a connection via the `connection` field:
```json
{
  "id": "orders",
  "label": "Orders",
  "table": "public.orders",
  "connection": "dwh-prod",
  "columns": [
    { "name": "id", "type": "number", "description": "Order ID" },
    { "name": "amount", "type": "number", "description": "Order amount in USD" },
    { "name": "status", "type": "string", "description": "Order status" }
  ]
}
```

Raw-SQL data blocks (no `model` field) can also target a connection directly via `"connection": "dwh-prod"`.

## Query routing

1. Data block has `connection` field – uses that connection
2. Data block has `model` referencing a model with `connection` – uses that connection
3. No connection found – uses built-in SQLite seed database

SQL dialect is native to the target warehouse – write Postgres SQL for postgres, BigQuery SQL for bigquery, etc.
