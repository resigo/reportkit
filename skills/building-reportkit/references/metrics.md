# Metrics Reference

Metrics are named, reusable SQL aggregations defined on a model. Always prefer referencing approved metrics over writing raw aggregate SQL.

## Reading existing metrics

Model files (`models/*.rk-model.json`) may contain `metrics` and `dimensions` arrays. Read these before writing data block SQL.

- Only metrics with `"status": "approved"` can be used in reports
- Metrics with `"status": "proposed"` are pending human review – **do not reference them**

## Referencing metrics in data blocks

Use `metrics` and `dimensions` arrays instead of raw SQL:

```json
{
  "id": "revenue_kpi",
  "type": "model",
  "model": "stg_orders",
  "metrics": ["total_revenue"],
  "variables": { "$REGION": "region" }
}
```

```json
{
  "id": "revenue_by_status",
  "type": "model",
  "model": "stg_orders",
  "metrics": ["total_revenue", "order_count"],
  "dimensions": ["status"]
}
```

`metrics` and `sql` are mutually exclusive – never set both.

## Proposing new metrics

When no approved metrics exist, you may propose them:

1. Read the model file – check existing `metrics[]` to avoid ID conflicts
2. Inspect `columns[]` to verify SQL expressions are valid
3. Validate via: `GET /api/models/:id/metrics/:metricId/preview-sql?dimensions=dim1,dim2`
4. Write **3-8 proposed metrics in a single file write** – never one at a time
5. Set `"status": "proposed"` – the human reviews and approves in the UI

## Metric types

**`aggregate`** – single SQL expression:
```json
{ "id": "total_revenue", "type": "aggregate", "expression": "SUM(amount)", "format": "currency", "status": "proposed" }
```

**`ratio`** – numerator / denominator (server handles division-by-zero):
```json
{ "id": "conversion_rate", "type": "ratio", "numerator": "SUM(converted)", "denominator": "SUM(visits)", "format": "percent", "status": "proposed" }
```

**`derived`** – references sibling approved metrics via `{{metric_id}}` placeholders:
```json
{ "id": "revenue_per_user", "type": "derived", "expression": "{{total_revenue}} / {{total_users}}", "status": "proposed" }
```

## Metric fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique metric identifier |
| `label` | string | Human-readable name |
| `description` | string | What this metric measures |
| `type` | `"aggregate"` / `"ratio"` / `"derived"` | Metric type |
| `expression` | string | SQL aggregate expression (aggregate/derived) |
| `numerator` | string | SQL aggregate (ratio only) |
| `denominator` | string | SQL aggregate (ratio only) |
| `format` | `"number"` / `"currency"` / `"percent"` | Display format |
| `unit` | string | Unit label (e.g. "$") |
| `status` | `"proposed"` / `"approved"` | Review status |
| `dimensionIds` | string[] | Restrict to subset of model dimensions |

## Dimensions

Dimensions define GROUP BY breakdowns for metrics:

```json
{
  "dimensions": [
    { "id": "region", "label": "Region", "column": "region" },
    { "id": "month", "label": "Month", "column": "created_at", "truncate": "month" }
  ]
}
```

`truncate` options: `day`, `week`, `month`, `quarter`, `year`.
