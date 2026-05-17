# ReportKit

[![Status: Beta](https://img.shields.io/badge/status-beta-orange)](https://github.com/resigo/reportkit/issues)

> ReportKit is in early beta – feedback, bug reports, and ideas are very welcome. [Open an issue](https://github.com/resigo/reportkit/issues) to share yours.

ReportKit turns JSON files into interactive dashboards – built and maintained by AI agents, steered and reviewed by you. Connections, models, reports, and themes are all plain files in your repo: version-controlled, diff-friendly, and designed to be built by AI agents or by hand if necessary.

![ReportKit demo](demo.gif)

## Key Features

- **Agent-native** – Built-in skill files for Claude Code so agents can create and edit reports
- **File-based** – Reports, models, themes, and connections are plain JSON files in your repo
- **Multi-warehouse** – BigQuery, DuckDB, Postgres, or CSV seeds with zero setup (Snowflake coming soon)
- **Hot-reload** – Save a file, see changes instantly in the browser
- **Metrics layer** – Define reusable metrics and dimensions on your models
- **Filters** – Interactive select, date range, and numeric range filters with automatic WHERE clause generation
- **Theming** – Customize colors, chart styles, table appearance, and panel borders
- **Multi-page reports** – Split dashboards into tabbed pages

## Quick Start

```bash
npx @reportkit/cli init my-reports
cd my-reports && npx @reportkit/cli
```

Opens on **http://localhost:3001** with a demo sales dashboard using CSV seed data – no database required.

## Project Structure

After `npx @reportkit/cli init my-reports`, your project looks like this:

```
my-reports/
  connections/       # Warehouse connections (.rk-conn.json)
  models/            # Data models (.rk-model.json)
  reports/           # Report definitions (.rk.json)
  themes/            # Theme files (.rk-theme.json)
  seeds/             # (optional) CSV files loaded into SQLite on startup
```

## Connecting a Data Warehouse

Warehouse drivers are optional – install only what you need. If a driver is missing, the server prints a clear install instruction. Connection files live in `connections/` and look like this per connector type:

### BigQuery

```bash
npx @reportkit/cli install bigquery
```

**Local gcloud** – Application Default Credentials, mainly for local development. Run `gcloud auth application-default login` first (or rely on the attached service account when running on GCP):
```json
{
  "id": "dwh-prod",
  "type": "bigquery",
  "label": "DWH Prod",
  "project": "$BIGQUERY_PROJECT"
}
```

**Key file** – path to a service account JSON file:
```json
{
  "id": "dwh-prod",
  "type": "bigquery",
  "label": "DWH Prod",
  "project": "$BIGQUERY_PROJECT",
  "keyFile": "$GOOGLE_APPLICATION_CREDENTIALS"
}
```

**Inline JSON** – paste the service account JSON directly (for secret managers / CI):
```json
{
  "id": "dwh-prod",
  "type": "bigquery",
  "label": "DWH Prod",
  "project": "$BIGQUERY_PROJECT",
  "credentials": {
    "type": "service_account",
    "project_id": "my-gcp-project",
    "private_key_id": "...",
    "private_key": "...",
    "client_email": "...",
    "client_id": "...",
    "auth_uri": "...",
    "token_uri": "..."
  }
}
```

Optional fields: `dataset` (default dataset for table listing), `location` (e.g. `"US"`, `"EU"`).

### DuckDB

```bash
npx @reportkit/cli install duckdb
```

Local file:
```json
{
  "id": "local",
  "type": "duckdb",
  "label": "Local DuckDB",
  "path": "./data/analytics.duckdb"
}
```

MotherDuck (`md:` paths are passed directly – no local file check):
```json
{
  "id": "md",
  "type": "duckdb",
  "label": "MotherDuck",
  "path": "md:my_db?motherduck_token=$MOTHERDUCK_TOKEN"
}
```

### Postgres

```bash
npx @reportkit/cli install postgres
```

Connection URL (simplest):
```json
{
  "id": "pg-prod",
  "type": "postgres",
  "label": "Postgres Prod",
  "connectionUrl": "$DATABASE_URL"
}
```

Individual fields:
```json
{
  "id": "pg-prod",
  "type": "postgres",
  "label": "Postgres Prod",
  "host": "localhost",
  "port": 5432,
  "database": "mydb",
  "user": "postgres",
  "password": "$PGPASSWORD"
}
```

### Snowflake *(coming soon)*

Snowflake support is under development. Check back soon.

## Connection Resolution

1. Data block has a `connection` field – uses that connection
2. Data block references a model with a `connection` – uses that connection
3. No connection found – uses the built-in SQLite seed database

String values starting with `$` are resolved from environment variables – commit your connection files and inject secrets at runtime.

## CSV Seeds

Drop CSV files in `seeds/` for zero-setup data. They are loaded into a built-in SQLite engine on startup – no database required. The filename (without `.csv`) becomes the table name.

```
seeds/sales.csv    →  table name: sales
seeds/products.csv →  table name: products
```

Reference seed tables in your models and SQL the same way you would any warehouse table.

## Models

Models are registered data sources that describe a table, its columns, and optional metrics and dimensions.

`models/sales.rk-model.json`:
```json
{
  "id": "sales",
  "label": "Sales",
  "table": "sales",
  "description": "Product sales by region",
  "columns": ["date", "region", "product", "units", "revenue"],
  "connection": "dwh-prod",
  "dimensions": [
    { "id": "product", "label": "Product", "column": "product" },
    { "id": "region", "label": "Region", "column": "region" }
  ],
  "metrics": [
    {
      "id": "total_revenue",
      "label": "Total Revenue",
      "type": "aggregate",
      "expression": "SUM(revenue)",
      "format": "currency",
      "unit": "$",
      "status": "approved"
    },
    {
      "id": "total_units",
      "label": "Total Units",
      "type": "aggregate",
      "expression": "SUM(units)",
      "format": "number",
      "status": "approved"
    },
    {
      "id": "avg_price",
      "label": "Avg Price",
      "type": "ratio",
      "numerator": "SUM(revenue)",
      "denominator": "SUM(units)",
      "format": "currency",
      "unit": "$",
      "status": "approved"
    }
  ]
}
```

### Metric Types

| Type | Fields | Description |
|------|--------|-------------|
| `aggregate` | `expression` | Single SQL aggregate – e.g. `SUM(amount)` |
| `ratio` | `numerator`, `denominator` | Server handles division-by-zero automatically |
| `derived` | `expression` | References other metrics via `{{metric_id}}` placeholders |

Metrics with `"status": "approved"` can be referenced in data blocks. Set `"status": "proposed"` for metrics pending human review.

### Dimensions

Dimensions define GROUP BY columns with optional time truncation:

```json
{ "id": "order_month", "label": "Order Month", "column": "created_at", "truncate": "month" }
```

Truncation options: `day`, `week`, `month`, `quarter`, `year`.

## Reports

Reports are `.rk.json` files in `reports/`. The `id` field must match the filename.

`reports/sales_dashboard.rk.json`:
```json
{
  "version": 1,
  "id": "sales_dashboard",
  "title": "Sales Dashboard",
  "description": "Product sales overview across regions",
  "theme": "default",
  "filters": [
    {
      "id": "region",
      "label": "Region",
      "uiType": "dropdown",
      "model": "sales",
      "column": "region",
      "default": "__all__"
    }
  ],
  "data": [
    {
      "id": "total_revenue",
      "type": "model",
      "model": "sales",
      "metrics": ["total_revenue"],
      "filterColumns": { "region": "region" }
    },
    {
      "id": "revenue_by_month",
      "type": "model",
      "model": "sales",
      "sql": "SELECT substr(date, 1, 7) as month, SUM(revenue) as revenue FROM {{table}} WHERE true [[AND region = @region]] GROUP BY month ORDER BY month"
    }
  ],
  "layout": [
    {
      "id": "kpi_revenue",
      "type": "KPICard",
      "title": "Total Revenue",
      "data": "total_revenue",
      "value": "total_revenue",
      "unit": "$",
      "row": 1, "col": 1, "span": 8, "height": 4
    },
    {
      "id": "revenue_chart",
      "type": "BarChart",
      "title": "Revenue by Month",
      "data": "revenue_by_month",
      "x": "month",
      "y": "revenue",
      "row": 5, "col": 1, "span": 12, "height": 12
    }
  ]
}
```

### Data Blocks

Data blocks define queries that panels reference by `id`.

**Metrics block** (preferred for aggregations):
```json
{
  "id": "revenue_kpi",
  "type": "model",
  "model": "sales",
  "metrics": ["total_revenue", "total_units"],
  "dimensions": ["region"],
  "filterColumns": { "region": "region", "date_range": "date" }
}
```

**SQL block** (for custom queries, joins, CTEs):
```json
{
  "id": "monthly_trend",
  "type": "model",
  "model": "sales",
  "sql": "SELECT substr(date, 1, 7) as month, SUM(revenue) as revenue FROM {{table}} WHERE true [[AND region = @region]] GROUP BY month ORDER BY month"
}
```

- `{{table}}` is replaced with the model's table name at runtime
- Place `@filter_id` references in SQL WHERE clauses, wrap in `[[...]]` for optional filtering
- For metric blocks, use `filterColumns` to map filter IDs to column names
- `metrics` and `sql` are mutually exclusive
- `aggregate` collapses rows: `{ "revenue": "sum" }` – built-ins: `count`, `sum`, `avg`, `min`, `max`

### Multi-Page Reports

Use `pages` for tabbed dashboards:

```json
{
  "pages": [
    {
      "id": "overview",
      "title": "Overview",
      "layout": [ ... ]
    },
    {
      "id": "details",
      "title": "Details",
      "layout": [ ... ]
    }
  ],
  "layout": []
}
```

## Panel Types

| Type | Description | Key Props |
|------|-------------|-----------|
| `LineChart` | Time-series line chart with multi-series, colorBy, dual Y-axis | `x`, `y`, `colorBy`, `y2` |
| `BarChart` | Vertical bars – grouped or stacked multi-series | `x`, `y`, `stack` |
| `AreaChart` | Filled area chart – multi-series, stacking, colorBy | `x`, `y`, `colorBy`, `stack` |
| `HorizontalBarChart` | Horizontal ranked bars | `x` (value), `y` (category) |
| `ScatterChart` | Scatter with categorical color encoding | `x`, `y`, `colorBy` |
| `PieChart` | Pie/donut chart | `label`, `value` |
| `FunnelChart` | Funnel visualization | `label`, `value` |
| `CohortChart` | Heatmap grid for retention analysis | `x`, `y`, `value` |
| `CustomChart` | Agent-generated visx chart | `render` |
| `KPICard` | Single-number summary | `value`, `unit`, `subtitle` |
| `DataTable` | Sortable, paginated table | `columns`, `limit` |
| `Title` | Heading text (no data) | `title`, `level`, `align` |
| `Text` | Body text block (no data) | `content`, `align` |
| `Image` | Image panel (no data) | `src`, `alt` |

## Grid Layout

Panels are placed on a **24-column grid** using four properties:

| Property | Description |
|----------|-------------|
| `row` | Row position (1-indexed) |
| `col` | Column position (1-indexed, 1–24) |
| `span` | Width in columns (1–24) |
| `height` | Height in row units |

`col + span` must not exceed 25. Two panels side-by-side:

```json
{ "row": 1, "col": 1,  "span": 12, "height": 10 }
{ "row": 1, "col": 13, "span": 12, "height": 10 }
```

Full width: `col: 1, span: 24`.

## Filters

Filters add interactive controls to your report. They expand to **values** in SQL – you control the column and operator in each data block.

```json
"filters": [
  {
    "id": "region",
    "label": "Region",
    "uiType": "dropdown",
    "model": "sales",
    "column": "region",
    "default": "__all__"
  },
  {
    "id": "date_range",
    "label": "Date Range",
    "uiType": "daterange",
    "default": "__all__"
  },
  {
    "id": "price",
    "label": "Price",
    "uiType": "range",
    "min": 0,
    "max": 1000,
    "step": 10,
    "default": "__all__"
  }
]
```

| UI Type | Description | Default |
|---------|-------------|---------|
| `dropdown` | Single-select populated from model or static values | `"__all__"` |
| `multiselect` | Multi-select with checkboxes | `"__all__"` |
| `daterange` | Date range picker – use `@filter.start` and `@filter.end` | `"__all__"` or `"start,end"` |
| `range` | Numeric slider – use `@filter.min` and `@filter.max` | `"__all__"` or `"min,max"` |
| `text` | Free text input | `"__all__"` |
| `toggle` | On/off toggle | `"true"` or `"false"` |

Place `@filter_id` references in SQL WHERE clauses. Wrap in `[[...]]` for optional filtering – the clause is removed when the filter is `__all__`. Set `"excludeNull": true` on a dropdown to hide null values.

## Themes

Create a `.rk-theme.json` in `themes/` and reference it by name in your report's `"theme"` field.

`themes/default.rk-theme.json`:
```json
{
  "name": "default",
  "accent": "#0066FF",
  "palette": ["#0066FF", "#7C3AED", "#10B981", "#F43F5E", "#F59E0B"],
  "panel": {
    "border": "subtle",
    "radius": "medium"
  },
  "charts": {
    "strokeWidth": 2,
    "pointRadius": 3,
    "fillOpacity": 0.06,
    "barRadius": 4,
    "showGrid": true,
    "showDots": false,
    "showValues": false
  },
  "table": {
    "compact": false,
    "striped": true,
    "pageSize": 10
  },
  "kpi": {
    "valueSize": "xl"
  }
}
```

| Field | Options |
|-------|---------|
| `accent` | Primary UI color (hex) |
| `palette` | Chart series colors (cycled in order) |
| `panel.border` | `none`, `subtle`, `visible` |
| `panel.radius` | `none`, `small`, `medium` |
| `charts` | `strokeWidth`, `pointRadius`, `fillOpacity`, `barRadius`, `showGrid`, `showDots`, `showValues` |
| `table` | `compact`, `striped`, `pageSize` |
| `kpi.valueSize` | `sm`, `md`, `lg`, `xl` |
| `accents` | Per-panel-type accent override – e.g. `{ "KPICard": "#000000" }` |
| `palettes` | Per-panel-type palette override – e.g. `{ "BarChart": ["#0066FF", "#7C3AED"] }` |
| `mode` | `light` or `dark` |
| `nav.position` | `left` or `top` |

## Agent Setup

ReportKit supports two AI agents for building and editing reports:

| Agent | Install |
|-------|---------|
| **Claude Code** | `npm install -g @anthropic-ai/claude-code` |
| **OpenAI Codex** | `npm install -g @openai/codex` |

`npx @reportkit/cli init` auto-detects which agent is installed and installs the right skill file. To install or update manually:

```bash
npx @reportkit/cli update-skills
```

This copies the full skill (including reference docs) to `.claude/skills/building-reportkit/` (Claude Code) or `.agents/skills/building-reportkit/` (Codex).

### Using the agent

**Agent panel (recommended)** – open the sidebar in ReportKit and type a prompt. The panel runs the agent in the background and streams output inline.

**Terminal** – point your agent at the `building-reportkit` skill and give it a prompt:

```bash
# Claude Code
claude --skill building-reportkit "add a bar chart of revenue by month"

# Codex
codex --instructions .agents/skills/building-reportkit/SKILL.md "add a bar chart of revenue by month"
```

## Deploying

### Serve mode

`npx @reportkit/cli serve` starts ReportKit in view-only mode – all editing, agent, and connection-change routes return 403. Use this to safely share dashboards with your team. `npx @reportkit/cli` (no subcommand) is the full dev server.

### Docker

```dockerfile
FROM node:20-slim
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 3001
CMD ["npx", "@reportkit/cli", "serve"]
```

Pass secrets as environment variables:

```bash
docker run -p 3001:3001 \
  -e BIGQUERY_PROJECT=my-gcp-project \
  -e BIGQUERY_KEY_FILE=/run/secrets/sa.json \
  my-reportkit-image
```

Set `PORT` if your platform assigns a different port.

### Platforms

Railway, Render, Fly.io and similar platforms work out of the box – point them at your repo and set env vars in the dashboard.

## Examples

| Example | Description |
|---------|-------------|
| [`examples/demo`](./examples/demo/) | Full demo with seed data, models, multi-page reports, and themes |
| [`examples/example-dbt`](./examples/example-dbt/) | Minimal dbt integration *(coming soon)* |

## Documentation

- [Agent skill reference](./skills/building-reportkit/) – full spec for report JSON, models, panel types, filters, themes, and troubleshooting

