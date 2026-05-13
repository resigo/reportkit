# Schema Exploration & Data Query Reference

When you need to find the correct schema, dataset, or table name – **never guess**. Use the search endpoint. When you need to validate SQL or preview data before writing a report – use the query endpoint.

## Search tables

```
GET /api/data/search-tables?q=<table_name_pattern>
GET /api/data/search-tables?q=<table_name_pattern>&connection=<connection_id>
```

Response (grouped by connection):
```json
{
  "dwh-prod": [
    {
      "name": "public.sales",
      "type": "table",
      "columns": [
        { "name": "date", "type": "DATE", "nullable": false },
        { "name": "region", "type": "STRING", "nullable": false },
        { "name": "product", "type": "STRING", "nullable": false },
        { "name": "units", "type": "INTEGER", "nullable": false },
        { "name": "revenue", "type": "FLOAT", "nullable": false }
      ]
    }
  ]
}
```

The `q` parameter matches against table names (case-insensitive, min 2 characters). Add `connection` to limit to a specific warehouse. This queries only metadata (INFORMATION_SCHEMA) – zero cost.

## Run a query

Use this to validate SQL, explore data, or check column values before writing a data block.

```
POST /api/data/query
Content-Type: application/json

{ "sql": "SELECT ...", "connection": "<connection_id>" }
```

- `connection` is optional – omit to query the seed SQLite DB (CSV seeds only)
- Returns `{ rows: [...], rowCount: N }` on success
- Returns `{ error: "..." }` with status 500 on failure

Example – check distinct values before writing a filter:
```json
{ "sql": "SELECT DISTINCT region FROM public.sales ORDER BY region LIMIT 50", "connection": "dwh-prod" }
```

Example – validate aggregation logic before writing a data block:
```json
{ "sql": "SELECT date_trunc('month', created_at) AS month, SUM(revenue) AS total FROM orders GROUP BY 1 ORDER BY 1 LIMIT 5", "connection": "dwh-prod" }
```

## List all sources

```
GET /api/data/sources
```

Returns both registered models and tables discovered from connected warehouses.

## When to use

- **Search tables** – find which dataset/schema a table lives in before writing SQL or creating models; discover available columns and their types; verify table names when a query fails with "not found"
- **Run a query** – validate SQL before putting it in a data block; check that column names, types, and aggregations are correct; preview sample data to understand the shape of results
