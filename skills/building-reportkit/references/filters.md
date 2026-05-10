# Filters Reference

Filters are interactive controls that narrow data across panels. The server auto-generates WHERE clauses – never write filter logic in SQL.

## Filter types

| Type | Description | Default value |
|------|-------------|---------------|
| `select` (default) | Dropdown of distinct values | `"__all__"` |
| `daterange` | Date range picker, generates `BETWEEN` | `"__all__"` or `"start,end"` |
| `range` | Numeric slider | `"min,max"` |

## Filter fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string (required) | Unique filter identifier |
| `column` | string (required) | Column name used in the generated WHERE clause |
| `default` | string (required) | `"__all__"` for no filter; range: `"min,max"`; daterange: `"start,end"` |
| `label` | string | Display label |
| `type` | `"select"` / `"daterange"` / `"range"` | Filter type (default: select) |
| `source` | string | Raw SQL returning distinct values (single column) |
| `model` | string | Model id to populate options from |
| `modelColumn` | string | Column to SELECT DISTINCT from (required when `model` is set) |
| `connection` | string | Connection id for raw SQL source queries |
| `excludeNull` | boolean | Exclude null values from select options (default: `false`) |
| `min` / `max` / `step` | number | For range filters only |

## Examples

**Select filter (model-backed):**
```json
{
  "id": "country",
  "label": "Country",
  "column": "country",
  "model": "orders",
  "modelColumn": "country",
  "default": "__all__"
}
```

**Date range filter:**
```json
{
  "id": "date_range",
  "label": "Date Range",
  "type": "daterange",
  "column": "created_at",
  "default": "__all__"
}
```

**Range slider:**
```json
{
  "id": "min_amount",
  "label": "Min Amount",
  "type": "range",
  "column": "amount",
  "min": 0,
  "max": 10000,
  "step": 100,
  "default": "0,10000"
}
```

## Wiring filters to data blocks

Add filter ids to a data block's `filterBy` array. Only listed filters apply to that block.

```json
{
  "id": "sales_data",
  "type": "model",
  "model": "orders",
  "sql": "SELECT * FROM {{table}}",
  "filterBy": ["country", "date_range"]
}
```

Every filter id should appear in at least one data block's `filterBy`.

## Important: model-backed filters need a connection

When using `model`/`modelColumn`, the model must have a `connection` field so the filter query runs on the correct warehouse. Without it, the query falls back to the seed SQLite DB and will likely return no results.

If you need a custom query for filter options (e.g. from a different table), use `source` with a `connection`:

```json
{
  "id": "region",
  "label": "Region",
  "column": "region",
  "source": "SELECT DISTINCT region FROM dim_regions ORDER BY region",
  "connection": "my_warehouse",
  "default": "__all__"
}
```
