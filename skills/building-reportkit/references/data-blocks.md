# Data Blocks Reference

Data blocks define how panels get their data. Each block runs a query and is referenced by panels via its `id`.

## Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string (required) | Unique block identifier, referenced by panels via `data` |
| `model` | string | Model id from `models/`. A block with `model` set is a model block; a block with only `sql` is a raw-SQL block. |
| `sql` | string | SQL query – use `{{table}}` for the model's table name |
| `connection` | string | Connection id for raw SQL blocks targeting a specific warehouse |
| `select` | string[] | Declarative column list (builds `SELECT col1, col2 FROM {{table}}`) |
| `metrics` | string[] | Metric ids from model.metrics[] (approved only). Mutually exclusive with `sql`. |
| `dimensions` | string[] | Dimension ids from model.dimensions[] for GROUP BY |
| `filterColumns` | Record<string, string> | Maps filter IDs to column names on this block's table (keys = filter IDs, values = columns) |
| `aggregate` | Record<string, string> | Post-filter aggregation – collapses rows into a single value |

## Writing data blocks

- Set `model` to a model id from `models/`
- **Prefer metrics over raw SQL** when the model has approved metrics – use `metrics: ["metric_id"]` with optional `dimensions: ["dim_id"]`
- Use `{{table}}` in SQL – replaced with the model's table name
- For cross-source joins: `{{sources.<other_model_id>}}` references other tables
- For raw SQL without a model: omit `model` and set `connection`
- `metrics` and `sql` are mutually exclusive – never set both

## Filters in SQL blocks

Place `@filter_id` references directly in your SQL WHERE clauses. Wrap in `[[...]]` so the clause is removed when the filter is `__all__`. The filter expands to a **value** – you control the column and operator.

```sql
SELECT * FROM {{table}} WHERE true [[AND country = @country]] [[AND created_at >= @date_range.start]]
```

Dot notation for compound types:
- Daterange: `@filter.start`, `@filter.end`
- Range: `@filter.min`, `@filter.max`

## Filters in metric blocks

Metric blocks don't have raw SQL. Use `filterColumns` to map filter IDs to column names on the block's table:

```json
{
  "id": "revenue_kpi",
  "model": "orders",
  "metrics": ["total_revenue"],
  "filterColumns": { "country": "country", "date_range": "created_at" }
}
```

## Aggregate

Collapses rows into a single result. Use for KPICards when no approved metric exists.

```json
{ "aggregate": { "revenue": "sum" } }
```

Built-in functions: `count`, `sum`, `avg`, `min`, `max`. Or use raw SQL expressions:
```json
{ "aggregate": { "value": "ROUND(100.0 * SUM(returned) / NULLIF(SUM(total), 0), 1)" } }
```

The key must match a column name in your SQL output. With metrics blocks, no `aggregate` is needed.

## Template placeholders

Only `{{table}}` and `{{sources.<id>}}` are supported. Do NOT use dbt Jinja (`{{ ref() }}`, `{% %}`), `{variable}`, or any other template syntax.

## SQL dialect

Must match the target warehouse: Postgres SQL for postgres, BigQuery SQL for bigquery, SQLite for seeds, etc.

## Registering a data source (model)

Create a `.rk-model.json` file in `models/`. The `id` must match the filename.

```json
{
  "id": "my_source",
  "label": "Human-readable name",
  "table": "schema.table_name",
  "connection": "connection_id",
  "description": "What this source contains",
  "columns": [
    { "name": "col1", "type": "date", "description": "Description of col1" },
    { "name": "col2", "type": "number", "description": "Description of col2" }
  ]
}
```

Every column entry must be an object with a `name` and a `description`. `description` is required – set it to `""` if there is no useful context. `type` is optional but recommended: the column's data type (e.g. `date`, `string`, `number`, `boolean`). Bare-string entries like `"col3"` are not allowed.

When a data block sets a `model` field, it must reference a registered model. If no model exists for the data you need, create one first.

## Examples

**Metrics block with filter binding (preferred for aggregations):**
```json
{
  "id": "revenue_kpi",
  "model": "orders",
  "metrics": ["total_revenue"],
  "filterColumns": { "region": "region" }
}
```

**Metrics with dimensions:**
```json
{
  "id": "revenue_by_status",
  "model": "orders",
  "metrics": ["total_revenue", "order_count"],
  "dimensions": ["status"]
}
```

**Raw SQL block with filter references:**
```json
{
  "id": "monthly_trend",
  "model": "orders",
  "sql": "SELECT date_trunc('month', created_at) as month, SUM(amount) as revenue FROM {{table}} WHERE true [[AND region = @region]] GROUP BY month ORDER BY month"
}
```

**KPI with aggregate:**
```json
{
  "id": "total_orders",
  "model": "orders",
  "sql": "SELECT COUNT(*) as count FROM {{table}} WHERE true [[AND region = @region]]",
  "aggregate": { "count": "count" }
}
```
