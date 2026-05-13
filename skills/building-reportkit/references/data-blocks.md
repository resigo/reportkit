# Data Blocks Reference

Data blocks define how panels get their data. Each block runs a query and is referenced by panels via its `id`.

## Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string (required) | Unique block identifier, referenced by panels via `data` |
| `type` | `"sql"` or `"model"` (required) | Block type |
| `model` | string | Model id from `models/` (required when type is "model") |
| `sql` | string | SQL query – use `{{table}}` for the model's table name |
| `connection` | string | Connection id for raw SQL blocks targeting a specific warehouse |
| `select` | string[] | Declarative column list (builds `SELECT col1, col2 FROM {{table}}`) |
| `metrics` | string[] | Metric ids from model.metrics[] (approved only). Mutually exclusive with `sql`. |
| `dimensions` | string[] | Dimension ids from model.dimensions[] for GROUP BY |
| `variables` | Record<string, string> | Variable-to-column mapping for metric blocks (see below) |
| `aggregate` | Record<string, string> | Post-filter aggregation – collapses rows into a single value |

## Writing data blocks

- Set `type: "model"` and `model` to a model id from `models/`
- **Prefer metrics over raw SQL** when the model has approved metrics – use `metrics: ["metric_id"]` with optional `dimensions: ["dim_id"]`
- Use `{{table}}` in SQL – replaced with the model's table name
- For cross-source joins: `{{sources.<other_model_id>}}` references other tables
- For raw SQL without a model: use `type: "sql"` with `connection`
- `metrics` and `sql` are mutually exclusive – never set both

## Variables in SQL blocks

Place `$VAR` references directly in your SQL WHERE clauses. Wrap in `[[...]]` so the clause is removed when the variable is `__all__`. The variable expands to a **value** – you control the column and operator.

```sql
SELECT * FROM {{table}} WHERE true [[AND country = $COUNTRY]] [[AND created_at >= $DATE_RANGE_START]]
```

## Variables in metric blocks

Metric blocks don't have raw SQL. Use a `variables` map to specify how each variable applies:

```json
{
  "id": "revenue_kpi",
  "type": "model",
  "model": "orders",
  "metrics": ["total_revenue"],
  "variables": { "$COUNTRY": "country", "$DATE_RANGE": "created_at" }
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
  "columns": ["col1", "col2", "col3"]
}
```

Every data block with `type: "model"` must reference a registered model. If no model exists for the data you need, create one first.

## Examples

**Metrics block with variable filtering (preferred for aggregations):**
```json
{
  "id": "revenue_kpi",
  "type": "model",
  "model": "orders",
  "metrics": ["total_revenue"],
  "variables": { "$REGION": "region" }
}
```

**Metrics with dimensions:**
```json
{
  "id": "revenue_by_status",
  "type": "model",
  "model": "orders",
  "metrics": ["total_revenue", "order_count"],
  "dimensions": ["status"]
}
```

**Raw SQL block with variable filtering:**
```json
{
  "id": "monthly_trend",
  "type": "model",
  "model": "orders",
  "sql": "SELECT date_trunc('month', created_at) as month, SUM(amount) as revenue FROM {{table}} WHERE true [[AND region = $REGION]] GROUP BY month ORDER BY month"
}
```

**KPI with aggregate:**
```json
{
  "id": "total_orders",
  "type": "model",
  "model": "orders",
  "sql": "SELECT COUNT(*) as count FROM {{table}} WHERE true [[AND region = $REGION]]",
  "aggregate": { "count": "count" }
}
```
