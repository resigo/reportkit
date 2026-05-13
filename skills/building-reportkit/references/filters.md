# Filters Reference

Filters are interactive controls that expand to **values** in SQL. You control WHERE clauses explicitly using `@filter_id` references and `[[...]]` optional clauses.

## Filter types (uiType)

| Type | Description | Default value | SQL expansion |
|------|-------------|---------------|---------------|
| `dropdown` | Single-select dropdown | `"__all__"` | `'value'` |
| `multiselect` | Multi-select checkboxes | `"__all__"` | `'val1', 'val2'` (for `IN`) |
| `daterange` | Date range picker | `"__all__"` | `@filter.start` / `@filter.end` |
| `range` | Numeric slider | `"__all__"` | `@filter.min` / `@filter.max` |
| `text` | Free text input | `"__all__"` | `'value'` |
| `toggle` | On/off toggle | `"__all__"` | `'true'` or `'false'` |

## Filter fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string (required) | `snake_case` identifier |
| `label` | string (required) | Display label |
| `uiType` | string (required) | UI control type (see above) |
| `default` | string (required) | `"__all__"` for no filter |
| `model` | string | Model id to populate options from |
| `column` | string | Column to SELECT DISTINCT from (populates dropdown/multiselect options) |
| `source` | string | Raw SQL returning distinct values (single column) |
| `connection` | string | Connection id for raw SQL source queries |
| `values` | string[] | Static option list |
| `excludeNull` | boolean | Exclude null values from options (default: `false`) |
| `min` / `max` / `step` | number | For range filters only |

## Examples

**Dropdown (model-backed):**
```json
{
  "id": "country",
  "label": "Country",
  "uiType": "dropdown",
  "model": "orders",
  "column": "country",
  "default": "__all__"
}
```

**Date range:**
```json
{
  "id": "date_range",
  "label": "Date Range",
  "uiType": "daterange",
  "default": "__all__"
}
```

**Range slider:**
```json
{
  "id": "revenue",
  "label": "Revenue",
  "uiType": "range",
  "min": 0,
  "max": 50000,
  "step": 500,
  "default": "__all__"
}
```

**Multiselect:**
```json
{
  "id": "status",
  "label": "Status",
  "uiType": "multiselect",
  "values": ["active", "pending", "cancelled"],
  "default": "__all__"
}
```

## Using filters in SQL

Place `@filter_id` directly in your SQL WHERE clauses. Wrap conditions in `[[...]]` so the clause is removed when the filter is `__all__`.

```json
{
  "id": "sales_data",
  "type": "model",
  "model": "orders",
  "sql": "SELECT * FROM {{table}} WHERE true [[AND country = @country]] [[AND created_at >= @date_range.start AND created_at <= @date_range.end]]"
}
```

**Dot notation for compound filters:**
- Daterange: `@filter.start` and `@filter.end`
- Range: `@filter.min` and `@filter.max`

**Key advantage**: The same `@country` filter can filter different columns in different blocks – one block uses `country = @country`, another uses `event_country = @country`.

## Using filters with metric blocks

Metric blocks (using `metrics`/`dimensions`, no raw SQL) use `filterColumns` to map filter IDs to column names on the block's table:

```json
{
  "id": "revenue_kpi",
  "type": "model",
  "model": "orders",
  "metrics": ["total_revenue"],
  "filterColumns": { "country": "country", "date_range": "created_at" }
}
```

Each key is a filter ID, each value is the column to filter on. The same filter can map to different columns in different blocks.

## Multiselect in SQL

Use `IN (@filter)` – the values expand to comma-separated quoted strings:

```sql
WHERE true [[AND status IN (@status)]]
```

When `status` = `"active,pending"`, this becomes:
```sql
WHERE true AND status IN ('active', 'pending')
```

## Model-backed filters need a connection

When using `model`/`column`, the model must have a `connection` field so the options query runs on the correct warehouse.

## Panel-local filters

Filters can be scoped to a single panel instead of the whole report. Define them on a layout item in `filters[]`:

```json
{
  "id": "revenue_by_customer",
  "type": "HorizontalBarChart",
  "data": "revenue_by_customer",
  "row": 1, "col": 1, "span": 12,
  "filters": [
    {
      "id": "local_min_rev",
      "label": "Min Revenue",
      "uiType": "range",
      "min": 0,
      "max": 50000,
      "step": 500,
      "default": "__all__"
    }
  ]
}
```

**UI behaviour:** The filter chips appear as a small overlay in the top-right corner of the panel when hovered – they are not shown in the global FilterBar. Changing a panel-local filter rematerializes only that panel's data block (not all blocks).

**SQL usage:** Exactly the same as global filters – use `@filter_id` in SQL or `filterColumns` in metric blocks. The data block has no idea whether a filter is global or local.

**When to use local vs global:**
- Use **global** filters when the same filter should apply across multiple panels (e.g. `@country` filtering all panels).
- Use **local** filters when a filter is specific to one panel and would clutter the top bar (e.g. a revenue range slider on a single chart).

**ID uniqueness:** Filter IDs must be unique across the report – a local filter ID must not clash with any global filter ID.
