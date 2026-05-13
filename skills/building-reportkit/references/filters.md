# Variables Reference

Variables are interactive filter controls that expand to **values** in SQL. You control WHERE clauses explicitly using `$VAR` references and `[[...]]` optional clauses.

## Variable types (uiType)

| Type | Description | Default value | SQL expansion |
|------|-------------|---------------|---------------|
| `dropdown` | Single-select dropdown | `"__all__"` | `'value'` |
| `multiselect` | Multi-select checkboxes | `"__all__"` | `'val1', 'val2'` (for `IN`) |
| `daterange` | Date range picker | `"__all__"` | `$VAR_START` / `$VAR_END` |
| `range` | Numeric slider | `"__all__"` | `$VAR_MIN` / `$VAR_MAX` |
| `text` | Free text input | `"__all__"` | `'value'` |
| `toggle` | On/off toggle | `"__all__"` | `'true'` or `'false'` |

## Variable fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string (required) | `$UPPER_SNAKE_CASE` name |
| `label` | string (required) | Display label |
| `uiType` | string (required) | UI control type (see above) |
| `default` | string (required) | `"__all__"` for no filter |
| `model` | string | Model id to populate options from |
| `modelColumn` | string | Column to SELECT DISTINCT from (required when `model` is set) |
| `source` | string | Raw SQL returning distinct values (single column) |
| `connection` | string | Connection id for raw SQL source queries |
| `values` | string[] | Static option list |
| `excludeNull` | boolean | Exclude null values from options (default: `false`) |
| `min` / `max` / `step` | number | For range variables only |

## Examples

**Dropdown (model-backed):**
```json
{
  "name": "$COUNTRY",
  "label": "Country",
  "uiType": "dropdown",
  "model": "orders",
  "modelColumn": "country",
  "default": "__all__"
}
```

**Date range:**
```json
{
  "name": "$DATE_RANGE",
  "label": "Date Range",
  "uiType": "daterange",
  "default": "__all__"
}
```

**Range slider:**
```json
{
  "name": "$REVENUE",
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
  "name": "$STATUS",
  "label": "Status",
  "uiType": "multiselect",
  "values": ["active", "pending", "cancelled"],
  "default": "__all__"
}
```

## Using variables in SQL

Place `$VAR` directly in your SQL WHERE clauses. Wrap conditions in `[[...]]` so the clause is removed when the variable is `__all__`.

```json
{
  "id": "sales_data",
  "type": "model",
  "model": "orders",
  "sql": "SELECT * FROM {{table}} WHERE true [[AND country = $COUNTRY]] [[AND created_at >= $DATE_RANGE_START AND created_at <= $DATE_RANGE_END]]"
}
```

**Key advantage**: The same `$COUNTRY` variable can filter different columns in different blocks – one block uses `country = $COUNTRY`, another uses `event_country = $COUNTRY`.

## Using variables with metric blocks

Metric blocks (using `metrics`/`dimensions`, no raw SQL) use a `variables` map to specify which column each variable applies to:

```json
{
  "id": "revenue_kpi",
  "type": "model",
  "model": "orders",
  "metrics": ["total_revenue"],
  "variables": { "$COUNTRY": "country" }
}
```

## Multiselect in SQL

Use `IN ($VAR)` – the values expand to comma-separated quoted strings:

```sql
WHERE true [[AND status IN ($STATUS)]]
```

When `$STATUS` = `"active,pending"`, this becomes:
```sql
WHERE true AND status IN ('active', 'pending')
```

## Model-backed variables need a connection

When using `model`/`modelColumn`, the model must have a `connection` field so the options query runs on the correct warehouse.
