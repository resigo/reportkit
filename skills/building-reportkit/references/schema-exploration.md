# Schema Exploration Reference

When you need to find the correct schema, dataset, or table name – **never guess**. Use the search endpoint.

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

## List all sources

```
GET /api/data/sources
```

Returns both registered models and tables discovered from connected warehouses.

## When to use

- Find which dataset/schema a table lives in before writing SQL or creating models
- Discover available columns and their types
- Verify table names when a query fails with "not found"
